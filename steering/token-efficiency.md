# Token Efficiency

Rules to minimise context waste. Follow these in every session.

## Hard Rules

1. **Never read full lockfiles** — `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`. Use `git diff --stat` or `grep` for specific entries.
2. **`git diff --stat` before `git diff`** — preview what changed before pulling full diffs. Only read full diffs for files you need to understand.
3. **Cap large file reads** — if a file is 300+ lines and you only need a section, use offset/limit or the `code` tool for symbol lookup first.
4. **Never search or list inside `node_modules/`, `dist/`, or build output** — exclude these from `grep`, `glob`, `find`, and directory listings. The only exception is when the task explicitly requires inspecting an installed package.
5. **Prefer `code` tool symbol search over full file reads** — when looking for a specific function, class, or export.

## Patterns

### Git operations
```
✗ git_diff_unstaged (can return entire deleted files)
✓ git_status → git diff --stat → targeted git diff on specific files
```

### File reads
```
✗ Read 400-line file to find one function
✓ code tool → search_symbols("functionName") → read only the relevant lines
```

### Handoffs
```
✗ Re-read entire spec at start of new session
✓ Read handoff doc first — only read spec sections if handoff is insufficient
```
