# wkts

Interactive git worktree janitor for your whole machine.

Git worktrees pile up — agent sandboxes, Conductor workspaces, Codex checkouts,
`.claude/worktrees`, that branch you forgot about. `wkts` finds every linked
worktree under your home directory, shows how big and how stale each one is,
and lets you sweep them away in one pass.

```
NAME                     PROJECT             SIZE  LAST ACTIVE  LOCATION
riverdale                catalog             5.1G  2026-07-13   ~/conductor/workspaces/catalog/riverdale
TICKET-142               catalog             4.5G  2026-07-09   ~/Workspace/catalog/.claude/worktrees/TICKET-142
harbor                   catalog             4.3G  2026-06-29   ~/conductor/workspaces/catalog/harbor
northstar                billing-api         212M  2026-06-17   ~/conductor/workspaces/billing-api/northstar
...
[space] select   [enter] delete selected   [esc] quit
```

```
Removed: ~/conductor/workspaces/catalog/harbor
Removed: ~/conductor/workspaces/catalog/riverdale
...

✨ Done: 12 worktree(s), total removed 19.4GB
```

## How it works

- **Discovery**: linked worktrees are directories whose `.git` is a *file*
  pointing into `<main-repo>/.git/worktrees/<name>`. One bounded `find` over
  `$HOME` catches them all — no per-tool config, works with anything that
  creates worktrees (Conductor, Claude Code, Codex, plain `git worktree add`).
- **Size**: `du` per worktree, streamed live as each finishes. Results are
  cached in `~/.cache/wkts` and refreshed by a detached background job while
  you're in the picker, so runs after the first are near-instant (cached sizes
  show a `~` prefix).
- **Last active**: mtime of the worktree's git metadata (`index`/`HEAD`) —
  O(1) and touched by virtually any interaction. `WKTS_DEEP=1` scans every
  file's mtime instead (slow, thorough).
- **Deletion**: `git worktree remove --force`, falling back to `rm -rf` +
  `git worktree prune` when the metadata is stale. Always behind a y/N
  confirmation.

## Install

Requires [`fzf`](https://github.com/junegunn/fzf) and macOS (BSD `stat`/`date`).

```sh
git clone https://github.com/vaporwavie/wkts.git
ln -s "$PWD/wkts/wkts" ~/.local/bin/wkts   # or anywhere on your PATH
```

## Usage

```sh
wkts
```

- `space` — toggle a worktree for deletion
- `enter` — confirm & delete the selected worktrees
- `esc` — quit without touching anything

The preview pane shows each worktree's recent commits and dirty files.

## Configuration

| Env var      | Default | What it does                                          |
| ------------ | ------- | ----------------------------------------------------- |
| `WKTS_ROOT`  | `$HOME` | Directory to scan                                     |
| `WKTS_DEPTH` | `7`     | Max scan depth                                        |
| `WKTS_JOBS`  | `4`     | Parallel scan batch size (only throttles >32 worktrees) |
| `WKTS_DEEP`  | `0`     | `1` = compute last-active from every file's mtime     |

## License

MIT
