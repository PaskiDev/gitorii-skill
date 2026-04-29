# gitorii-skill

Claude Code skill that forces exclusive use of [`torii`](https://github.com/paskidev/gitorii) (the Gitorii CLI) for all version control operations. Claude will never invoke `git` directly — every operation is translated to its `torii` equivalent.

## What it does

When active, Claude:

- Translates every `git` command you mention into the matching `torii` command and runs that instead.
- Never suggests `git` in chat, scripts, hooks, or CI configs.
- Stops and asks when an operation has no `torii` equivalent — never falls back to raw `git`.
- Writes commit messages via `torii save -m "..."`, including HEREDOC for multi-line.

## Install

Clone into your Claude Code skills directory:

```bash
torii clone github paskidev/gitorii-skill ~/.claude/skills/gitorii
```

Or with git, if torii is not installed yet (chicken-and-egg, only this once):

```bash
git clone https://github.com/paskidev/gitorii-skill ~/.claude/skills/gitorii
```

Then install [`torii`](https://github.com/paskidev/gitorii):

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/paskidev/gitorii/releases/latest/download/gitorii-installer.sh | sh
```

## Activate

In any Claude Code session, the skill auto-triggers when you:

- Open a repo that already uses torii.
- Say "use torii", "use gitorii", "no git", "torii only".
- Invoke `/gitorii`.

It stays active for the whole session until you say `stop gitorii` or `normal mode`.

## What's included

- Full translation table: core, branches, log, stash/snapshot, tags, remote, history rewrite, multi-repo workspace.
- Hard rules — Claude will refuse to invoke `git` even on edge cases (failed hooks, detached HEAD, CI configs).
- Boundary clarification — reading torii's own Rust source (which uses git2/libgit2) is fine; the ban is on shell-level `git` calls.

See [SKILL.md](SKILL.md) for the full spec.

## Why

Gitorii replaces a dozen `git` incantations with simpler, safer commands:

| git | torii |
|-----|-------|
| `git add . && git commit -m "msg"` | `torii save -am "msg"` |
| `git pull && git push` | `torii sync` |
| `git switch -c branch` | `torii branch <name> -c` |
| `git stash push -u` | `torii snapshot stash -u` |
| Push to GitHub + GitLab + Codeberg | `torii mirror sync` |

Plus features git lacks: pre-commit secret scanner, named persistent snapshots, multi-platform mirrors, multi-repo workspaces, conventional-commits auto-tagging.

This skill ensures Claude uses all of it, every time.

## License

MIT — see [LICENSE](LICENSE).

## Author

Built by [Pasqual Peñalver Collado](https://paski.dev) (PaskiDev).
