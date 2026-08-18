---
name: commit-messages
description: Use when creating commits, writing commit messages, staging changes, pushing code to a GitHub repository, or creating changeset files.
---
# Commit Message Convention

## Format

```
type(scope): description

[optional body]

[optional footer(s)]
```

If the project uses a ticket tracker, include the ticket reference:

```
type(scope): PROJ-123 | description
```

## Rules

- **type** (required): one of `feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `perf`, `test`, `ci`, `build`
- **scope** (optional): freeform, in parentheses — e.g., `feat(player):`, `fix(auth):`
- **ticket** (optional): issue tracker reference if the project uses one (e.g., `PROJ-123`, `#42`)
- **`|`** separates ticket from description (only when ticket is present)
- **description**: lowercase, imperative mood, no trailing period

## Examples

```
feat(player): add series stats table widget
fix: resolve race condition in request handler
chore: update dependencies
refactor(auth): extract token validation into utility

# With ticket references:
feat(player): PROJ-123 | add series stats table widget
fix: #42 | resolve race condition in request handler
```

## How to Commit

All commits are created via shell (signed automatically if configured):

```bash
git add <specific-files>
git commit -m "type(scope): description"
```

## How to Push

Always push to a feature branch:

```bash
git push -u origin <branch-name>
```

If a push is rejected: check `git status` and `git log --oneline -5`. Diagnose the cause. Ask the user for guidance if unsure.

## Changesets

### Detection

Before committing, check if the repo uses changesets by looking for `.changeset/config.json`. If it exists, a changeset file may be required.

### When a Changeset is Required

A changeset is needed when the commit includes **source code changes that affect the published package** — anything a consumer would notice (new feature, bug fix, behaviour change, API change, dependency update that changes runtime behaviour).

A changeset is **NOT needed** for:
- CI/CD workflow changes only
- Documentation-only changes (README, inline comments, storybook prose)
- Test-only changes (new tests, test refactors)
- Dev tooling changes (ESLint config, prettier, editor configs)
- Internal refactors with no public API or behaviour change

When unsure, ask: "Would a consumer of this package notice anything different?" If no → skip changeset.

### How to Create a Changeset

1. Read the package name from `package.json` (the `name` field)
2. Determine the bump type from the commit type:
   - `feat` → `minor`
   - `fix`, `perf` → `patch`
   - Breaking changes (indicated by `!` suffix or user confirmation) → `major`
   - `chore`, `refactor`, `style`, `build` → `patch` (if it affects published output)
3. Name the file based on the change: `<ticket-or-slug>-<short-description>.md` (lowercased, kebab-case). E.g., `proj-123-add-stats-table.md` or `add-responsive-breakpoints.md`.
4. Write the changeset file:

```markdown
---
"<package-name>": <bump-type>
---

<changeset summary>
```

5. Stage it alongside the other files: `git add .changeset/<filename>.md`

### Changeset Summary Style

Use the same format as the commit message:

```
type(scope): description
```

Or with ticket if the project uses one:

```
type(scope): PROJ-123 | description
```

This keeps the changelog consistent and traceable. Examples:
- `feat(player): add series stats table widget`
- `fix(tooltip): PROJ-123 | fix positioning when parent has overflow hidden`
- `chore: update styled-components peer dependency to v6`

### Agent Behaviour with Changesets

When a changeset is needed:
1. Draft the changeset content and present it to the user alongside the commit message
2. After user confirms, create the file and stage it with the commit

The changeset file is part of the same commit as the code change — not a separate commit.

## Agent Behaviour

1. **Verify you're on a feature branch** — if on `main`, create a branch first
2. **Stage specific files** — `git add <files>`, not `git add .`
3. **Check for changesets** — if `.changeset/config.json` exists and the change warrants it, draft a changeset file
4. **Draft the commit message** (and changeset if applicable) and confirm with the user before committing
5. **Commit via shell** — `git commit -m "message"`
6. **Push to the feature branch** — `git push -u origin <branch>`
7. **If something goes wrong** — diagnose, explain to the user, and ask for guidance. Do not escalate.
