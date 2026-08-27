# To Tickets

Break a spec or conversation into a set of **tickets** — tracer-bullet vertical slices, each declaring the tickets that **block** it.

## Process

### 1. Gather context

Work from whatever is already in the conversation. If the user passes a reference (a spec path, issue number, or URL), fetch and read its full body.

### 2. Explore the codebase

If you haven't already explored the codebase, do so. Ticket titles and descriptions should use the project's domain vocabulary, and respect ADRs in the area you're touching.

Look for opportunities to prefactor the code to make the implementation easier. "Make the change easy, then make the easy change."

### 3. Draft vertical slices

Break the work into **tracer bullet** tickets.

**Vertical slice rules:**

- Each slice cuts a narrow but COMPLETE path through every layer (schema, API, UI, tests) — vertical, NOT a horizontal slice of one layer
- A completed slice is demoable or verifiable on its own
- Each slice is sized to fit in a single fresh context window
- Any prefactoring should be done first

Give each ticket its **blocking edges** — the other tickets that must complete before it can start. A ticket with no blockers can start immediately.

**Wide refactors are the exception to vertical slicing.** When one mechanical change fans across the codebase (rename a column, retype a shared symbol), sequence it as expand–contract: first expand (add the new form beside the old), then migrate call sites in batches, then contract (delete the old form). Each batch is its own ticket blocked by the expand.

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each ticket, show:

- **Title**: short descriptive name
- **Blocked by**: which other tickets (if any) must complete first
- **What it delivers**: the end-to-end behaviour this ticket makes work

After the ticket list, present the **wave plan** — how the tickets group for execution:

- **Parallel wave**: tickets sharing no blocking edges. Each can be implemented independently (own worktree, own agent).
- **Sequential wave**: a chain where A blocks B blocks C. One worktree, one agent, stacked PRs (each PR based on the one below it).
- **Mixed**: parallel wave first, then the sequential wave starts after the parallel wave merges.

Example:

> **Wave 1 (parallel):** #101, #102, #103 — 3 worktrees, merge any order
> **Wave 2 (sequential):** #104 → #105 → #106 — 1 worktree, stacked PRs, merge bottom-up

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the blocking edges correct?
- Should any tickets be merged or split further?
- Does the wave plan look right?

Iterate until the user approves the breakdown.

### 5. Publish tickets

Publish the approved tickets to whatever tracker the project uses. Create in dependency order (blockers first) so each ticket's blocking edges can reference real identifiers.

**GitHub Issues:**
- Create issues with the template below
- Use task lists or "blocked by #N" references for dependencies

**Linear:**
- Create issues with native blocking relationships

**Local files (no tracker):**
- Write one file per ticket under `.scratch/<feature>/issues/<NN>-<slug>.md`
- Number from `01` in dependency order

**Other trackers:**
- Adapt to the platform's native blocking/dependency mechanism

#### Ticket template

```markdown
## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective — not a layer-by-layer implementation list.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by

- #N / ticket-ref (or "None — can start immediately")
```

Avoid specific file paths or code snippets in ticket descriptions — they go stale fast. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, schema, type shape), inline it and note it came from a prototype.

### 6. Offer parallel worktrees

After publishing, check if any tickets share no blocking edges (i.e., they can start simultaneously). If two or more tickets are independent, offer:

> "Tickets X and Y have no blocking edges — want me to set up worktrees so they can be implemented in parallel?"

If accepted:

1. **Create a worktree per ticket**, branch per ticket:
   ```bash
   git worktree add ../<repo>-<ticket>-<slug> -b <type>/<ticket>-<slug>
   ```

2. **Install dependencies** in each worktree.

3. **Create a scoped spec** in each worktree containing only that ticket's scope and acceptance criteria.

4. **Launch an agent per worktree.** The launch prompt is self-contained: "implement the spec at `<path>`, then open a PR."

**Cleanup:** After a ticket's PR is merged, remove its worktree: `git worktree remove ../<dir>`

#### Automating the handoff

Steps 1–4 are mechanical, so a terminal orchestrator can do them for you instead of hand-copying launch prompts between sessions. A tool that manages worktrees, panes, and agent lifecycle (example: [Herdr](https://herdr.dev)) collapses the loop to: create worktree + workspace, run install in its pane, write the scoped spec, start the agent, send the prompt. The orchestrator agent then spawns its siblings directly.

The generic shape, whatever the orchestrator:

```
orchestrator:
  for each independent ticket:
    create worktree + isolated workspace
    install dependencies
    write scoped spec
    start agent in that workspace, seeded with the implement prompt
  → agents run in parallel, each surfacing a status (working / blocked / done)
you:
  → jump to whichever agent is blocked, approve, move on
  → review + merge PRs
```

The win is that the orchestrator is itself an agent session that runs shell commands, so it spawns the others — no copy-paste, no launch scripts to go stale.

### Sequential wave (stacked PRs)

When the wave plan identifies a sequential chain (A → B → C), use a single worktree with one agent that implements the whole stack:

1. Create one worktree for the chain.
2. Install dependencies and write all scoped specs (one per ticket in the chain).
3. Start one agent with a stacking prompt: implement each ticket in order, commit, branch the next ticket on top, and at the end open the stack as a set of PRs (each based on the one below).

Review bottom-up, merge bottom-up, rebasing the remainder after each merge. Tooling like [gh-stack](https://github.com/timothyandrew/gh-stack) or Graphite automates the rebase cascade, but plain git (`git rebase --onto`) works too.

### Wave orchestration (merging a batch of parallel PRs)

After parallel agents finish and PRs are open:

1. **Review all PRs** from the orchestrator session — cross-cutting visibility catches issues individual agents can't see (phantom dependencies, conflicting changes).
2. **Merge the first PR.**
3. **Update remaining branches** — rebase each remaining worktree onto the updated main.
4. **Resolve conflicts** if any (often in lockfiles) — regenerate, commit.
5. **Repeat** until all PRs in the wave are merged.
6. **Clean up worktrees** for each merged ticket.

## Rules

- Always quiz the user before publishing. Never publish unreviewed tickets.
- Each ticket must be independently demoable or verifiable.
- A ticket's "What to build" describes user-visible behaviour, not implementation steps.
- Size tickets to fit one agent session (~one context window).
- Work the **frontier**: any ticket whose blockers are all done.
