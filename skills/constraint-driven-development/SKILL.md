---
name: constraint-driven-development
description: Establishes a project's quality bar as a written contract and stops agents quietly lowering it. Interviews for the dimensions that matter, supplies default thresholds when you have no number in mind, records everything in CONSTRAINTS.md at the repo root, and watches the diff for a weakened bar — new lint or type suppressions, skipped or deleted tests, assertions stripped out, thresholds edited down. Use when no quality bar is written down, when you say "set up constraints" or "define our standards", when you want dimensions you care about (accessibility, coverage, complexity) set up as enforced constraints, when an agent keeps silencing checks to get to green, or when an agent writes more code than anyone will read.
---

# Constraint-Driven Development

## Overview

Other skills describe what good looks like. A code-review skill gives you axes. A TDD
skill gives you a cycle. A build-verify skill gives you a post-change loop. All of it
lives in prose the agent reads and may or may not follow, and none of it survives the
end of the session.

This skill produces something different: a written record of **this repo's** bar, with
numbers, that outlives the conversation and can be checked mechanically. An agent writes
more in an afternoon than you will read that week, so the judgement moves out of your
head and into checks that run around the loop. Those checks need to exist, need numbers
you actually chose, and need to fire close enough to the work that the agent fixes its
own output.

A spec says what to build. Tests prove it works. Constraint-driven development defines
what "good enough to ship" means, before anyone argues about it in a pull request.

## When to Use

- Starting a project or a significant feature and no quality bar is written down
- You ask to "set up constraints", "add quality gates", or "stop the agent shipping junk"
- An agent is producing volume nobody is reading line by line
- CI has checks but nobody can say which block a merge and which are decoration
- Coverage, complexity, or dead-code numbers get argued per-PR instead of decided once
- Before an autonomous or long unattended run where the only thing between it and a PR
  is a test suite the agent also wrote

**When NOT to use:**

- The repo already has a `CONSTRAINTS.md` and you aren't changing it — read it and follow it
- One-off scripts, spikes, throwaway prototypes
- You want a review right now, or a CI pipeline built
- A dependency you consume but don't own the standards for — follow its existing bar

## Loading Constraints

The interview needs a live user. **Don't run it in non-interactive contexts**
(autonomous runs, CI). If constraints are missing and you're in one of those, apply the
Floor below, note that you did, and flag the rest for a human.

## The Process

### Step 1: Detect before you ask

Never ask what you can read. Before the first question, gather the stack, test runner,
existing linters, coverage today, CI config, and any static-analysis tooling — from the
dependency manifest, config files, and CI workflows. Report what you found in two lines,
then ask only what's left.

### Step 2: Four questions, each with a default

Follow a one-question-at-a-time interview discipline, with one change: every question
has a default, so "I don't know" is a complete answer that still produces a working
config.

```
Q1: Beyond the floor, which of these do you want enforced?
    (a) Test coverage on changed lines
    (b) Static analysis (dead code, complexity, duplication)
    (c) Accessibility
    (d) Browser / platform-support verification
    (e) Architecture boundaries
    DEFAULT if unsure: (a) and (b).
    Say what each costs: (c) needs a running URL, (d) is a per-API compat lookup,
    (e) needs boundary rules configured.
```

```
Q2: When a check fails mid-task, should it block or warn?
    DEFAULT: Block on the floor, warn on everything else for the first two weeks.
```

```
Q3: Target numbers in mind, or measure where you are today and hold that line?
    DEFAULT: Measure and hold. See "Ratchets" below.
```

```
Q4: Slowest check you'll tolerate before the agent hands work back?
    DEFAULT: 90 seconds at task end, unlimited in CI.
```

Stop at four. A twelve-question intake produces a config nobody understands.

### Step 3: Write CONSTRAINTS.md

One file at the repo root. Any agent reads it, and a change to it shows up in review
where it belongs.

```markdown
# Constraints

Last reviewed: <date> by <owner>

## Floor (always enforced, no setup required)

- No new suppression comments (lint-disable, type-ignore, coverage-ignore) without a reason in the commit message
- No unimplemented stubs: a throw-not-implemented, an empty catch
- No skipped or deleted tests without a reason in the commit message
- No secrets in source
- This file does not get weakened to make a change pass

## Enforced with numbers

| Dimension | Rule | Checked by | Runs at |
|-----------|------|-----------|---------|
| Lint | Zero errors from our config | <lint command> | every edit |
| Coverage | Changed lines >= 80% covered | <test runner> --coverage + git diff | task end, CI |
| Static analysis | No new findings | <analysis command> | task end, CI |
| Accessibility | Zero critical or serious | <a11y command against a URL> | preview deploy |

Every row names the command that produces the verdict. A dimension with a number and no
command is an aspiration, not a constraint.

## Measured, not yet enforced

| Metric | Today | Direction |
|--------|-------|-----------|
| Project coverage | 62.4% | must not fall |
| Bundle size | 184 kB | must not grow |

## Exceptions

| ID | Rule | Path | Reason | Owner | Expires |
|----|------|------|--------|-------|---------|
| W1 | <rule> | <path> | <tracked in ticket> | <owner> | <date> |
```

### Step 4: Install what each dimension needs — map to the existing toolchain

Don't invent a checker when the team already runs one. Map every dimension to a tool
already in the stack, so the team's existing config keeps working:

| Dimension | Map to | Gate on |
|-----------|--------|---------|
| Lint | your existing lint config | any error |
| Coverage | your test runner's coverage output | coverage of changed lines |
| Static analysis | your dead-code / complexity / duplication tool (example: Fallow, a structural analyser) | new findings only |
| Architecture | your boundary / dependency rules | any violation |
| Accessibility | an accessibility engine (example: axe-core) against a running URL | zero critical or serious |
| Browser / platform support | an authoritative compat source (MDN, caniuse) per API | unsupported in a target |

Four things that bite if you skip them:

1. **Scope static analysis to the diff.** Gate on findings this change *introduced*, not
   the inherited backlog. Whole-project counts are a ratchet (below), not a per-PR gate.
2. **Accessibility needs a URL.** It only works against a running app, so it belongs at a
   preview/staging stage, not the edit loop. If there's no URL (a CLI, a library), say so
   and drop the dimension rather than inventing a check that can't run.
3. **Platform support is a lookup, not a tool.** Query an authoritative compat source
   against the target matrix. Transpilers handle syntax; this is for runtime/platform APIs.
4. **Coverage needs no second test run.** Read the coverage report your suite already
   writes and intersect with the diff. Running the suite twice is how people learn to hate this.

Mirror the commands into the project's own scripts (`check:fast`, `check:task`,
`check:full`) so they're reproducible without an agent. `CONSTRAINTS.md` is the canonical
source; the scripts mirror it, and if they drift, the file wins.

### Step 5: Wire it to the lifecycle

The biggest mistake is running everything everywhere. A check that stalls the agent gets
switched off, and a gate switched off is worse than no gate — the bar still looks like it
exists.

| Phase | What runs | Budget |
|-------|-----------|--------|
| BUILD | Lint, the floor | under 5s, changed file only |
| VERIFY | Related tests, coverage on changed lines | under 90s |
| REVIEW | Static analysis, everything, plus the guards below | minutes |
| PREVIEW | Accessibility against a running URL | preview/staging |
| SHIP | Direction checks, no regressions | CI |

Two rules that keep this tolerable: **scope to the diff** (numbers the agent can move, not
inherited totals), and **cost decides placement** (anything over a few seconds leaves the
edit loop).

### Step 6: Guard the bar itself

If the agent writes the code and the checks, the checks prove nothing — half right.
Agents don't craft clever loopholes; they hit a red check and take the cheapest road to
green. Watch for these five moves in the diff at review time:

1. **The threshold moved.** Compare `CONSTRAINTS.md` against its state at the branch point.
2. **A test got easier.** A skip added, a test file deleted, assertions pulled out.
3. **A checker got silenced.** A new suppression comment without a reason.
4. **Work is unfinished.** A stub that throws, an empty catch, a TODO where the code should be.
5. **An exception appeared.** A new row in the Exceptions table nobody discussed.

None of this needs tooling beyond a diff. Tightening the bar should be silent; loosening
it should be loud.

**Not all checks are equally circular.** Rank by one question: can the agent make this pass
by writing code that doesn't work?

- **External** — an accessibility engine encodes a standard, a compat database is real,
  structural analysis doesn't care about intent. The agent can't argue with these.
- **Project** — your lint rules, your boundaries. A human owns the rules.
- **Suite** — your own tests. Most useful, and the only genuinely circular one.

A bar made entirely of the third kind is worth less than one with an outside opinion.
Check that at least one external constraint is present.

### Step 7: Ratchets, when you don't have a number

Set 80% coverage on a repo at 62% and you get a red build forever, then a team that learns
to ignore red builds. The alternative asks for no decision: record where you are, then
refuse to get worse. Put it in "Measured, not yet enforced" with today's number and a
direction. Every check compares against the recorded value. When a number improves, update
it; when it drops, that's the finding. This is the natural home for whole-project
static-analysis counts and bundle size — inherited backlogs you hold flat while gating the
diff tight.

## Sane Defaults

| Constraint | Default | Why |
|------------|---------|-----|
| Coverage of changed lines | >= 80% | High enough to force a test, low enough to allow a config line |
| Project coverage | today's value, must not fall | No argument needed to adopt |
| Static analysis (project) | today's count, must not grow | Backlog held flat, diff gated tight |
| Accessibility | zero critical or serious violations | Moderate/minor often debatable |
| Exception lifetime | 90 days | Long enough to plan the fix, short enough to remember |
| Ratchet tolerance | 0.5% | Absorbs drift when an unrelated file moves the number |

State the number and the reason together. A threshold without a rationale gets deleted by
the next person who hits it.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "We'll add constraints once the code settles" | Code settles around whatever was allowed while it was moving |
| "The tests are the constraints" | Tests you wrote prove you agree with yourself; they say nothing about coverage of new code, dead code, or duplication |
| "We can't hit 80% coverage" | Then don't set 80%. Set today's number and hold it |
| "This will slow the agent down" | Only if you put slow checks in the fast loop. That's a placement error |
| "I'll remember our standards" | The agent won't, and it's writing most of the code |
| "Constraints block shipping" | An exception with an owner and a date unblocks you. Deleting the constraint unblocks everyone forever |

## Red Flags

- The interview ran past four questions, or produced a config the user can't explain
- A budget was set that the repo fails today, with no plan to reach it
- A dimension written with a number but no tool behind it
- A checker hand-rolled when an existing tool already does it
- Every constraint checked by the project's own tests, with no external opinion
- `CONSTRAINTS.md` changed in the same commit as the feature that was failing
- An exception with no owner or an expiry over a year out
- The agent proposed relaxing a threshold instead of fixing the code
- Slow checks landed in the edit loop and someone started bypassing the hook

## Verification

- [ ] `CONSTRAINTS.md` exists at the repo root, and every number has a stated reason
- [ ] The floor is enforced and passes on the current code without changes
- [ ] Every chosen dimension maps to a tool already in the stack with a command that runs today
- [ ] Each constraint says where it runs, and the fast stage stays under a few seconds
- [ ] At least one external constraint is present
- [ ] Measured-only metrics record today's value and a direction
- [ ] Exceptions have an owner and an expiry date
- [ ] A trial run on the current branch produces no failures the user disagrees with

## See Also

- A grilling/interview skill — the one-question-at-a-time discipline this skill's intake borrows
- A code-review skill — how to review; this skill decides what the review enforces
- A build-verify skill — the post-change loop the fast checks run in
- A TDD skill — the suite coverage constraints measure
