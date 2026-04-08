---
description: Create or update the Product Requirements Document
argument-hint: "[topic]"
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Create PRD

Guide the creation of a Product Requirements Document.

## Arguments
`$ARGUMENTS` - Optional: "update" to modify existing PRD

## Process

### If Creating New
Guide user through:
1. Project name and summary
2. Core principles
3. Target users
4. MVP scope (in/out)
5. User stories
6. Tech stack decisions
7. Success criteria

### If Updating
- Read existing `.claude/PRD.md`
- Ask what sections to update
- Preserve existing content

## PRD Template

```markdown
# {Project Name} - Product Requirements Document

## Executive Summary
{2-3 sentences describing the project and its value}

## Mission
{Core mission statement}

## Core Principles
1. **{Principle 1}**: {explanation}
2. **{Principle 2}**: {explanation}
3. **{Principle 3}**: {explanation}

## Target Users

### Primary User
{Description of main user persona}

### User Characteristics
- {characteristic 1}
- {characteristic 2}

## MVP Scope

### In Scope
- {feature 1}
- {feature 2}
- {feature 3}

### Out of Scope (Future)
- {future feature 1}
- {future feature 2}

## User Stories

1. **As a {user}**, I want to {action}, so that {benefit}.
2. **As a {user}**, I want to {action}, so that {benefit}.

## Architecture

### Services
| Service | Responsibility |
|---------|---------------|
| {service} | {purpose} |

### Tech Stack
- **Frontend**: {technologies}
- **Backend**: {technologies}
- **Database**: {technology}
- **Infrastructure**: {technology}

## Success Criteria
- [ ] {measurable criterion 1}
- [ ] {measurable criterion 2}

## Implementation Phases

### Phase 1: {Name}
- {deliverable 1}
- {deliverable 2}

### Phase 2: {Name}
- {deliverable 1}

## Risks & Mitigations
| Risk | Impact | Mitigation |
|------|--------|------------|
| {risk} | {impact} | {mitigation} |
```
