---
description: Track feature progress through requirement → design → implementation → testing → merged
---

# Feature Tracker

Manage feature lifecycle from requirement to production.

## Arguments
`$ARGUMENTS` - One of:
- `"{feature name}"` — Create new feature tracker
- *(empty)* — List all features and their stages
- `advance` — Advance current feature to next stage
- `advance {feature-slug}` — Advance a specific feature

## Stages

| # | Stage | What Happens |
|---|-------|-------------|
| 1 | **requirement** | Define scope, acceptance criteria, user story |
| 2 | **design** | Architecture decisions, affected services, approach |
| 3 | **implementation** | Task list, execute code changes |
| 4 | **testing** | Run validation, code review, fix issues |
| 5 | **merged** | Commit, push, merge to prod |

---

## Process by Action

### Action: Create New Feature (`/feature "Add user auth"`)

1. Convert name to slug: `add-user-auth`
2. Create file: `.agents/plans/add-user-auth.md`
3. Set stage to `requirement`
4. Ask the user to describe:
   - What problem this solves
   - Who it's for
   - Acceptance criteria (when is it done?)
5. Fill in the **Requirement** section from their answers
6. Report: feature created, next step is `/feature advance` when ready for design

**File template to create:**

```markdown
# Feature: {Feature Name}

## Status: requirement
- [x] requirement
- [ ] design
- [ ] implementation
- [ ] testing
- [ ] merged

**Slug**: {feature-slug}
**Branch**: feat/{feature-slug}
**Created**: {date}
**Updated**: {date}

---

## Requirement
**Problem**: {what problem this solves}
**User Story**: As a {user}, I want {action}, so that {benefit}.

### Acceptance Criteria
- [ ] {criterion 1}
- [ ] {criterion 2}
- [ ] {criterion 3}

---

## Design
*(filled during design stage)*

---

## Implementation Plan
*(filled during design stage, tasks checked off during implementation)*

---

## Test Results
*(filled during testing stage)*

---

## Merge Notes
*(filled during merge stage)*
```

### Action: List All Features (`/feature`)

1. Read all `.md` files in `.agents/plans/`
2. For each file, read the `## Status:` line
3. Display summary table:

```markdown
## Active Features

| Feature | Stage | Updated |
|---------|-------|---------|
| {name} | {stage} | {date} |
| {name} | {stage} | {date} |

Next: `/feature advance {slug}` to progress a feature.
```

Skip files that don't have a `## Status:` line (they're plain plans, not feature trackers).

### Action: Advance Feature (`/feature advance`)

Determine which feature to advance:
- If a slug is provided, use that
- If only one feature is in-progress (not `merged`), use that
- If multiple active features, ask which one

Then advance based on **current stage**:

#### requirement → design

1. Read the Requirement section for context
2. Analyze codebase: which services are affected, existing patterns
3. Ask the user about approach if multiple options exist
4. Fill in the **Design** section:
   - Architecture decisions
   - Affected services and files
   - Integration points
5. Fill in the **Implementation Plan** section:
   - Ordered task list with checkboxes
   - File-level detail (create/modify which files)
6. Update status to `design`, check off the checkbox
7. Report: design complete, review the plan, then `/feature advance` for implementation

#### design → implementation

1. Read the full feature file (requirement + design + plan)
2. Confirm user has reviewed and approved the design
3. Create feature branch: `git checkout -b feat/{slug}`
4. Execute implementation tasks one by one:
   - For each task: implement → validate → check off
5. Update status to `implementation`
6. Report: implementation complete, run `/feature advance` for testing

#### implementation → testing

1. Run validation: tests, lint, build (same as `/validation:validate all`)
2. Run code review (same as `/validation:code-review`)
3. Fill in **Test Results** section with:
   - Validation output
   - Code review summary
   - Any issues found and whether they were fixed
4. If issues remain: report them, do NOT advance yet
5. If all clear: update status to `testing`
6. Report: testing passed, run `/feature advance` to merge

#### testing → merged

1. Verify all tests still pass
2. Stage and commit with conventional commit message (same as `/commit`)
3. Fill in **Merge Notes** section:
   - Commit hash
   - Files changed summary
   - Any deployment notes
4. Update status to `merged`
5. Report: feature complete, merged on branch `feat/{slug}`

---

## Status Update Format

When updating a feature file, change these:
- The `## Status: {stage}` line
- The checkbox list (check completed stages)
- The `**Updated**:` date

Example after advancing to implementation:
```markdown
## Status: implementation
- [x] requirement
- [x] design
- [x] implementation
- [ ] testing
- [ ] merged
```

## Error Handling

- If `.agents/plans/` doesn't exist, create it
- If feature file already exists on create, warn and ask to resume or rename
- If advancing fails (tests fail, review issues), stay at current stage and report what needs fixing
- Never skip stages — each must complete before the next
