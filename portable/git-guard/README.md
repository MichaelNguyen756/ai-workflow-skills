# git-guard

A git-layer safety boundary: `pre-push` and `pre-commit` hooks that refuse direct
pushes/commits to protected branches (`main`/`master`/`develop`/`qa`) and **all**
force-pushes — for every repo and every actor on the machine, human or agent.

## Why it exists

A prompt rule like "never push to main" is a *request* an agent is asked to honour, not
a boundary it cannot cross — especially an agent holding a general-purpose shell tool,
which can reach `git` directly. git-guard moves the rule from the prompt into git
itself: git runs the hook regardless of who (or what) typed the command, so the
protection holds even when an agent is instructed to bypass it.

Use it as defence-in-depth wherever an autonomous agent has shell/git access.

## Install

```sh
./install.sh
```

This sets a **global** `git config core.hooksPath` pointing at `~/.config/git-guard/hooks`,
so the guard applies to every repo at once. Idempotent — safe to re-run.

## Behaviour

- **pre-push** — rejects pushes to protected branches and any non-fast-forward (force) push to any branch.
- **pre-commit** — rejects commits made while `HEAD` is on a protected branch.
- **Chaining** — a repo's own `pre-push`/`pre-commit` are preserved as `<hook>.local` and run first, so existing lint/format/test hooks still fire.

## Configuration

| Env var | Purpose | Default |
|---|---|---|
| `GIT_GUARD_ALLOW=1` | Human override for a rare, deliberate manual intervention. Never give this to an agent. | (unset) |
| `GIT_GUARD_PROTECTED_BRANCHES` | Space-separated protected branch names. | `main master develop qa` |
| `GIT_GUARD_DIR` | Where hooks are installed. | `~/.config/git-guard/hooks` |

## Tests

```sh
bash test/git-guard.test.sh
```

Self-contained (throwaway repos in `$TMPDIR`, touches no global config).

## Limitation

git-guard is the unconditional floor for *git* actions. It does not constrain non-git
shell actions (e.g. `rm -rf`, cloud CLI calls). Pair it with tool-scoping on the agent
itself (allow-listed commands, path-scoped writes) to shrink the rest of the surface.
