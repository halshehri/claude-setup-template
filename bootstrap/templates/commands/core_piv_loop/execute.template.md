---
description: Execute an implementation plan task by task with validation
argument-hint: "[plan-file]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task
---

# Execute: Implement from Plan

Execute an implementation plan created by `/core_piv_loop:plan-feature`.

## Arguments

`$ARGUMENTS` - Path to plan file (e.g., `.agents/plans/add-user-auth.md`)

## Coding Rules: Minimal Code That Works

Before writing any code, climb this ladder and stop at the first rung that holds:

1. **Does this need to exist at all?** Speculative need = skip it, note it in the report. (YAGNI)
2. **Already in this codebase?** A helper, util, type, or pattern that already lives here → reuse it. Look before you write.
3. **Standard library covers it?** Use it.
4. **Native platform feature covers it?** DB constraint over app code, CSS over JS.
5. **Already-installed dependency solves it?** Use it. Never add a new dependency for what a few lines can do.
6. **Can it be one line?** One line.
7. **Only then:** the minimum code that works.

Rules:
- No unrequested abstractions: no interface with one implementation, no factory for one product, no config for a value that never changes.
- Bug fix = root cause, not symptom: one guard in the shared function beats a guard in every caller.
- Deletion over addition. Boring over clever.
- Never simplify away: input validation at trust boundaries, error handling that prevents data loss, security measures, or anything the plan explicitly requires.
- **Test policy: the plan's validation commands govern.** The ladder shrinks production code, never the validation loop.

## Execution Process

### 1. Load Plan
- Read the ENTIRE plan file
- Read the `Assumptions` and `Global Constraints` sections first — every task implicitly includes them
- Parse affected services
- Note validation commands

### 2. Load Service Context
For each affected service, read:
- `{service}/CLAUDE.md`
- Key files referenced in plan

### 3. Execute Tasks Sequentially
For each task in the plan:
1. Read the task specification, including its `Interfaces` block
2. Read pattern reference files
3. Implement the change (apply the ladder above)
4. Run task's validation command and confirm the expected output
5. Fix any issues before proceeding

### 4. Per-Phase Validation
After completing each phase, run service tests.

### 5. Final Validation
Run all validation commands from the plan — per-service first, then
integration.

### 6. Fresh-Eyes Review
Dispatch ONE review subagent with fresh context (no session history) to run
`/validation:code-review` against the full diff, with the plan file as its
spec. One fresh-context pass at the end catches the drift that accumulates
during long implementation sessions.

- Fix all Critical issues it reports, then re-run final validation
- Log Warnings in the execution report; fix them only if cheap

### 7. Verify Before Claiming Done
Apply the `verification-before-completion` skill: no completion claims
without fresh verification evidence. Run each validation command, read the
full output, and report the actual state — especially in unattended runs,
where the report is the only witness.

## Error Handling
- If a task fails validation, fix before proceeding
- If blocked, document the blocker and ask user for guidance
- In unattended runs: mark the task BLOCKED in the report, skip dependents, continue with independent tasks
- If final validation fails in an unattended run and cannot be fixed: execute the plan's `Rollback` section rather than leaving the tree half-changed, and record the rollback in the report
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

### Skipped as Unnecessary (ladder rung 1)
- {anything the plan called for that turned out not to need to exist, and why}

### Files Created
| File | Purpose |
|------|---------|
| `{path}` | {description} |

### Files Modified
| File | Changes |
|------|---------|
| `{path}` | {summary} |

### Validation Results
{actual output of validation commands — fresh runs, not recalled}

### Review Findings
- Critical: {n} found, {n} fixed
- Warnings: {list, with fixed/deferred}

### Next Steps
- [ ] Run `/commit`
```
