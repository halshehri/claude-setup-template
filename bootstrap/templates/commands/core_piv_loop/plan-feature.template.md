---
description: Create a comprehensive implementation plan for a feature
argument-hint: "<feature-description>"
allowed-tools: Read, Write, Glob, Grep
---

# Plan Feature

Transform a feature request into a comprehensive implementation plan.

## Arguments

`$ARGUMENTS` - Feature description (required)

## Core Principle

**No code in this phase.** Create a context-rich plan that enables single-pass implementation.

## Planning Process

### Phase 1: Feature Understanding
- What core problem is being solved?
- Feature type: New | Enhancement | Refactor | Bug Fix
- Create user story: "As a {user}, I want {action}, so that {benefit}"

### Phase 2: Identify Affected Services
Determine which services this feature impacts and why.

### Phase 3: Codebase Analysis
For each affected service:
- Find similar implementations (patterns to follow)
- Identify files to modify
- Identify files to create
- Note integration points between services

### Phase 4: Create Implementation Plan
Save to: `.agents/plans/{kebab-case-feature-name}.md`

## Plan Template

```markdown
# Feature: {Feature Name}

## Overview
**Description**: {Detailed description}
**Type**: {New Feature | Enhancement | Bug Fix | Refactor}
**Created**: {date}

## User Story
As a {user type},
I want to {action},
So that {benefit}.

## Affected Services
- [ ] `{service1}` - {why affected}
- [ ] `{service2}` - {why affected}

---

## Context References

### Files to Read Before Implementing
| File | Reason |
|------|--------|
| `{path}` | {Pattern to follow} |

### Patterns to Follow
{Specific code patterns observed in the codebase}

---

## Implementation Tasks

### Phase 1: {Service/Component Name}

#### Task 1.1: {ACTION} `{target_file}`
**Type**: {Create | Modify}
**Description**: {What to do}
**Pattern Reference**: `{file}:{line-range}`
**Validation**: `{command}`

---

## Validation Commands
```bash
cd {service1} && npm test && npm run lint
```

## Acceptance Criteria
- [ ] {Criterion 1}
- [ ] All validation commands pass
```

## After Creating Plan
Inform user: "Plan created at `.agents/plans/{name}.md`. Review it, then run `/core_piv_loop:execute .agents/plans/{name}.md`"
