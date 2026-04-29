---
name: gitorii-skill
description: >
  Force exclusive use of the `torii` CLI (Gitorii) for all version control. Never call `git`
  directly. Translates every git operation — status, add, commit, push, pull, branch, log,
  diff, stash, rebase, tag, blame, clone, remote — into the equivalent `torii` command.
  Auto-triggers in repos using torii or when user says "use torii", "use gitorii", "no git",
  or invokes /gitorii. Stays active until "stop gitorii" or "normal mode".
---

Exclusive `torii` mode. `git` forbidden. Every VCS action goes through `torii`. If no torii
equivalent exists, say so — do NOT fall back to `git`.

## Hard rules

1. **Never invoke `git` in Bash.** Not `git status`, not `git log`, not `git diff`. Zero exceptions.
2. **Never suggest `git` commands** in chat, comments, docs, scripts, or CI configs the user is editing.
3. **Translate first, ask second.** If user types a `git` command, run the `torii` equivalent without asking.
4. **Unknown mapping → stop.** If a git operation has no `torii` equivalent, tell the user and wait. Do not improvise with raw `git`.
5. **`gh` / platform CLIs allowed** for PR/issue work that torii does not cover. `git` itself still forbidden.
6. **Hooks / scripts** the user asks you to write must call `torii`, not `git`.

## Translation table

### Core

| git | torii |
|-----|-------|
| `git init` | `torii init` |
| `git status` | `torii status` |
| `git add <file>` | (folded into save) `torii save <file> -m "..."` |
| `git add . && git commit -m "msg"` | `torii save -am "msg"` |
| `git commit -m "msg"` | `torii save -m "msg"` |
| `git commit --amend -m "msg"` | `torii save --amend -m "msg"` |
| `git revert <hash>` | `torii save --revert <hash> -m "revert"` |
| `git reset --soft HEAD~1` | `torii save --reset HEAD~1 --reset-mode soft` |
| `git reset --mixed HEAD~1` | `torii save --reset HEAD~1 --reset-mode mixed` |
| `git reset --hard HEAD~1` | `torii save --reset HEAD~1 --reset-mode hard` |
| `git pull && git push` | `torii sync` |
| `git pull` | `torii sync --pull` |
| `git push` | `torii sync --push` |
| `git push --force` | `torii sync --force` |
| `git fetch` | `torii sync --fetch` |
| `git diff` | `torii diff` |
| `git diff --staged` | `torii diff --staged` |
| `git show HEAD` | `torii show` |
| `git show <hash>` | `torii show <hash>` |

### Branches

| git | torii |
|-----|-------|
| `git branch` | `torii branch` |
| `git branch -a` | `torii branch --all` |
| `git switch -c <name>` / `git checkout -b <name>` | `torii branch <name> -c` |
| `git switch <name>` / `git checkout <name>` | `torii branch <name>` |
| `git branch -d <name>` | `torii branch -d <name>` |
| `git branch -m <new>` | `torii branch --rename <new>` |
| `git merge <branch>` | `torii sync <branch> --merge` |
| `git rebase <branch>` | `torii sync <branch> --rebase` (or `torii history rebase <branch>`) |
| `git rebase -i HEAD~5` | `torii history rebase -i HEAD~5` |
| `git rebase --continue` | `torii history rebase --continue` |
| `git rebase --abort` | `torii history rebase --abort` |

### History / log

| git | torii |
|-----|-------|
| `git log` | `torii log` |
| `git log -n 50` | `torii log -n 50` |
| `git log --oneline --graph` | `torii log --oneline --graph` |
| `git log --author X` | `torii log --author X` |
| `git log --grep X` | `torii log --grep X` |
| `git log --stat` | `torii log --stat` |
| `git reflog` | `torii log --reflog` |
| `git blame <file>` | `torii blame <file>` |
| `git cherry-pick <hash>` | `torii cherry-pick <hash>` |

### Stash / snapshots

| git | torii |
|-----|-------|
| `git stash` | `torii snapshot stash` |
| `git stash -u` | `torii snapshot stash -u` |
| `git stash pop` | `torii snapshot unstash` |
| `git stash apply` | `torii snapshot unstash <id> --keep` |
| (no git equiv) | `torii snapshot create -n "name"` — named persistent saves |
| (no git equiv) | `torii snapshot undo` — undo last torii operation |

### Tags

| git | torii |
|-----|-------|
| `git tag` | `torii tag list` |
| `git tag -a v1.0.0 -m "msg"` | `torii tag create v1.0.0 -m "msg"` |
| `git tag -d v1.0.0` | `torii tag delete v1.0.0` |
| `git push --tags` | `torii tag push` |
| `git push origin v1.0.0` | `torii tag push v1.0.0` |
| (no git equiv) | `torii tag create --release` — auto-bump from Conventional Commits |

### Remote / clone

| git | torii |
|-----|-------|
| `git clone <url>` | `torii clone <url>` |
| `git clone git@github.com:u/r.git` | `torii clone github u/r` |
| `git remote -v` | `torii config list --local` (filter remote.*) |
| `gh repo create` | `torii remote create github <name> --public` |
| (no git equiv) | `torii mirror sync` — push to GitHub+GitLab+Codeberg etc at once |

### History rewrite / cleanup

| git | torii |
|-----|-------|
| `git filter-branch --tree-filter 'rm -f X'` / `git filter-repo` | `torii history remove-file <file>` |
| `git gc --prune=now && git reflog expire` | `torii history clean` |
| (no git equiv) | `torii history rewrite "<start>" "<end>"` — rewrite commit dates |
| (no git equiv) | `torii scan` / `torii scan --history` — secret scanner |

### Multi-repo

| git | torii |
|-----|-------|
| (no git equiv) | `torii workspace add <name> <path>` |
| Loop `git status` over N repos | `torii workspace status <name>` |
| Loop `git commit` over N repos | `torii workspace save <name> -am "msg" --all` |
| Loop `git pull && git push` over N repos | `torii workspace sync <name>` |

## When user types `git ...`

Translate silently and run `torii`. Mention the mapping once per session, then stop narrating it.

Example:
> User: "run git status"
> You: `torii status` (run it, show output)

## Commit messages

Follow Conventional Commits. Pass via `torii save -m "..."`. Never `git commit`. Never add Anthropic / Claude attribution.

For multi-line commit messages with HEREDOC, use `torii save`:

```bash
torii save -am "$(cat <<'EOF'
feat(auth): add token refresh

Why: sessions expired mid-request after 1h.
EOF
)"
```

## Edge cases

- **Hooks failed on save** → fix root cause, re-run `torii save`. Never `--no-verify` workaround with raw git.
- **Detached HEAD / weird state** → `torii status` first. If torii cannot resolve, report to user. Do NOT reach for raw git as escape hatch.
- **CI configs / scripts** the user edits → write `torii` commands inside them only if the CI runner has torii installed; otherwise warn user and stop. Do not silently emit `git` to "make it work".
- **Submodules / worktrees / sparse-checkout** → torii has no direct command (as of 0.5.0). Tell user, do not fall back to git.
- **`.gitignore`** → still valid file. Also offer `torii ignore add <pattern>` and `.toriignore` / `.toriignore.local` for richer rules (secrets, size limits, hooks).

## Boundaries

- Reading torii's own Rust source (`/home/outsider/repos/gitorii/`) is fine — that codebase calls git2/libgit2 internally and that is correct. The ban is on **you** invoking `git` as a shell command on behalf of the user.
- "stop gitorii" / "normal mode": revert to allowing git.
- Skill stays active across the session until explicitly disabled.
