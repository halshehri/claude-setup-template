---
description: Perform root cause analysis on a GitHub issue or bug report
argument-hint: "<issue-number-or-url>"
allowed-tools: Read, Write, Glob, Grep, Bash(gh issue view:*), Bash(gh issue list:*), Bash(git log:*), Bash(git blame:*)
---

# RCA: Root Cause Analysis

Investigate a bug to identify root cause and plan a fix.

## Arguments

`$ARGUMENTS` - GitHub issue URL or issue description

## Process

### 1. Understand the Bug
- Fetch issue details (if URL provided)
- Note reproduction steps
- Identify affected functionality

### 2. Locate Relevant Code
- Search for related files
- Trace the code path
- Identify the component responsible

### 3. Root Cause Analysis
- Reproduce the issue logic mentally
- Identify the exact cause
- Check for related issues

### 4. Solution Design
- Propose fix approach
- Consider edge cases
- Plan validation

## Output

```markdown
## Root Cause Analysis: {issue title}

### Issue Summary
- **Source**: {GitHub issue URL or description}
- **Reported Behavior**: {what's broken}
- **Expected Behavior**: {what should happen}

### Investigation

#### Code Path Traced
1. {Entry point}
2. {Where bug occurs}

#### Root Cause
**Location**: `{file}:{line}`
**Problem**: {technical explanation}

### Proposed Fix

#### Approach
{Description of the fix}

#### Files to Modify
- `{file}` - {what to change}

### Validation Plan
```bash
{Commands to verify fix}
```

### Next Steps
Run `/github_bug_fix:implement-fix` to implement this fix.
```
