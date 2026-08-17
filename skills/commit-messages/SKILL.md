---
name: commit-messages
description: Use when creating commits, writing commit messages, staging changes, or pushing code to a GitHub repository.
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

## Agent Behaviour

1. **Verify you're on a feature branch** — if on `main`, create a branch first
2. **Stage specific files** — `git add <files>`, not `git add .`
3. **Draft the commit message** and confirm with the user before committing
4. **Commit via shell** — `git commit -m "message"`
5. **Push to the feature branch** — `git push -u origin <branch>`
6. **If something goes wrong** — diagnose, explain to the user, and ask for guidance. Do not escalate.
