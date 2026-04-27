# scripts

Personal shell scripts, symlinked to `/usr/local/bin` for global access.

## tmux-session

Attach to an existing tmux session or create a new one. If a tmuxinator config exists for the session it is used; otherwise a plain `tmux new-session` is created.

### Usage

```bash
tmux-session [--dir <directory>] [--name <session-name>] [--config <config-name>]
```

| Flag | Description |
|---|---|
| `--dir`, `-d` | Working directory passed to tmuxinator. Defaults to `$PWD`. |
| `--name`, `-n` | Session name. Defaults to the basename of `$PWD`. |
| `--config`, `-c` | Tmuxinator config name to use (filename without `.yml`). Defaults to the session name. |

### Examples

```bash
# Auto-derive everything from the current directory
cd ~/Work/Projects/printing_office
tmux-session
# → session: "printing_office", config: ~/.config/tmuxinator/printing_office.yml

# Worktree: different session, same tmuxinator config
cd ~/Work/Projects/printing_office-qb-payments
tmux-session --config printing-office
# → session: "printing_office-qb-payments", config: ~/.config/tmuxinator/printing-office.yml

# Explicit session name
tmux-session --name my-session --config printing-office
```

### How it works

1. Derives the **session name** from `--name` if provided, otherwise from `basename "$PWD"`.
2. Checks if a tmux session with that name already exists — if so, attaches to it.
3. Looks for `~/.config/tmuxinator/<config>.yml`.
   - Found → `tmuxinator start <config> --name=<session> -- <dir>`
   - Not found → `tmux new-session -s <session> -c <dir>`

### tmuxinator configs

Configs live in a separate repo at `~/tmuxinator-configs/`, with each project in its own subdirectory. Configs are symlinked into `~/.config/tmuxinator/`:

```bash
ln -s ~/tmuxinator-configs/printing-office/printing-office.yml ~/.config/tmuxinator/printing-office.yml
```

### Setup

```bash
# Symlink the script to make it globally available
ln -s ~/scripts/tmux-session /usr/local/bin/tmux-session
chmod +x ~/scripts/tmux-session
```
