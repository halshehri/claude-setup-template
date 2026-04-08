---
description: Generate a summary report of recent implementation work
allowed-tools: Read, Glob, Grep, Bash(git log:*), Bash(git diff:*), Bash(git status:*), Write
---

# Execution Report

Generate a comprehensive report of recent implementation work for documentation or handoff.

## Arguments
`$ARGUMENTS` - Optional: plan file path to report on

## Process

### 1. Gather Information
- Recent git commits
- Files changed
- Plan file (if specified)
- Test results

### 2. Generate Report

```markdown
## Implementation Report

### Feature/Task
{Description from plan or commit messages}

### Changes Summary
- Commits: {n}
- Files created: {n}
- Files modified: {n}
- Lines added: {n}
- Lines removed: {n}

### Commits
| Hash | Message | Files |
|------|---------|-------|
| {short-hash} | {message} | {count} |

### Files Changed
#### Created
- `{path}` - {purpose}

#### Modified
- `{path}` - {summary}

#### Deleted
- `{path}` - {reason}

### Test Coverage
{Test results summary}

### Documentation Updated
- [ ] CLAUDE.md files
- [ ] README
- [ ] API docs

### Known Issues / Tech Debt
{Any items deferred or known limitations}

### Deployment Notes
{Any special deployment considerations}
```
