# AI Workflow Skills

A practical skill system for AI coding agents. Install the skills you want, type a slash command, and the agent follows a process you actually trust.

Based on [Matt Pocock's Skills](https://github.com/mattpocock/skills) ([aihero.dev/skills](https://www.aihero.dev/skills)), adapted for Jira workflows, git worktrees for parallelism, and real-world CI/CD conventions.

Works with **Kiro CLI** and **VS Code + Copilot**.

## The Main Flow

Each skill's output feeds the next:

```
/grill-with-docs → /to-spec → /to-tickets → /implement-from-spec → /code-review → PR
```

| Situation | Entry point | Flow |
|-----------|-------------|------|
| **Large / foggy** | `/wayfinder` | Chart decisions → resolve → spec → tickets → implement |
| **Medium** | `/grill-with-docs` | Grill → spec → tickets → implement per ticket |
| **Small** | `/grill-with-docs` | Grill → spec → implement directly |
| **Trivial** | Direct | Just do it |

## Installation

### Kiro CLI (interactive picker)

```bash
git clone https://github.com/MichaelNguyen756/ai-workflow-skills.git
cd ai-workflow-skills
./setup.sh              # Pick skills by category
./setup.sh --universal  # Install all, no prompts
```

### VS Code + Copilot (per-repo conventions)

```bash
./install.sh /path/to/your-project
```

Installs 5 passive instruction files into `.github/instructions/`:
- `commit-messages.instructions.md` — commit format (always-on)
- `pull-requests.instructions.md` — PR conventions (always-on)
- `branch-naming.instructions.md` — branch format (always-on)
- `code-review.instructions.md` — code conventions (always-on)
- `testing.instructions.md` — test conventions (`*.spec.*` files only)

## Skills (29)

### The Main Flow

| Skill | Purpose |
|-------|---------|
| `/grill-with-docs` | Rounds-based interview + domain docs (ADRs, glossary) |
| `/to-spec` | Synthesise decisions into a spec file |
| `/to-tickets` | Break spec into vertical-slice tickets with blocking edges |
| `/implement-from-spec` | Plan → build → test → review → commit |
| `/code-review` | Review diff against conventions and spec |
| `/pull-requests` | PR creation with conventions |
| `/commit-messages` | Conventional commit format |

### Shaping (exploration)

| Skill | Purpose |
|-------|---------|
| `/wayfinder` | Chart large efforts as a decision map |
| `/prototype` | Throwaway code to answer design questions |
| `/research` | Background agent for primary-source investigation |
| `/grill-me` | Stress-test an idea (no docs output) |

### Upkeep

| Skill | Purpose |
|-------|---------|
| `/improve-codebase-architecture` | Find modules worth refactoring |
| `/diagnosing-bugs` | Systematic diagnosis loop |
| `/resolving-merge-conflicts` | Hunk-by-hunk conflict resolution |
| `/security-audit` | Full security scan checklist |
| `/dependency-check` | Evaluate packages before adding |
| `/qa-build` | Push to QA branch for staging |

### Reference (invoked by other skills)

| Skill | Purpose |
|-------|---------|
| `/grilling` | Rounds-based interview loop (design tree + frontier) |
| `/domain-modeling` | Maintain glossary + ADRs |
| `/tdd` | Red-green-refactor at seam boundaries |
| `/codebase-design` | Deep modules vocabulary |
| `/build-verify` | Post-change lint + test + CI simulation |
| `/writing-for-agents` | Reference for authoring skills and agent docs |
| `/cicd-conventions` | GitHub Actions, AWS, anti-patterns |

### Productivity

| Skill | Purpose |
|-------|---------|
| `/handoff` | Compress session state for another agent |
| `/teach` | Learn a topic across sessions |
| `/wait-what` | Re-pitch last message in plain English |
| `/to-questionnaire` | Turn a decision into a questionnaire for someone else |
| `/version-bump` | Version bump + release PR |

## Key Features

### Rounds-based grilling

Questions come in rounds — the agent asks all unblocked questions at once:

```
❓ Q1 - **State management**: Local state or shared store?
➡️ Local — page-scoped

❓ Q2 - **Error handling**: Inline or toast?
➡️ Inline — matches existing patterns
```

Answer by number: "Q1 agree, Q2 change to..."

### Parallel work with git worktrees

When `/to-tickets` produces independent tickets, implement them in parallel:

```bash
git worktree add ../my-project-PROJ-101-feature -b feat/PROJ-101-feature
```

Each worktree gets its own branch, `node_modules`, scoped spec, and launch prompt.

### CI simulation

Before pushing, `/build-verify` can simulate your CI pipeline locally:

```
CI Simulation:
✓ Install (frozen lockfile)
✓ Lint
✓ Typecheck
✓ Tests (27 suites, 203 passed)
✓ Build

Prediction: CI will pass ✅
```

### Phase boundaries

At each skill boundary, decide what to do with context:

1. **Continue** — keep working
2. **`/tangent`** — quick side-investigation without polluting main thread
3. **Subagent** — fire off AFK work
4. **`/handoff`** — when work travels
5. **`/compact`** — last resort

## Customisation

After installing, adapt to your team:

- Edit `/to-tickets` to point at your issue tracker (Jira, Linear, GitHub Issues)
- Edit `/commit-messages` for your ticket format (PROJ-XXX, #123, etc.)
- Create steering docs at `~/.kiro/steering/` for your product context
- Add a `.kiroignore` to repos to exclude `node_modules/`, `dist/`, `build/`

## Acknowledgements

This workflow is heavily based on [Matt Pocock's Skills](https://github.com/mattpocock/skills) ([aihero.dev/skills](https://www.aihero.dev/skills)). The main flow, rounds-based grilling, vertical-slice tickets, wayfinder, and writing-for-agents are all adapted from his work. MIT licensed.

## License

MIT
