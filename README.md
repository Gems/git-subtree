# git-subtree

A convenient bash wrapper around `git subtree` that simplifies managing multiple subtrees using a configuration file.

## Problem

Git subtree is powerful for managing nested repositories, but using it manually is cumbersome:
- Long, repetitive commands with many options
- Hardto remember URLs, paths, and branches for each subtree
- No way to standardize squash behavior across a project
- Easy to make mistakes with `--prefix` paths or forget `--squash`

## Solution

This tool provides:
- **Configuration file**: Store all subtree settings in `.git/subtree.conf` (INI format)
- **Simple commands**: `git-sub-add`, `git-sub-pull`, `git-sub-push` with minimal arguments
- **Auto-detection**: Automatically detect which subtree you're working with based on your current directory
- **Consistent behavior**: Enforce squash settings per subtree
- **Git integration**: Optional git aliases (`git sub-add`, etc.)

## Solution

This tool provides:
- **Configuration file**: Store all subtree settings in `.git/subtree.conf` (INI format)
- **Simple commands**: `git-sub-add`, `git-sub-pull`, `git-sub-push` with minimal arguments
- **Auto-detection**: Automatically detect which subtree you're working with based on your current directory
- **Consistent behavior**: Enforce squash settings per subtree
- **Git integration**: Optional git aliases (`git sub-add`, etc.)

## Quick Start

### 1. Install

**Via Homebrew** (coming soon):
```bash
brew install Gems/tap/git-subtree
```

**Manual installation**:
```bash
git clone https://github.com/Gems/git-subtree.git
cd git-subtree
./bin/git-subtree-helper --install
```

This creates git aliases `sub-add`, `sub-pull`, and `sub-push` globally.

### 2. Add Your First Subtree

From your project repository:

```bash
git sub-add mylib \
  --path vendor/mylib \
  --url https://github.com/user/mylib.git \
  --branch main
```

This will:
1. Run `git subtree add --prefix vendor/mylib --squash https://github.com/user/mylib.git main`
2. Save the configuration to `.git/subtree.conf`

### 3. Pull Updates

```bash
git sub-pull mylib
```

Or, from within the subtree directory:
```bash
cd vendor/mylib
git sub-pull  # Auto-detects "mylib"
```

### 4. Push Changes Upstream

```bash
git sub-push mylib
```

## Configuration Format

The configuration file is located at `<repo-root>/.git/subtree.conf` and uses INI format:

```ini
[subtree "name"]
path = local/path/to/subtree
url = https://github.com/user/repo.git
branch = main
squash = true
```

### Configuration Keys

- **path** (required): Local path where the subtree is located
- **url** (required): Git repository URL  
- **branch** (required): Branch to track (default: `main`)
- **squash** (optional): Whether to squash commits (default: `true`)

### Example Configuration

```ini
[subtree "mylib"]
path = vendor/mylib
url = https://github.com/user/mylib.git
branch = main
squash = true

[subtree "docs"]
path = docs/external
url = https://github.com/company/docs.git
branch = master
squash = false

[subtree "auth"]
path = src/components/auth
url = https://github.com/company/auth-module.git
branch = v2
squash = true
```

## Commands

### git-sub-add

Add a new subtree and save its configuration.

**Syntax**:
```bash
git-sub-add [name] [options]
```

**Options**:
- `--path PATH`: Local path for the subtree (required for new subtrees)
- `--url URL`: Repository URL (required for new subtrees)
- `--branch BRANCH`: Branch to track (default: `main`)
- `--no-squash`: Disable squash mode (squash is enabled by default)
- `--force`: Overwrite existing configuration

**Examples**:
```bash
# Add a new subtree
git-sub-add mylib --path vendor/mylib --url https://github.com/user/mylib.git

# Add with specific branch
git-sub-add docs --path docs/external --url https://github.com/company/docs.git --branch v1.0

# Add without squashing
git-sub-add nosquash --path lib/nosquash --url https://github.com/x/y.git --no-squash

# Use existing config (if mylib already configured)
git-sub-add mylib
```

### git-sub-pull

Pull updates from a subtree.

**Syntax**:
```bash
git-sub-pull [name]
```

**Examples**:
```bash
# Pull by name
git-sub-pull mylib

# Auto-detect from current directory
cd vendor/mylib
git-sub-pull
```

**Note**: Squash mode is determined by the config (defaults to `true` if not specified). Config is never modified.

### git-sub-push

Push changes to the subtree repository.

**Syntax**:
```bash
git-sub-push [name]
```

**Examples**:
```bash
# Push by name
git-sub-push mylib

# Auto-detect from current directory
cd vendor/mylib
git-sub-push
```

**Note**: The `--squash` flag is never used for push (not supported by `git subtree push`).

## Auto-Detection

When you omit the `[name]` argument, the tool will attempt to auto-detect based on your current directory:

1. Determine your current directory relative to the repository root
2. Extract the first-level directory (e.g., `vendor` from `vendor/mylib/src`)
3. Find all configured subtrees whose `path` starts with that directory
4. If exactly one match is found, use it
5. If multiple matches exist, error and list candidates

**Example**:
```bash
cd vendor/mylib/src
git-sub-pull  # Auto-detects "mylib" if path=vendor/mylib
```

If multiple subtrees match:
```
ERROR: Multiple subtrees match directory 'vendor': mylib, otherlib. Please specify name explicitly.
```

## Installation Options

### Global Git Aliases

Run from any of the symlinks or the main helper:

```bash
./bin/git-subtree-helper --install
# or
git-sub-add --install
```

This configures global git aliases:
```bash
git config --global alias.sub-add '!/path/to/git-sub-add'
git config --global alias.sub-pull '!/path/to/git-sub-pull'
git config --global alias.sub-push '!/path/to/git-sub-push'
```

Now you can use:
```bash
git sub-add mylib --path vendor/mylib --url https://github.com/user/mylib.git
git sub-pull mylib
git sub-push mylib
```

### Add to PATH

Add the `bin` directory to your `PATH`:
```bash
export PATH="/path/to/git-subtree/bin:$PATH"
```

Then use commands directly:
```bash
git-sub-add mylib --path vendor/mylib --url https://github.com/user/mylib.git
git-sub-pull mylib
git-sub-push mylib
```

## Design Decisions

### Why INI format for config?

- **Simple and readable**: Easy to edit manually
- **Standard format**: Familiar to most developers
- **No external dependencies**: Parsed with awk/sed/grep
- **Git-like**: Similar to `.gitconfig` format

### Why default squash to true?

Most projects want clean history when incorporating external code. Squashing:
- Prevents cluttering your project history with upstream commits
- Makes it easier to see what changed in your project
- Simplifies reverting or removing a subtree

Non-squashed mode is available via `--no-squash` or `squash = false` in config for cases where you need full history (e.g., maintaining bidirectional sync).

### Why auto-detection?

When working within a subtree directory, it's inconvenient to remember or type the subtree name. Auto-detection based on the directory structure makes common workflows faster.

### Why separate commands instead of subcommands?

Using symlinks (`git-sub-add`, `git-sub-pull`, `git-sub-push`) instead of subcommands (`git-subtree-helper add`):
- Enables tab completion in most shells
- Follows git's convention for extension commands
- Makes git alias configuration simpler
- Allows running commands directly from PATH

## Limitations

### Git Subtree Limitations

This tool inherits limitations from `git subtree`:
- **Large histories**: Subtrees with extensive history can be slow
- **Merge conflicts**: Can occur when pulling updates
- **One-way push**: Pushing from a squashed subtree loses commit granularity

### Tool-Specific Limitations

- **First-level directory matching only**: Auto-detection only looks at the first path component
- **No nested subtree support**: Subtrees within subtrees may not work correctly
- **Config file location**: Must be at `.git/subtree.conf` (not configurable)
- **No validation**: Doesn't check if paths or URLs are valid before running git commands

## Benefits vs. Raw Git Subtree

| Feature | git subtree (raw) | git-subtree-helper |
|---------|-------------------|-------------------|
| Add subtree | Long command with all options | `git sub-add name --path P --url U` |
| Pull updates | Must remember URL, path, branch | `git sub-pull name` or auto-detect |
| Push changes | Must remember URL, path, branch | `git sub-push name` or auto-detect |
| Squash mode | Manual `--squash` flag each time | Configured once, applied automatically |
| Multiple subtrees | Track manually in documentation | Centralized in `.git/subtree.conf` |
| Onboarding | Team members must learn commands | Simple commands + readable config |

## Examples

### Example: Adding a Shared Library

```bash
# Add the library
git sub-add utils \
  --path lib/utils \
  --url https://github.com/company/utils.git \
  --branch v2

# Later, pull updates
git sub-pull utils

# Make changes and push back
cd lib/utils
# ... make changes ...
git commit -am "Fix bug in utils"
git sub-push  # Auto-detects "utils"
```

### Example: Managing Documentation

```bash
# Add external docs without squashing (preserve full history)
git sub-add apidocs \
  --path docs/api \
  --url https://github.com/company/api-docs.git \
  --no-squash

# Pull updates
cd docs/api
git sub-pull  # Auto-detects "apidocs"
```

### Example: Multiple Subtrees

```bash
# Add multiple libraries
git sub-add auth --path src/auth --url https://github.com/co/auth.git
git sub-add logging --path src/logging --url https://github.com/co/logging.git
git sub-add utils --path src/utils --url https://github.com/co/utils.git

# Update all (in a loop)
for name in auth logging utils; do
  git sub-pull $name
done
```

## Requirements

- Bash 4.0 or later
- Git 2.0 or later (with `git subtree` support)
- Standard POSIX tools: `awk`, `sed`, `grep`

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Built on top of [git-subtree](https://github.com/git/git/blob/master/contrib/subtree/git-subtree.sh)
- Inspired by the need for simpler subtree management in large projects

## Links

- [GitHub Repository](https://github.com/Gems/git-subtree)
- [Issue Tracker](https://github.com/Gems/git-subtree/issues)
- [Git Subtree Documentation](https://git-scm.com/docs/git-subtree)