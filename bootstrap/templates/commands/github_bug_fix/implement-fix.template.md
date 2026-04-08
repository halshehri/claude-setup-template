---
description: Implement a bug fix based on RCA findings
argument-hint: "<rca-file>"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Implement Fix

Implement a bug fix based on root cause analysis.

## Arguments

`$ARGUMENTS` - Optional: path to RCA output or brief description

## Process

### 1. Review RCA
- Confirm root cause understanding
- Review proposed fix approach

### 2. Implement Fix
- Make minimal necessary changes
- Follow existing code patterns
- Add/update tests for the bug

### 3. Validate
- Run relevant tests
- Verify fix logic

## Output

```markdown
## Bug Fix Implementation

### Issue
{Brief description}

### Fix Applied

#### Changes Made
| File | Change |
|------|--------|
| `{path}` | {description} |

### Tests
- [ ] Existing tests pass
- [ ] New test added for this bug

### Validation
```
{Test output}
```

### Ready for Review
Run `/validation:code-review` then `/commit`
```
