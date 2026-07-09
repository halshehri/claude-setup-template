---
description: Create a spec-driven implementation plan for a feature
argument-hint: "<feature-description>"
allowed-tools: Read, Write, Glob, Grep, AskUserQuestion
---

# Plan Feature

Transform a feature request into a complete, executable implementation plan.

## Arguments

`$ARGUMENTS` - Feature description (required)

## Core Principle

**No code changes in this phase.** The plan must be complete enough that an implementer with zero context — who sees ONLY the plan — can execute it in a single pass. Every gap in the plan becomes a wrong guess during execution.

## Planning Process

### Phase 0: Assumptions & Ambiguities (spec gate)

Before any planning, make the spec explicit:

1. Restate the request in one sentence.
2. List every assumption you are making: scope, behavior, edge cases, non-goals.
3. Flag anything interpretable two different ways. Pick the most likely reading and write it down explicitly.
4. If a decision genuinely blocks planning, ask the user ONCE — all questions batched in a single message, never one at a time.
5. **Unattended/autonomous runs:** do not ask. Record the assumption under the plan's `Assumptions` section and proceed. The assumption record is the audit trail.

### Phase 1: Feature Understanding
- What core problem is being solved?
- Feature type: New | Enhancement | Refactor | Bug Fix
- Create user story: "As a {user}, I want {action}, so that {benefit}"

### Phase 2: Identify Affected Services
Determine which services this feature impacts and why.

### Phase 3: Codebase Analysis
For each affected service:
- Find similar implementations (patterns to follow)
- Identify files to modify and files to create
- Note integration points between services
- Note existing helpers/utils the implementation must reuse instead of re-writing

### Phase 3b: External Research (conditional)
Only if the plan touches an unfamiliar library, API, or pattern: check its
documentation before writing task code. Skip this phase entirely for
features built on familiar ground — it is not a standing ceremony.

### Phase 4: Create Implementation Plan
Save to: `.agents/plans/{kebab-case-feature-name}.md`

## No Placeholders (hard rule)

Every task must contain the actual content the implementer needs. These are **plan failures** — never write them:

- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" without the actual test code
- "Similar to Task N" — repeat the code; tasks may be read in isolation
- Steps that describe what to do without showing the code (code blocks required for code steps)
- References to types, functions, or methods not defined in any task

**Uncertainty is not a placeholder license:** where genuine unknowns remain
after Phase 3b, write a complete skeleton with real names and signatures and
record the unknown as an `Assumptions` entry — never confident fiction that
looks finished but was never verified.

## Plan Template

````markdown
# Feature: {Feature Name}

**Goal**: {One sentence describing what this builds}
**Type**: {New Feature | Enhancement | Bug Fix | Refactor}
**Created**: {date}

## User Story
As a {user type}, I want to {action}, so that {benefit}.

## Assumptions
- {Every assumption made in Phase 0, including ambiguities resolved and the reading chosen}

## Global Constraints
{Project-wide requirements every task implicitly includes — version floors,
naming conventions, platform requirements — one line each, exact values.}

## Affected Services
- [ ] `{service1}` - {why affected}

## Context References

### Files to Read Before Implementing
| File | Reason |
|------|--------|
| `{path}` | {Pattern to follow} |

---

## Implementation Tasks

### Task 1.1: {ACTION} `{target_file}`

**Files**:
- Create: `{exact/path/to/file}`
- Modify: `{exact/path/to/existing}:{line-range}`
- Test: `{exact/path/to/test}`

**Interfaces**:
- Consumes: {what this task uses from earlier tasks — exact signatures}
- Produces: {what later tasks rely on — exact function names, parameter and
  return types. This block is how a task's implementer learns the names and
  types neighboring tasks use.}

**Implementation**:
```{language}
{The actual code or a complete skeleton with real names, signatures, and
logic — not a description of code}
```

**Test**:
```{language}
{The actual test code}
```

**Validation**: `{exact command}` → expected: `{expected output}`

---

## Validation Commands

### Per-Service
```bash
# {service1}
cd {service1} && npm test && npm run lint
```

### Integration
```bash
{Commands that exercise the flow ACROSS services — the breakage per-service
suites can't see}
```

## Rollback
{How to undo if something goes wrong. Usually "revert commits for this
plan"; be explicit about anything a revert alone won't fix — migrations,
config changes, external state.}

## Acceptance Criteria
- [ ] {Criterion 1}
- [ ] All validation commands pass
````

## Self-Review (before handing off)

Look at the plan with fresh eyes and check:

1. **Spec coverage**: Can you point to a task for each requirement and each recorded assumption? Add tasks for any gaps.
2. **Placeholder scan**: Search the plan for the red flags in "No Placeholders" above. Fix them.
3. **Type consistency**: Do names and signatures used in later tasks match what earlier tasks define? `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

Fix issues inline, then move on. No re-review needed.

## After Creating Plan

Inform user: "Plan created at `.agents/plans/{name}.md`. Review it, then run `/core_piv_loop:execute .agents/plans/{name}.md`"
