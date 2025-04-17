# Learn Git Hooks And Templates

## Git Templates

### Commit Template

1. Create a .git.message.txt file

2. Enter a template such as:

```

# <type>(<scope>): <subject>
#
# example: feat(auth): add login endpoint
#
# Body (optional, go into detail)
#
# Footer (optional, references, breaking changes)

```

3. Execute a command:

`git config commit.template .git.message.txt`

## Git Hooks

### Commit Message Hook

1. Go to .git/hooks folder

2. Create a commit-msg file

3. Enter hook code content (example below: conventional commits enforcement)

```
#!/bin/sh

# Grab the first line of the commit message
commit_msg=$(head -n1 "$1")

# Regex for Conventional Commits
conventional_regex="^(feat|fix|chore|docs|style|refactor|test|perf)(\([a-z0-9\-]+\))?!?: .{1,}"

if echo "$commit_msg" | grep -qE "$conventional_regex"; then
    echo "✅ Conventional commit message detected."
    exit 0
else
    echo "❌ Commit message does not follow Conventional Commits format."
    echo ""
    echo "Expected format:"
    echo "  <type>(<scope>): <subject>"
    echo "Example:"
    echo "  feat(auth): add login endpoint"
    exit 1
fi
```
