---
description: Perform pre-commit code review on staged or modified changes
---

# Code Review: Pre-Commit Quality Check

## Arguments

`$ARGUMENTS` - Optional: specific service to focus on

## Process

### 1. Identify Changes
```bash
git diff --name-only HEAD
git diff --staged --name-only
```

### 2. Review Each Changed File

#### Priority 1: Critical Issues
- **Logic Errors**: Off-by-one, incorrect conditionals, null handling
- **Security**: SQL injection, XSS, exposed secrets
- **Data Loss**: Missing transactions, race conditions

#### Priority 2: Important Issues
- **Performance**: N+1 queries, missing indexes
- **Error Handling**: Unhandled exceptions
- **API Contract**: Breaking changes

#### Priority 3: Suggestions
- **Code Quality**: Readability, naming
- **Best Practices**: Patterns, conventions

## Output Report

```markdown
# Code Review: {brief description}

## Summary
- Files reviewed: {n}
- Critical issues: {n}
- Warnings: {n}

## Critical Issues (Must Fix)

### Issue 1: {title}
- **File**: `{path}`
- **Line**: {number}
- **Problem**: {description}
- **Fix**: {suggested solution}

## Warnings (Should Fix)

### Warning 1: {title}
- **File**: `{path}`
- **Fix**: {suggestion}

## Recommendation
{APPROVE | REQUEST CHANGES}
```
