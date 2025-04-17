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
