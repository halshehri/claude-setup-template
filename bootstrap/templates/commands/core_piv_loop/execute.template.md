---
description: Execute an implementation plan task by task with validation
---

# Execute: Implement from Plan

Execute an implementation plan created by `/core_piv_loop:plan-feature`.

## Arguments

`$ARGUMENTS` - Path to plan file (e.g., `.agents/plans/add-user-auth.md`)

## Execution Process

### 1. Load Plan
- Read the ENTIRE plan file
- Parse affected services
- Note validation commands

### 2. Load Service Context
For each affected service, read:
- `{service}/CLAUDE.md`
- Key files referenced in plan

### 3. Execute Tasks Sequentially
For each task in the plan:
1. Read the task specification
2. Read pattern reference files
3. Implement the change
4. Run task's validation command
5. Fix any issues before proceeding

### 4. Per-Phase Validation
After completing each phase, run service tests.

### 5. Final Validation
Run all validation commands from the plan.

## Error Handling
- If a task fails validation, fix before proceeding
- If blocked, document the blocker and ask user for guidance
- Do not skip tasks without user approval

## Output Report

```markdown
## Execution Report: {feature-name}

### Summary
- Plan: `{plan-path}`
- Status: {Complete | Partial | Blocked}

### Completed Tasks
- [x] Task 1.1: {description}
- [ ] Task 2.1: {description} - {reason if incomplete}

### Files Created
| File | Purpose |
|------|---------|
| `{path}` | {description} |

### Files Modified
| File | Changes |
|------|---------|
| `{path}` | {summary} |

### Validation Results
{output of validation commands}

### Next Steps
- [ ] Run `/validation:code-review`
- [ ] Run `/commit`
```
