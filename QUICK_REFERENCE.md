# Quick Reference

## Installation

```bash
git clone https://github.com/Gems/git-subtree.git
cd git-subtree
./bin/git-subtree-helper --install
```

## Basic Usage

### 1. Add a subtree
```bash
git sub-add <name> --path <path> --url <url> [--branch <branch>] [--no-squash]
```

### 2. Pull updates
```bash
git sub-pull <name>
```

### 3. Push changes
```bash
git sub-push <name>
```

## Auto-detection

From within a subtree directory, omit the name:
```bash
cd vendor/mylib
git sub-pull   # Auto-detects "mylib"
git sub-push   # Auto-detects "mylib"
```

## Configuration Format

Location: `<repo-root>/.git/subtree.conf`

```ini
[subtree "name"]
path = local/path
url = https://github.com/user/repo.git
branch = main
squash = true  # Default if omitted
```

## Environment Variables

- `QUIET=1` - Suppress command printing

## Common Workflows

### Add multiple libraries
```bash
git sub-add lib1 --path vendor/lib1 --url https://github.com/x/lib1.git
git sub-add lib2 --path vendor/lib2 --url https://github.com/x/lib2.git
git sub-add lib3 --path vendor/lib3 --url https://github.com/x/lib3.git
```

### Update all subtrees
```bash
for name in lib1 lib2 lib3; do
  git sub-pull $name
done
```

### Work on a subtree and push back
```bash
cd vendor/lib1
# Make changes...
git commit -am "Fix bug"
cd ../..
git sub-push lib1
```

## Help

```bash
git-sub-add --help
git-sub-pull --help
git-sub-push --help
```

## Troubleshooting

### "Not in a git repository"
Make sure you're inside a git repository when running commands (except `--install`).

### "Could not auto-detect subtree name"
Either:
- Specify the name explicitly: `git sub-pull <name>`
- Or create/update `.git/subtree.conf` with the subtree configuration

### "Multiple subtrees match directory"
Multiple subtrees have paths starting with the same directory. Specify the name explicitly.

### "Missing 'path' in config"
The subtree is not configured. Use `git sub-add` with `--path` and `--url` options first.
