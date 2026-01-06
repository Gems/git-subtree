# AGENTS.md — Copilot guardrails for this repository

This repository must remain a **single-script** POSIX shell implementation that provides convenient wrappers around `git subtree`, dispatched via symlinks.

If any instruction below conflicts with other project docs, **this file wins**.

## Non-negotiable constraints

1. **Single source of logic**
   - All logic lives in **one** POSIX-compatible shell script (bash is acceptable on macOS, but the script must remain POSIX-oriented).
   - Do **not** split logic across multiple scripts or introduce helper scripts.

2. **Symlink-based command dispatch**
   - The repository must include symlinks:
     - `git-sub-add`  → main script
     - `git-sub-pull` → main script
     - `git-sub-push` → main script
   - The script must determine behavior from `basename "$0"`.

3. **Repository deliverables**
   - `README.md` must explain: purpose, usage, configuration, installation (Homebrew + manual), alias registration, examples, design decisions, limitations.
   - A GitHub Actions workflow must publish as a Homebrew formula on **tagged releases** (e.g., `vX.Y.Z`) and install:
     - main script into `bin/`
     - the 3 symlinked commands

4. **No placeholders**
   - Do not add TODOs, stubs, pseudocode, or “replace me” sections.
   - Anything produced must be commit-ready.

## Shell script: safety & structure

The script must implement:

- Strict mode: `set -euo pipefail` **or the closest portable equivalent**.
- `err()` function:
  - prints to stderr
  - exits non-zero
- Per-command `usage()` output (or per-command usage functions).
- Validate running inside a git repo for all commands **except** `--install`.
- Determine repo root via: `git rev-parse --show-toplevel`.
- Must work from any subdirectory (operate relative to repo root).
- Must echo the exact `git subtree ...` command before running it:
  - prefix with `+ `
  - suppress if `QUIET=1`

## Global alias install command (`--install`)

Supported forms:

- `git-sub-add --install`
- `git-sub-pull --install`
- `git-sub-push --install`
- `git-subtree-helper --install` (the underlying script name)

Behavior:

- Must work **outside** a git repository.
- Register **global git aliases** using `git config --global`:
  - `sub-add  = !git-sub-add`
  - `sub-pull = !git-sub-pull`
  - `sub-push = !git-sub-push`
- Detect absolute path of the installed executable:
  - use `command -v` and/or `$0`
  - resolve to an absolute path (not relative)
- Ensure aliases point to the resolved executable path (idempotent).
- Re-running `--install` must not duplicate or corrupt config.
- Print which aliases were installed/updated.

After install, users must be able to run:

- `git sub-add`
- `git sub-pull`
- `git sub-push`

## Configuration file format & rules

Config file path (per-repo):

- `<repo-root>/.git/subtree.conf`

Format: INI-like (no external parsers). `awk/sed/grep` allowed.

Example sections:
```
[subtree “commons”]
path=libs/commons
url=git@bitbucket.org:org/commons.git
branch=main
squash=true
```

Rules:

- Multiple subtrees supported.
- `squash` defaults to `true` if missing.
- Users must not need to retype URLs once stored.
- Only `subtree add` is allowed to modify the config file.
- Updates must be atomic: write temp → `mv` replace.

## Command behaviors (must match exactly)

### `git-sub-add [name] [options]`

Options:
- `--path <path>`
- `--url <url>`
- `--branch <branch>`
- `--no-squash`
- `--force`
- `--install`

Behavior:
- If `--install` present: run alias install and exit.
- If `name` already exists and `--force` not provided: error.
- Run:
  - `git subtree add --prefix <path> <url> <branch> --squash` **only if** squash=true
  - If squash=false, do not pass `--squash`.
  - After successful add, **always** run `git subtree split --prefix <path> --rejoin` to optimize future operations.
- After success, persist `path`, `url`, `branch`, `squash` to `subtree.conf`.
- Create `subtree.conf` if missing.

### `git-sub-pull [name]`

Options:
- `--install`

Behavior:
- If `--install` present: run alias install and exit.
- Run:
  - `git subtree pull --prefix <path> <url> <branch>`
- Include `--squash` **IFF**:
  - config has `squash=true`, or
  - `squash` key is missing
- After successful pull, **always** run `git subtree split --prefix <path> --rejoin` to optimize future operations.
- Never modify `subtree.conf`.

### `git-sub-push [name]`

Options:
- `--install`

Behavior:
- If `--install` present: run alias install and exit.
- Run:
  - `git subtree push --prefix <path> <url> <branch>`
- Never use `--squash`.

## Name auto-detection (when `[name]` omitted)

If `name` is missing:

1. Compute repo root.
2. Determine the **first-level directory under repo root** for the current working directory.
   - Example: repo=`/r`, pwd=`/r/libs/commons/src` → first-level dir=`libs`
3. Match subtrees whose `path` starts with that directory.
4. If exactly one match: use it.
5. If zero or multiple matches: `err` with:
   - explanation
   - list of candidate subtree names
   - instruction to pass `[name]`

## Output & UX invariants

- Errors must be clear and actionable (use `err`).
- Commands must display the exact `git subtree ...` command before execution with `+ ` prefix.
- Respect `QUIET=1` to suppress command echoing.
- Do not add interactive prompts.

## README requirements (enforced)

README must include (concise, technical):

- Problem solved vs raw `git subtree`
- Installation:
  - Homebrew
  - Manual install
  - `--install` alias registration
- Quick start
- `subtree.conf` format explanation
- Examples:
  - add / pull / push
  - auto-detection behavior
  - global aliases (`git sub-add`, etc.)
- Design decisions:
  - single script
  - symlink dispatch
  - squash semantics
- Limitations & assumptions

No marketing language.

## Homebrew distribution requirements (enforced)

GitHub Actions workflow must:

- Run on tagged releases (`vX.Y.Z`).
- Generate a Homebrew formula using the release artifact checksum.
- Install:
  - main script to `bin/`
  - symlinks (`git-sub-add`, `git-sub-pull`, `git-sub-push`)
- Result must allow:
  - `brew install <toolname>`

## Change policy

When making changes:

- Do not introduce additional languages, runtimes, or dependencies.
- Do not move logic out of the single script.
- Do not change CLI semantics, squash rules, config path, or auto-detection rules unless this file is updated first.
- 
