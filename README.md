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

```
#!/bin/sh
powershell -ExecutionPolicy Bypass -File "$(dirname "$0")/commit-msg-validate-and-lint.ps1" "$1"
```

3. Create a commit-msg-validate-and-lint.ps1 file

```
param([string]$CommitMsgFile)

# Read the commit message and split their lines
$commitMsg = Get-Content $CommitMsgFile -Raw -Encoding UTF8
$commitMsg = $commitMsg -replace "`r", ""

$lines = $commitMsg -split "`n"
$header = $lines[0].Trim()

# Remove leading and trailing empty lines
$nonEmptyLines = $lines | Where-Object { $_.Trim() -ne "" }

if ($nonEmptyLines.Count -lt 2) {
    Write-Host "Commit message must contain at least header and refs line"
    exit 1
}

$refsLine = $nonEmptyLines[-1].Trim()
if ($nonEmptyLines.Count -gt 2) {
    $descriptionLines = $nonEmptyLines[1..($nonEmptyLines.Count - 2)]
    $description = ($descriptionLines -join "`n").Trim()
} else {
    $description = ""
}

# Regex patterns
$typePattern = 'feat|fix|chore|style|refactor|test'
$modulePattern = 'auth|lead|contract|multisite-deal|merchant|maintenance|git|cicd'
$ticketPattern = '(BOFFT|APPCM|UAT)-\d{1,6}'
$subjectPattern = "[A-Za-z0-9.,' -]+"

# Lint the header
if ($header -match "^\s*($typePattern)\s*\(?\s*($modulePattern)?\s*\)?\s*:\s*($ticketPattern)\s*-\s*($subjectPattern)$") {
    $type = $matches[1]
    $module = $matches[2] # undefined if not matched
    $ticket = $matches[3]
    $subject = $matches[5].Trim().ToLower()

    if ($module) {
        $header = "${type}(${module}): ${ticket} - ${subject}"
    } else {
        $header = "${type}: ${ticket} - ${subject}"
    }
} else {
    Write-Host "Invalid commit header format after normalization: ${header}"
    exit 1
}

# Truncate description if too long
if ($description.Length -gt 300) {
    $description = $description.Substring(0, 300)
}

# Validate mandatory refs line
if ($refsLine -match "^(?i)refs\s*#\s*($ticketPattern)$") {
    $refsLine = "refs #$($matches[1])"
} else {
    Write-Host "Commit message must contain refs#<ticket-number>. e.q. refs#BOFFT-19871"
    exit 1
}

# Repack the commit message
if ($description) {
    $finalMessage = "$header`n`n$description`n`n$refsLine"
} else {
    $finalMessage = "$header`n`n$refsLine"
}

[System.IO.File]::WriteAllText($CommitMsgFile, $finalMessage, [System.Text.UTF8Encoding]::new($false))
exit 0
```

### Pre-Commit Hook

1. Go to .git/hooks folder

2. Create a pre-commit file

3. Enter hook code content

```
#!/bin/sh

# Get current branch name
branch_name=$(git rev-parse --abbrev-ref HEAD)

# Define branch name pattern
pattern="^(feature|fix|chore|docs|refactor|test|style)/[a-z0-9\-]+$"

if echo "$branch_name" | grep -qE "$pattern"; then
    echo "✅ Branch name '$branch_name' follows the convention."
    exit 0
else
    echo "❌ Branch name '$branch_name' does not follow the convention."
    echo ""
    echo "Expected format:"
    echo "  <type>/<name>"
    echo "Examples:"
    echo "  feature/login-page"
    echo "  fix/api-crash"
    echo "  chore/cleanup"
    exit 1
fi
```
