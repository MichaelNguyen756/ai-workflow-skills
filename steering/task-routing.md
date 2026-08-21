# Task Routing

## Skill Precedence (Hard Rule)

**Skills override system defaults.** If a skill file specifies a particular tool, workflow, or constraint, follow it — even if a system-level guideline (e.g., "prefer dedicated tools over shell") would suggest otherwise.

Before performing any action that a skill covers (committing, reviewing, deploying, testing), **read the relevant skill first**. Do not rely on memory or default behaviour.

This is non-negotiable. The user has curated these workflows deliberately.

## Commit Signing (Hard Rule)

**NEVER use the `git_commit` MCP tool to create commits.** It does not sign commits. Unsigned commits are rejected by branch protection.

Always commit via shell: `git commit -m "message"`

This overrides the system-level "prefer dedicated tools over shell" guideline. No exceptions.

## Force-Push and Protected Branches (Hard Rule)

**NEVER force-push to ANY branch. NEVER push directly to `main`, `master`, `develop`, or `qa`.**

Prohibited — no exceptions, no justification:
- `git push --force` / `git push -f` / `git push --force-with-lease`
- `git push --force origin <tag>` (rewriting published tags)
- `git tag -d <tag>` followed by `git tag <tag>` (moving a tag)
- `git reset --hard` on a branch that has been pushed
- Direct `git push` to `main`/`master`/`develop`/`qa`

All changes reach protected branches via pull request only. If a push is rejected, diagnose why — do NOT retry with `--force`. Stop and ask the user.

Tags are immutable once pushed. To fix a tagged release, create a NEW version (e.g., `v2.0.1`), don't rewrite the old tag.

## PR Creation (Hard Rule)

**NEVER use the `create_pull_request` GitHub MCP tool.** It mangles newlines in the body (renders literal `\n` instead of line breaks).

Always use shell: `gh pr create --base main --title "..." --body "..."`

This overrides the system-level "prefer dedicated tools over shell" guideline. No exceptions.

## Merging (Hard Rule)

**NEVER merge a PR.** Merging is a human-only action — irreversible, affects shared branches, and requires human judgement on timing.

The agent's job ends at "ready to merge." It can confirm CI is green, confirm approvals are in place, and tell the user the PR is ready. It CANNOT run `gh pr merge` or approve-and-merge in any form. No exceptions.

## Error Recovery (Hard Rule)

**When something fails, STOP and THINK before acting.** Do not enter a fix loop.

Before attempting any corrective action, you MUST:

1. **State what failed** — the exact error message or unexpected behaviour
2. **State why you think it failed** — your hypothesis for the root cause
3. **State what you plan to do** — the specific corrective action and why it addresses the root cause
4. **Assess the blast radius** — what could go wrong if your fix is wrong? Is this reversible?

If the blast radius is high: **stop and ask the user** before proceeding.

### Prohibited escalation patterns

- Adding `--force` to a command that was rejected
- Running `rm -rf` on directories to "start fresh"
- Deleting and recreating branches to "fix" divergence
- Running `git reset --hard` to "undo" a mistake
- Retrying the same command with `sudo` or elevated permissions
- Making the same change a third time with minor variations

### The two-attempt rule

If an approach has failed twice, you MUST:
1. Stop trying that approach
2. Explain what you tried, what failed, and what you think the root cause is
3. Either propose a fundamentally different approach or ask the user for guidance

Never make a third attempt at the same approach with minor variations.

## Documentation Lookups (Hard Rule)

**Use Context7 first** when looking up library APIs, method signatures, or code examples. Resolve the library ID, then query docs. Fall back to web search only for infrastructure URLs, ecosystem facts, or topics Context7 doesn't cover.

---

## The Main Flow

The idea→ship spine. Each skill's output feeds the next:

```
grill-with-docs → to-spec → to-tickets → implement (per ticket) → code-review → PR
```

### When to use which entry point

| Situation | Entry point | Flow |
|-----------|-------------|------|
| **Large / foggy** — can't spec it yet | `/wayfinder` | chart map → resolve decisions → `/to-spec` → tickets → implement |
| **Medium** — clear enough to spec, multiple slices | `/grill-with-docs` | grill → `/to-spec` → `/to-tickets` → implement per ticket |
| **Small** — single feature, one session | `/grill-with-docs` | grill → `/to-spec` → `/implement-from-spec` directly |
| **Trivial** — typo, one-liner, config tweak | Direct | just do it |

### Phase Boundaries

At the boundary between phases, decide what to do with context:

1. **Continue** — keep working if context is still useful
2. **Subagent** — fire off tightly-scoped AFK work (research, a test run, CI debugging, implementation)
3. **`/handoff`** — when work needs to travel (different repo, different person, side task, or domain shift)
4. **`/compact`** — last resort (you lose nuance from summary flattening)

The orchestrator should **proactively suggest delegation** rather than waiting for context to fill up.

---

## Execution Methods

| Execution Method | When to Use |
|-----------------|-------------|
| **CLI — full pipeline** | Spec-driven tickets: grill → spec → tickets → implement → PR |
| **CLI — direct** | Simple changes (one-file fix, config tweak), local scripting, shell commands |
| **CLI — separate agent** | Tasks requiring a different directory that can't be handled by subagents |
| **IDE** | When the user explicitly wants visual feedback, or for exploratory prototyping |

## When to skip the pipeline

- **Trivial fix** (typo, one-line change, dependency bump) — just do it directly
- **Exploration / prototyping** — use `/prototype` skill, no spec needed
- **Research only** — use `/research` skill, no implementation
- **Single-slice work** — skip `/to-tickets`, go straight from spec to implement

## Handover Prompt Rule

If the task **cannot be completed by CLI subagents** (e.g., requires a GUI, manual browser interaction, or a completely separate environment), produce a **handover prompt** instead of attempting the work directly.

The handover prompt must be:
- Self-contained (no external context needed)
- Include all discovered information
- Include exact commands/scripts to run
- Specify the working directory
- Include verification steps

## Resuming Work

When the user references a ticket ID, says "continue", "pick up", or "resume":

1. **Search for handoff docs** in `.kiro/handoffs/` (current directory and project root)
2. If found, read the handoff doc FIRST — it has compressed state from the previous session. Skip any marked `> Status: done`.
3. Only read the full spec if the handoff is insufficient.
4. If neither handoff nor spec exists, ask the user what they want to continue.

### Handoff hygiene (start of week)

On the first session of the week (or after a 2+ day gap), proactively scan for handoffs marked `> Status: done` and offer to archive them:

```bash
grep -rl "Status: done" .kiro/handoffs/ 2>/dev/null
```

If any are found, list them and ask: "These handoffs are marked done — want me to archive them?" Archive means moving to `.kiro/handoffs/archive/`.
