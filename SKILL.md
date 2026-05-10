---
name: gitorii-skill
description: >
  Force exclusive use of the `torii` CLI (Gitorii) for all version control. Never call `git`
  directly. Translates every git operation — status, add, commit, push, pull, branch, log,
  diff, stash, rebase, tag, blame, clone, remote, fsck, scan — into the equivalent `torii`
  command. Auto-triggers in repos using torii or when user says "use torii", "use gitorii",
  "no git", or invokes /gitorii. Stays active until "stop gitorii" or "normal mode".
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
| `git init` | `torii init` (creates `main` by default + scaffolds `.toriignore` and `policies/commits.toml`) |
| `git status` | `torii status` |
| `git add <file>` | (folded into save) `torii save <file> -m "..."` |
| `git add . && git commit -m "msg"` | `torii save -am "msg"` |
| `git commit -m "msg"` | `torii save -m "msg"` |
| `git commit --amend -m "msg"` | `torii save --amend -m "msg"` |
| `git revert <hash>` | `torii save --revert <hash> -m "revert"` |
| `git reset --soft HEAD~1` | `torii save --reset HEAD~1 --reset-mode soft` |
| `git reset --mixed HEAD~1` | `torii save --reset HEAD~1 --reset-mode mixed` |
| `git reset --hard HEAD~1` | `torii save --reset HEAD~1 --reset-mode hard` |
| `git rm --cached <file>` | `torii save --unstage <file>` |
| `git pull && git push` | `torii sync` |
| `git pull` | `torii sync --pull` |
| `git push` | `torii sync --push` |
| `git push --force` | `torii sync --force` |
| `git fetch` | `torii sync --fetch` |
| `git diff` | `torii diff` |
| `git diff --staged` | `torii diff --staged` |
| `git diff HEAD~1` | `torii diff --last` |
| `git show HEAD` | `torii show` |
| `git show <hash>` | `torii show <hash>` |

### Branches

| git | torii |
|-----|-------|
| `git branch` | `torii branch` |
| `git branch -a` | `torii branch --all` |
| `git switch -c <name>` / `git checkout -b <name>` | `torii branch <name> -c` |
| `git checkout --orphan <name>` | `torii branch <name> -c --orphan` |
| `git switch <name>` / `git checkout <name>` | `torii branch <name>` |
| `git branch -d <name>` | `torii branch -d <name>` |
| `git branch -m <new>` | `torii branch --rename <new>` |
| `git push origin --delete <name>` | `torii branch --delete-remote <name>` |
| `git merge <branch>` | `torii sync <branch> --merge` |
| `git rebase <branch>` | `torii sync <branch> --rebase` (or `torii history rebase <branch>`) |
| `git rebase -i HEAD~5` | `torii history rebase -i HEAD~5` |
| `git rebase -i --root` | `torii history rebase --root -i` |
| `git rebase --continue` | `torii history rebase --continue` |
| `git rebase --abort` | `torii history rebase --abort` |
| `git rebase --skip` | `torii history rebase --skip` |

### Interactive rebase with `--todo-file`

`torii history rebase --todo-file plan.txt [--root | <base>]` accepts an
extended grammar: each `reword <sha>` line takes an inline message:

```
reword 1a2b3c4 feat: clean subject without trailers
pick   d5e6f7g
```

When git pauses on `edit`, torii detects the pause and prints
`torii history rebase --continue` instead of git's instructions.

### History / log

| git | torii |
|-----|-------|
| `git log` | `torii log` |
| `git log -n 50` | `torii log -n 50` |
| `git log --oneline --graph` | `torii log --oneline --graph` (always on in TUI Log view) |
| `git log --author X` | `torii log --author X` |
| `git log --grep X` | `torii log --grep X` |
| `git log --stat` | `torii log --stat` |
| `git log --since 2026-01-01` | `torii log --since 2026-01-01` |
| `git reflog` | `torii log --reflog` |
| `git blame <file>` | `torii blame <file>` |
| `git blame -L 10,30 <file>` | `torii blame <file> --lines 10-30` |
| `git cherry-pick <hash>` | `torii cherry-pick <hash>` |
| `git cherry-pick --continue` | `torii cherry-pick --continue` |
| `git cherry-pick --abort` | `torii cherry-pick --abort` |

### Recovery — `git fsck`

| git | torii |
|-----|-------|
| `git fsck --unreachable --no-reflogs` | `torii history fsck` |
| `git cat-file -p <oid>` | `torii history fsck --show <oid>` (accepts short OID) |
| (manual recovery) | `torii history fsck --restore <oid> --to <path>` |

`torii history fsck` is the canonical tool for recovering work lost to a
destructive op (reset --hard, force-push, rebase). Lists orphaned
commits/blobs/trees with size + content preview.

### Stash / snapshots

| git | torii |
|-----|-------|
| `git stash` | `torii snapshot stash` |
| `git stash -u` | `torii snapshot stash -u` |
| `git stash pop` | `torii snapshot unstash` |
| `git stash apply` | `torii snapshot unstash <id> --keep` |
| (no git equiv) | `torii snapshot create -n "name"` — named persistent saves |
| (no git equiv) | `torii snapshot undo` — undo last torii operation |

> **Caveat:** `torii snapshot stash` has a known bug where it sometimes
> reports success without actually saving. After `stash`, verify with
> `torii status` (working tree should be clean) before destructive ops.
> If unsure, use `torii snapshot create -n "wip"` instead.

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
| `gh repo create <name>` | `torii remote create github <name> --public` |
| `gh repo create org/<name>` | `torii remote create github org/<name>` (owner/repo syntax) |
| GitLab subgroup repo | `torii remote create gitlab eng/web/api` |
| Multi-platform create + push | `torii remote create github,gitlab acme/repo --push` |
| (no git equiv) | `torii mirror sync` — push to GitHub+GitLab+Codeberg etc at once |

### History rewrite / cleanup

| git | torii |
|-----|-------|
| `git filter-branch --tree-filter 'rm -f X'` / `git filter-repo` | `torii history remove-file <file>` |
| `git gc --prune=now && git reflog expire` | `torii history clean` |
| (no git equiv) | `torii history rewrite "<start>" "<end>"` — rewrite commit dates |
| (no git equiv) | `torii scan` / `torii scan --history` — secret scanner |
| (no git equiv) | `torii scan --commits` — enforce policies/commits.toml on commit history |

### Multi-repo

| git | torii |
|-----|-------|
| (no git equiv) | `torii workspace add <name> <path>` |
| Loop `git status` over N repos | `torii workspace status <name>` |
| Loop `git commit` over N repos | `torii workspace save <name> -am "msg" --all` |
| Loop `git pull && git push` over N repos | `torii workspace sync <name>` |

### PRs / Issues (alternative to `gh` / `glab`)

| git / gh | torii |
|----------|-------|
| `gh pr list` | `torii pr list` |
| `gh pr create` | `torii pr create` |
| `gh pr view <n>` | `torii pr view <n>` |
| `gh pr merge <n>` | `torii pr merge <n>` |
| `gh issue list` | `torii issue list` |
| `gh issue create` | `torii issue create` |
| `gh issue close <n>` | `torii issue close <n>` |

### Ignore rules

| git | torii |
|-----|-------|
| `echo 'pat' >> .gitignore` | `torii ignore add 'pat'` |
| (no git equiv) | `torii ignore secret 'pat'` — adds to `.toriignore.local` (machine-private) |
| (no git equiv) | `torii ignore list` |
| `.gitignore` (file) | `.toriignore` (committed) + `.toriignore.local` (machine-private) |

### Cloud / API key (gitorii.com)

| operation | torii |
|-----------|-------|
| Save API key | `torii auth login` (prompts) or `torii auth login --key gitorii_sk_…` |
| Show org / plan | `torii auth status` (or `torii auth whoami`) |
| Delete local key | `torii auth logout` |
| Override per-process | `TORII_API_KEY=gitorii_sk_… torii …` |
| Custom endpoint | `TORII_API_ENDPOINT=https://… torii …` (or `--endpoint` on login) |

Key lives at `~/.config/torii/auth.toml` (chmod 600 on Unix).

### Commit policy enforcement

`policies/commits.toml` (TOML, scaffolded by `torii init`) drives
`torii scan --commits`:

```toml
forbid_trailers      = ["Co-Authored-By:.*Claude", "Co-Authored-By:.*Copilot"]
require_trailers     = ["Signed-off-by:"]
forbid_subjects      = ["^(wip|tmp|temp)$"]
author_email_matches = ".*@example\\.com$"
subject_max_length   = 72
subject_min_length   = 8
require_conventional = true
```

| operation | torii |
|-----------|-------|
| Lint last 200 commits | `torii scan --commits` |
| Lint custom range | `torii scan --commits --limit 50` |
| Custom policy file | `torii scan --commits --policy-file path/to.toml` |

Exit code 1 on any violation → CI-ready.

## TUI

| operation | how |
|-----------|-----|
| Open dashboard | `torii` (no args) or `torii tui` |
| Navigate | Tab → sidebar; j/k or ↑/↓; Enter to open |
| Quit | `q` or Ctrl+C |
| Help | `?` |
| Toggle event log | `e` |
| Open repo picker (multi-workspace) | `W` |

The Log view always renders the commit graph (5 styles selectable in
Settings → Appearance → graph style: ascii / curves / heavy / bubbles /
bubbles-x). View-switcher hotkeys (`g`, `l`, `b`, etc.) were removed
in 0.6.1 — they conflicted with in-view keys; navigate via the sidebar.

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
- **Submodules / worktrees / sparse-checkout** → torii has no direct command. Tell user, do not fall back to git.
- **`.gitignore`** → still valid file. Also offer `torii ignore add <pattern>` and `.toriignore` / `.toriignore.local` for richer rules (secrets, size limits, hooks).
- **Recovering destroyed work** → `torii history fsck` first. Lists unreachable objects with content preview; `--restore <oid> --to <path>` writes blobs back to disk.
- **Push silently succeeded but remote unchanged** → upgrade to torii ≥ 0.6.1. Earlier versions hid server-side rejections (branch protection, pre-receive hooks).

## Boundaries

- Reading torii's own Rust source (`/home/outsider/repos/gitorii/`) is fine — that codebase calls git2/libgit2 internally and that is correct. The ban is on **you** invoking `git` as a shell command on behalf of the user.
- "stop gitorii" / "normal mode": revert to allowing git.
- Skill stays active across the session until explicitly disabled.
