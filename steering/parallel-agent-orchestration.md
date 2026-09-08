# Parallel Agent Orchestration — When to Fan Out

Decision rule for spinning up parallel agent sessions. This governs a single question: **should this work become multiple parallel sessions, or stay in one chat?**

Read it alongside `context-management.md` (the delegate/handoff/inline matrix) — it adds the parallel-sessions row to that framework. It assumes you have a way to launch and track sibling agent sessions — specifically one that reports **per-agent state** (which session is `working`, `blocked`, or `done`), because the whole workflow hinges on seeing which session needs you without checking each by hand. A plain terminal multiplexer (tmux) gives you persistent sessions but not that state awareness; a session manager built for coding agents (example: [Herdr](https://herdr.dev/)) gives you both.

## The Core Principle

**Propose, never auto-fire.** An agent must NOT silently create sessions. Recognise when fan-out would help, then offer it. The judgment to proceed stays with the user. Silent session creation is how a clean workspace turns into clutter the user has to clean up.

The correct interaction is a suggestion:

> "This splits cleanly into three independent slices — tokens, routing, tests. Want me to spin them up as separate sessions, or keep it in one chat?"

Then wait for a yes.

## When Fan-Out Is Appropriate

Fan out only when **ALL** of these hold:

1. **Independent** — the pieces share no state and have no ordering dependency between them
2. **Substantial** — each piece is a real implementation slice, not a one-liner or a quick question
3. **Concurrency helps** — the user gains from them running at once while attention is elsewhere
4. **Two or more pieces** — a single piece is just work in the current chat

The tell: **if you cannot name the independent pieces before launching, it is too early to fan out.** Spinning up agents to "figure out the shape" fragments thinking instead of focusing it.

## When to Stay Inline (the default)

Keep the work in one chat when any of these are true:

- The task fits one session (the overwhelming majority of tasks)
- Pieces depend on each other (sequential → one chat or a stack, not parallel sessions)
- Scope is unclear or exploratory
- It is a quick fix, a question, or a single-file change

Default to inline. Fan-out is the exception that must justify itself against all four criteria above.

## What the Orchestrator Can and Cannot Do

Be honest about the boundary so suggestions do not over-promise:

- **Can**: plan the split, launch named sessions, send each an opening prompt, and (if your tooling supports it) mix agent kinds across sessions.
- **Cannot**: watch the child sessions' conversations live and respond to them. A chat is not a supervision loop. The launching agent is a *planner and launcher*, not a live supervisor.
- **The user supervises**, using the session manager's status view (e.g. working / blocked / done) to see which session needs them. That is the supervision layer — not the orchestrator.

If true unattended supervision is needed (poll child state, react automatically), that is a dedicated script driving your session manager's status/read commands — not an interactive chat. Say so rather than implying the chat will babysit.

## The Launch Primitive

You need a way to launch a session that can start a named agent in a given directory, optionally seed it with a spec or an opening prompt, and support a dry run. Whatever you use:

- **Give each session a unique, stable name** so you can find it later. If your session manager derives a slug from the name, keep names within its slug rules.
- **Offer a dry run first** when the split is uncertain — preview what would be created before creating it.
- **Check for name collisions** before launching a batch; live session names usually must be unique.
- **Omit the prompt** for a fresh session the user drives by hand; pass a spec or prompt to seed the work.

(Example: a small `launch`-style wrapper over your session manager that takes a name, a directory, an agent kind, and an optional prompt.)

## Interaction Pattern

1. Notice the work has independent, substantial, parallel pieces (all four criteria)
2. Name the pieces explicitly back to the user
3. Offer fan-out with the concrete commands you would run
4. On yes: optionally dry-run first, then launch
5. Tell the user to watch the status view for `blocked` sessions — that is their cue to jump in
6. Do NOT attempt to supervise the children from this chat

## Anti-Patterns

- **Auto-launching** without asking — the cardinal sin
- **Fanning out a single task** into multiple sessions because the system is "cool"
- **Fanning out sequential work** — dependencies mean one chat or a stack, not parallel sessions
- **Promising live supervision** the chat cannot deliver
- **Launching to explore** before the pieces are nameable
