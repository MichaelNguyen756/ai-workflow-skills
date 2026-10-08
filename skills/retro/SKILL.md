---
name: retro
description: "Run a retrospective on a coding-agent session transcript and suggest improvements to the agent's environment — steering/AGENTS files, skills, checks, tooling. Use when the user asks for a retro, a session retrospective, or wants to improve how the agent works after a run."
disable-model-invocation: true
---

# Retro

The user has asked for a **retrospective**. You are suggesting improvements to the coding
agent's **environment** — the always-loaded instructions, skills, automated checks, and
tooling that shape future runs. You are **not** reviewing the code the session produced;
that is the review stage's job. Retro improves the loop, not the diff.

This is **strictly human-in-the-loop.** You surface candidates; the human decides every
one. Never apply a finding automatically, and never run this as a scheduled or background
task — a retro that edits its own environment unattended loops on false positives and
drags the setup somewhere bad.

## Steps

1. Call the Skill tool with `writing-for-agents` for the writing-style guide. Every
   environment edit a finding proposes (an instruction tweak, a new skill, a pointer) must
   follow it.

2. **Read the primary sources for the session under retro.**

   Find where your agent stores session transcripts. Many CLIs keep one log per session
   (for example, a per-session JSONL file under the tool's config directory, one message
   per line tagged as a user turn, an assistant message, or a tool result). Locate that
   store for your tool before going further.

   - **Which session?** If the user named one, use it. Otherwise default to the
     **current** session: resolve it as the most-recently-modified transcript, state which
     file and its timestamp, and confirm with the user before reading. There is usually no
     "current" pointer on disk, so the resolution is a guess until confirmed.
   - **Sample, do not slurp.** Session transcripts run large — tool-result payloads
     (file reads, command output) dwarf the turn count, so a short conversation can still
     be hundreds of MB. Read the whole file into context and you will blow the window.
     Instead sample: the first and last N lines for the arc, plus the turns around any
     tool result carrying an error (grep for error markers), plus the user turns for
     intent. Pull more only where a finding needs it.

3. **Hunt for improvement candidates** in these categories. Each names *when* it fires —
   look for that trigger in the transcript.

   - **Navigation** — how hard was it for the agent to find the right file or fact? A
     missing **navigation pointer** (a line in an always-loaded doc that names
     out-of-context material and the branch that should reach it) is the usual fix. _Fires
     when_ the session spent many turns hunting for something a pointer could have named.
   - **Automated checks** — could a deterministic check have caught a mistake the agent
     made? Read the repo's own check command first (its lint/typecheck/test scripts, its
     CI workflow) so the finding is "this check exists but sits unwired or broken," not a
     reinvention. A repo with no **guardrail** at all — no pre-commit hook, no CI running
     its lint/typecheck/test — is itself a finding. _Fires when_ the agent made a mistake
     a check could have caught, or the repo has no guardrail.
   - **Coding standards** — classify the violation first. A **mechanical** one (a fixed
     syntactic pattern, a banned API, an import shape, a file-location rule) gets a
     deterministic check: a custom linter rule, a pre-commit hook, or a CI job — whichever
     is cheapest. Default to building the check over writing prose. Reserve a written
     standard for genuine **judgement calls** (cross-file consistency, "matches the
     surrounding style") that no check could substitute for — and put that rule where the
     **review stage** reads it, not where every implementation turn pays for it. _Fires
     when_ the review stage failed to catch a mistake.
   - **Steering health** — are there always-loaded instructions (a global or repo-level
     `AGENTS.md` / `CLAUDE.md` / steering doc) that should move to an automated check or a
     review-stage rule instead? Always-loaded prose is the most expensive place to put
     anything a check could own. _Fires when_ an always-loaded doc is large and carries
     rules a check could enforce.
   - **Tool economy** — did the agent make expensive or repetitive tool calls that a
     narrower tool, a cached lookup, or a reference doc could streamline? Is a custom
     CLI/MCP token-inefficient (dumping huge payloads the agent samples a sliver of)?
     _Fires when_ a tool call was expensive relative to what it yielded.
   - **No-ops** — instructions in always-loaded docs or skills that don't change behaviour
     versus the agent's default. They pay context load to say nothing. _Fires when_ a doc
     is large and you can point at a line the agent would obey anyway.
   - **Information access** — a crucial fact the agent never had access to. Teeing a dev
     server's logs, a read-only credential for a third-party dashboard, a reference store
     for a domain it kept guessing at. _Fires when_ the session stalled on information
     that was retrievable but not wired in.

4. **Present the candidates to the user, in order of severity.** For each: the category,
   the evidence from the transcript (which turns), the proposed environment change, and
   where it would live (which doc, which skill, which config). Then stop. The human picks
   which to apply.

## Reference

### Implementation vs Review

All work goes through two stages, and they carry different **context pressure**. The
implementation agent has the most: it explores, writes code, and debugs failures, all in
one window. The review agent has the least — it receives a diff, so no exploration, often
no writing or debugging. This is why mechanical standards belong at **review** time, not
loaded into every implementation turn: the reviewer can afford to hold the rulebook, the
implementer cannot.

### Where environment material lives

Map these to your own tool; the tiers are what matter, not the names:

- **Always-loaded instructions** (`AGENTS.md` / `CLAUDE.md` / a global steering doc) —
  pushed into the context window on **every** turn, every session. The most expensive real
  estate you have. Keep it to **navigation pointers** and genuine judgement conventions;
  anything a check or a review-stage rule could own does not belong here.
- **Skills** — loaded only when their description fires (or when user-invoked). A skill's
  *description* is always-loaded, though, so it pays the pointer cost — write it as a
  pointer. Use skills for on-demand reference docs and for user-invoked commands.
- **The review stage** (a review skill or a dedicated reviewer agent) — reads the diff,
  not the whole repo. A new mechanical rule should become a linter check before it becomes
  a line here; reserve this for judgement calls.
- **Repo-level instructions** — loaded only when working in that repo. Prefer them over
  global ones for anything product- or repo-specific.
- **Reference stores** (a docs folder, a knowledge base) — for large reference material
  the agent should search on demand rather than hold in context.

Look for existing material to extend before adding new — a pointer into an existing doc
beats a new doc. Follow `writing-for-agents` for every edit.

---

_Adapted from Matt Pocock's `retro` skill ([mattpocock/skills](https://github.com/mattpocock/skills), MIT)._
