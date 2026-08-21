# Kiro Crew

Kiro Crew is the asynchronous, autonomous layer that complements Kiro CLI. It runs as a local desktop app with its own memory, knowledge, and app ecosystem.

## Routing

| Signal | Route to |
|--------|----------|
| Code implementation (features, fixes, refactors) | **Kiro CLI** |
| Deploys and infrastructure operations | **Kiro CLI** |
| AWS operations, S3, filesystem, data tasks | **Kiro CLI** |
| Jira/Confluence access | **Kiro CLI** (has Atlassian MCP tools) |
| Git operations, PRs, commits | **Kiro CLI** |
| Spec → tickets → implement pipeline | **Kiro CLI** |
| PR review with persistent per-repo learning | **Kiro Crew** (Code Review Sage) |
| Multi-cycle background research | **Kiro Crew** (Research Lab) |
| Ops incident auto-response / alarm watching | **Kiro Crew** (Ops Mission Control) |
| GitHub issue triage across repos | **Kiro Crew** (Issue Radar) |
| Periodic background checks (heartbeat tasks) | **Kiro Crew** |
| Autonomous performance improvement cycles | **Kiro Crew** (Auto-Improvement) |

## Shared infrastructure

Kiro Crew reads from the same `~/.kiro/` directory as Kiro CLI:
- **Steering docs** (`~/.kiro/steering/`) — shared. Both systems load the same conventions.
- **Skills** (`~/.kiro/skills/`) — shared. Crew agents can invoke grilling, to-spec, research, etc.
- **Agent templates** — shared. Crew uses your agent definitions.

## What Kiro Crew cannot do

- Access AWS profiles or S3 buckets (no AWS CLI access)
- Bulk-ingest documents (knowledge builds organically through conversations)
- Merge PRs or push code (it stages draft reviews — you submit)
- Access Jira/Confluence directly (no Atlassian MCP connection)
- Write to your local filesystem outside its workspace (sandboxed)
- Read files from your project directories outside `~/.kiro/crew/workspace/`

## Scheduled jobs & heartbeat constraints

Scheduled jobs and heartbeat tasks can only use:
- `gh` CLI (GitHub PRs, issues, CI status, repo search) ✓
- Web browsing ✓
- Crew's own memory and knowledge store ✓

They CANNOT use:
- Jira/Confluence (no Atlassian MCP)
- AWS CLI
- Local filesystem reads
- Any tool requiring your terminal session

Keep scheduled prompts scoped to GitHub-accessible data.

## When to suggest Kiro Crew

| Trigger | Suggestion |
|---------|-----------|
| `/wayfinder` creates a research ticket | "This could run as a Research Lab campaign — it'll keep investigating while you work on other things." |
| `/research` question is broad (multiple sub-questions) | "This is bigger than a single research pass. Want to fire a Research Lab campaign instead?" |
| `/code-review` passes clean but PR has significant logic changes | "For a deeper design-level review, you could also run this through Code Review Sage." |
| User asks "what needs attention?" or "any stale PRs?" | "That's a good heartbeat task for Kiro Crew." |

Do NOT suggest Kiro Crew for: implementation, deploys, AWS ops, git operations, or anything requiring filesystem/tool access.

## Code Review Sage — how it learns

1. Does NOT bulk-ingest old PRs
2. Learns incrementally: when reviewing a fix-type PR, it traces back to the introducing commit and learns what class of defect was missed
3. Seed conventions manually at: `~/.kiro/crew/apps/code-review-sage/data/learnings/common/learned-patterns.md`
4. Candidate patterns stage in a `.candidate.md` file — a human triggers consolidation into the live ruleset

## Knowledge & Memory

- **Memory** (`memory.db`): semantic + episodic memory from conversations. Builds up over time.
- **Knowledge** (`knowledge.db`): vector store for reference material. Ingested through the Kiro Crew chat interface, not a CLI bulk import.
- **Learn** (`kirocrew learn add`): stores short correction strings. Not for document ingestion.

## Integration seams

1. **Research tickets** → fire a Research Lab campaign for deep, multi-cycle questions
2. **Code Review Sage** → review PRs through Kiro Crew, use findings in CLI
3. **Issue Radar** → triage in Kiro Crew, implement in Kiro CLI
4. **Heartbeat → CLI** — if a heartbeat task surfaces something actionable, pick it up in a CLI session
