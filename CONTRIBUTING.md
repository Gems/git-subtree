# Contributing to git-subtree

Thank you for your interest in contributing to git-subtree!

## Getting Started

1. Fork the repository
2. Clone your fork: `git clone https://github.com/YOUR-USERNAME/git-subtree.git`
3. Create a branch: `git checkout -b feature/your-feature-name`

## Development

The main script is `bin/git-subtree-helper`. It's a bash script that:
- Uses `set -euo pipefail` for strict error handling
- Dispatches based on symlink name (`git-sub-add`, `git-sub-pull`, `git-sub-push`)
- Parses `.git/subtree.conf` using awk/sed/grep (no external dependencies)

### Testing

Test your changes manually:

```bash
# Create a test repository
cd /tmp
git init test-repo
cd test-repo
git commit --allow-empty -m "Initial commit"

# Create a test config
cat > .git/subtree.conf <<EOF
[subtree "example"]
path = vendor/example
url = https://github.com/octocat/Hello-World.git
branch = master
squash = true
EOF

# Test the commands
/path/to/git-subtree/bin/git-sub-add example
/path/to/git-subtree/bin/git-sub-pull example
```

### Code Style

- Use 4-space indentation
- Follow existing function naming conventions
- Add comments for complex logic
- Keep lines under 120 characters when practical
- Use `err()` for error messages
- Use `print_cmd()` / `run_cmd()` for executing git commands

## Pull Request Process

1. Ensure your code works with bash 4.0+
2. Test all three commands (add, pull, push)
3. Update README.md if you add new features
4. Update the example config if needed
5. Write clear commit messages
6. Submit a pull request with a description of your changes

## Reporting Bugs

Please open an issue with:
- A clear description of the problem
- Steps to reproduce
- Expected behavior
- Actual behavior
- Your environment (OS, bash version, git version)

## Feature Requests

Open an issue describing:
- The problem you're trying to solve
- Your proposed solution
- Any alternatives you've considered
- How it aligns with the project goals

## Questions

If you have questions, feel free to:
- Open a discussion on GitHub
- Ask in an issue
- Contact the maintainers

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0.
