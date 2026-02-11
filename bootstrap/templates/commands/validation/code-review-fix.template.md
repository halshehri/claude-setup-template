---
description: Automatically fix issues identified in code review
---

# Code Review Fix

Automatically apply fixes for issues identified by `/validation:code-review`.

## Arguments
`$ARGUMENTS` - Optional: "critical-only" to fix only critical issues

## Process

### 1. Get Review Results
Read the most recent code review output or re-run `/validation:code-review`.

### 2. Categorize Fixes
- **Auto-fixable**: Linting, formatting, simple patterns
- **Manual required**: Logic changes, architectural issues

### 3. Apply Fixes
For each auto-fixable issue:
1. Read the file
2. Apply the fix
3. Verify the fix doesn't break anything

### 4. Report Manual Items
List issues that require human decision.

## Output

```markdown
## Code Review Fix Report

### Auto-Fixed
- [x] `{file}:{line}` - {issue} - Fixed
- [x] `{file}:{line}` - {issue} - Fixed

### Manual Attention Required
- [ ] `{file}:{line}` - {issue} - {why manual}

### Verification
```bash
{validation command output}
```

### Next Steps
{If manual items}: Address manual items, then `/validation:code-review`
{If all fixed}: Run `/validation:validate` then `/commit`
```
