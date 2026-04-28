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

---

## worktree

Manage Docker Compose services for a git worktree with isolated ports and database. Run from the worktree directory. Ports are derived deterministically from the directory name so worktrees never conflict with each other or with the main branch.

Requires `APP_PORT` and `DB_PORT` (projects with a db service) to be read from `.env` in the docker-compose files.

### Usage

```bash
worktree --start [--seed]
worktree --stop  [--dump|--clean]
```

| Flag | Description |
|---|---|
| `--start` | Write ports to `.env`, then `docker compose up -d`. |
| `--start --seed` | Start only the `db` service, restore a dump from the main DB, then start `web`. |
| `--stop` | `docker compose down` (keeps volumes). |
| `--stop --dump` | Dump the worktree DB to a `.sql` file, then stop. |
| `--stop --clean` | `docker compose down -v` (destroys volumes). |

### Environment variables

| Variable | Default | Description |
|---|---|---|
| `POSTGRES_USER` | `postgres` | PostgreSQL user. |
| `POSTGRES_PASSWORD` | `password` | PostgreSQL password. |
| `POSTGRES_DB` | read from compose config | Database name. |
| `MAIN_DB_PORT` | `5432` | Host port of the main branch DB, used by `--seed`. |

### Examples

```bash
# Start a fresh worktree
cd ~/Work/Projects/my-app-feature-x
worktree --start

# Start with a copy of the main branch data
worktree --start --seed

# Done — keep containers down but preserve the DB volume
worktree --stop

# Done — save a DB snapshot before tearing down
worktree --stop --dump

# Done — destroy everything
worktree --stop --clean
```

### How it works

1. Hashes the worktree directory name to derive unique `APP_PORT` (4000–4999) and `DB_PORT` (6000–6999).
2. Writes those ports to `.env` (creates the file if it doesn't exist).
3. Docker Compose reads them via `${APP_PORT:-<default>}` in the compose file, so host ports never clash.
4. Named volumes are scoped to the Compose project name (the directory name by default), so each worktree's DB data is automatically isolated.

### Setup

```bash
ln -s ~/scripts/worktree /usr/local/bin/worktree
chmod +x ~/scripts/worktree
```
