# .envrc + direnv: Reference Guide

## TL;DR
- `.envrc` is a bash script auto-loaded by `direnv` per directory.
- Add the shell hook, create `.envrc`, **export** vars, then `direnv allow .`.
- Re-run `direnv allow .` after every change.
- Keep secrets in private files ignored by Git.

---

## Install and Hook

```bash
# macOS
brew install direnv

# Ubuntu/Debian
sudo apt-get install direnv

# Hook (choose your shell)
# Bash
echo 'eval "$(direnv hook bash)"' >> ~/.bashrc
# Zsh
echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc
# Fish
echo 'direnv hook fish | source' >> ~/.config/fish/config.fish
```

Restart the shell.

---

## Quick Start

```bash
# project/.envrc
export NODE_ENV=development
export DATABASE_URL=postgresql://localhost:5432/myproject
PATH_add bin
dotenv_if_exists .env.local
```

Approve:
```bash
direnv allow .
```

---

## Core Concepts

- **Auto load/unload** on `cd` into and out of the directory.
- **Bash semantics**: functions, conditionals, command execution allowed.
- **Allow-list security**: `.envrc` is blocked until you approve it.
- **Stdlib helpers**: `PATH_add`, `dotenv`, `dotenv_if_exists`, `source_up`, `layout`, `use`, `has`.

---

## Common Patterns

### Load dotenv files
```bash
dotenv                # .env if present
dotenv .env.development
dotenv_if_exists .env.local
```

### Manage PATH safely
```bash
PATH_add bin
PATH_add node_modules/.bin
```

### Conditional logic
```bash
if has psql; then
  export DB_CLI=psql
fi

if [[ -f ".env.staging" ]]; then
  dotenv .env.staging
fi
```

### Language environments
```bash
# Python (virtualenv via direnv’s layout)
layout python python3.11

# Node, Ruby, etc. via version managers
# Requires corresponding manager integrations (asdf, nvm, pyenv)
use node 20.11.1
use ruby 3.2.2
```

### Pull from parent dir
```bash
# In a subproject
source_up .envrc
PATH_add local-bin
```

---

## Security

### Allow system
```text
direnv: error .envrc is blocked. Run `direnv allow` to approve its content
```
Approve or re-approve:
```bash
direnv allow .
```

### Secrets hygiene
- Keep secrets in a separate file. Example:
  ```bash
  # .envrc
  source_env_if_exists .envrc.private
  ```
- Ignore secret files in Git.

### File permissions
```bash
chmod 644 .envrc
```

---

## Team Setup

### Recommended repo layout
```
project/
├─ .envrc              # Main env config (no secrets)
├─ .envrc.sample       # Template for teammates
├─ .env                # Default env vars (non-secret)
├─ .env.development    # Dev-specific vars
├─ .env.staging        # Staging vars
└─ .gitignore          # Ignore secret and local files
```

### Collaboration tips
- Commit `.envrc.sample`, not your real `.envrc`.
- Document required variables in the sample.
- Optionally:
  ```bash
  # .envrc
  source_env .envrc.sample
  ```

---

## .gitignore

```gitignore
# Direnv
.direnv/
.envrc.private

# Environment files
.env.local
.env.*.local
```

---

## Troubleshooting

### Checklist
- Variables not loading
  - Use `export VAR=...`
  - Shell hook added and shell restarted
  - Run `direnv allow .`
- Permission errors
  - Re-run `direnv allow .`
  - Check `ls -la .envrc`
- Slow or stuck
  - Keep scripts fast
  - Avoid network calls in `.envrc`

### Useful commands
```bash
direnv status      # Show status and allow/deny info
direnv reload      # Force reload
direnv export bash # Show environment diff
direnv edit .      # Edit and auto-allow on save
```

---

## Advanced

### Nested environments
**Parent `.envrc`:**
```bash
export PROJECT_ROOT=$PWD
PATH_add bin
```
**Child `.envrc`:**
```bash
source_up .envrc
export SUBPROJECT_NAME=api
PATH_add local-bin
```

### Custom functions (~/.config/direnv/direnvrc)
```bash
layout_docker() {
  export COMPOSE_PROJECT_NAME="$(basename "$PWD")"
  export DOCKER_BUILDKIT=1
}
```

### Version managers
- `use node ...` with nvm/asdf.
- `use ruby ...` with rbenv/asdf.
- `layout python python3.x` or `layout python-venv python3.x` with pyenv + virtualenv.

---

## Performance
- Target <500 ms execution.
- Cache heavy results to files under `.direnv/`.
- Avoid subshell loops and network calls.

---

## Templates

### `.envrc.sample`
```bash
# Non-secret defaults and structure for teammates

# Load base dotenv if present
dotenv_if_exists .env

# Example required vars
export APP_NAME=myapp
export PORT=3000

# Paths
PATH_add bin
PATH_add node_modules/.bin

# Language toolchains (uncomment if used)
# layout python python3.11
# use node 20.11.1

# Optional local overrides (not committed)
source_env_if_exists .envrc.private
dotenv_if_exists .env.local
```

### `.envrc.private` (do not commit)
```bash
export DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
export API_KEY=...redacted...
```
