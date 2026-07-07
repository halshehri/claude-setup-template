# Claude Code Project Setup Template

This document is a comprehensive guide to set up Claude Code for efficient AI-assisted development. It supports two project types and can be used for greenfield projects or injected into existing codebases.

---

## Table of Contents

1. [Project Types](#project-types)
2. [Quick Setup Checklist](#quick-setup-checklist)
3. [Folder Structure](#folder-structure)
4. [Step-by-Step Setup](#step-by-step-setup)
5. [Injecting into Existing Projects](#injecting-into-existing-projects)
6. [Skills & Commands Reference](#skills--commands-reference)
7. [File Templates](#file-templates)

---

## Project Types

### Type 1: Monorepo
Single repository containing all services.

```
project-root/              <- has .git
├── .claude/
├── .agents/plans/
├── CLAUDE.md
├── backend/
│   └── CLAUDE.md
└── frontend/
    └── CLAUDE.md
```

### Type 2: Multi-repo Microservices
Parent folder (no repo) with child folders, each being a separate repo under one GitHub organization.

```
org-parent/                <- NO .git (local folder only)
├── .claude/               <- shared commands/agents
├── .agents/plans/
├── CLAUDE.md              <- org-level overview
├── accounts-service/      <- has .git (own repo)
│   └── CLAUDE.md
├── gateway-service/       <- has .git (own repo)
│   └── CLAUDE.md
└── frontend/              <- has .git (own repo)
    └── CLAUDE.md
```

**Important**: Always run Claude Code from the root folder (project-root or org-parent). This ensures access to shared commands and enables cross-service work.

---

## Quick Setup Checklist

### For New Projects

- [ ] Create folder structure (`.claude/`, `.agents/plans/`)
- [ ] Create root `CLAUDE.md`
- [ ] Create `.claude/PRD.md`
- [ ] Create agent definitions in `.claude/agents/`
- [ ] Create commands in `.claude/commands/`
- [ ] Create reference docs in `.claude/reference/`
- [ ] Create service-specific `CLAUDE.md` files
- [ ] (Optional) Create `.claude/settings.local.json`
- [ ] Test commands by typing `/` in Claude Code

### For Existing Projects

- [ ] Create `.claude/` folder structure
- [ ] Create root `CLAUDE.md` documenting existing architecture
- [ ] Create `.claude/PRD.md` (or link to existing docs)
- [ ] Add service-specific `CLAUDE.md` files
- [ ] Copy commands and agents from this template
- [ ] Create reference docs for your tech stack
- [ ] Add `.claude/` and `.agents/` to `.gitignore` if needed

---

## Folder Structure

```
project-root/
├── .agents/
│   └── plans/                     # Implementation plans (created by plan-feature)
├── .claude/
│   ├── agents/                    # AI agent definitions
│   │   ├── senior-architect.md
│   │   └── fullstack-engineer.md
│   ├── commands/                  # Custom slash commands (skills)
│   │   ├── core_piv_loop/
│   │   │   ├── prime.md           # /core_piv_loop:prime
│   │   │   ├── plan-feature.md    # /core_piv_loop:plan-feature
│   │   │   └── execute.md         # /core_piv_loop:execute
│   │   ├── validation/
│   │   │   ├── validate.md        # /validation:validate
│   │   │   ├── code-review.md     # /validation:code-review
│   │   │   ├── code-review-fix.md # /validation:code-review-fix
│   │   │   └── execution-report.md# /validation:execution-report
│   │   ├── github_bug_fix/
│   │   │   ├── rca.md             # /github_bug_fix:rca
│   │   │   └── implement-fix.md   # /github_bug_fix:implement-fix
│   │   ├── commit.md              # /commit
│   │   ├── feature.md             # /feature
│   │   ├── init-project.md        # /init-project
│   │   └── create-prd.md          # /create-prd
│   ├── reference/                 # Best practices documentation
│   │   └── {tech}-best-practices.md
│   ├── settings.local.json        # Allowed bash commands (optional)
│   └── PRD.md                     # Product requirements document
├── CLAUDE.md                      # Root-level project context
└── {service}/
    └── CLAUDE.md                  # Service-specific context
```

---

## Step-by-Step Setup

### Step 1: Create Folder Structure

```bash
# Create directories
mkdir -p .claude/agents
mkdir -p .claude/commands/core_piv_loop
mkdir -p .claude/commands/validation
mkdir -p .claude/commands/github_bug_fix
mkdir -p .claude/reference
mkdir -p .agents/plans
```

---

### Step 2: Create Root CLAUDE.md

Create `CLAUDE.md` at the project root:

```markdown
# {Project Name}

This file provides guidance for Claude Code. For service-specific details, see the CLAUDE.md in each service directory.

## Project Type
{Monorepo | Multi-repo Microservices}

## Quick Reference

### Services
| Service | Directory | Purpose | Repo (if multi-repo) |
|---------|-----------|---------|----------------------|
| {Service1} | `{dir}/` | {description} | {org/repo-name} |
| {Service2} | `{dir}/` | {description} | {org/repo-name} |

### Common Commands
```bash
# Development
cd {service1} && npm run dev
cd {service2} && python -m uvicorn app.main:app --reload

# Testing
cd {service1} && npm test
cd {service2} && pytest

# Build
cd {service1} && npm run build
```

## Architecture Overview

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Frontend   │────▶│  API Gateway │────▶│  Services   │
└─────────────┘     └─────────────┘     └─────────────┘
                                              │
                                              ▼
                                        ┌──────────┐
                                        │ Database │
                                        └──────────┘
```

### Data Flow
1. {Step 1}
2. {Step 2}
3. {Step 3}

## Tech Stack

- **Frontend**: {technologies}
- **Backend**: {technologies}
- **Database**: {technology}
- **Infrastructure**: {technology}

## Claude Code Workflow

### PIV Loop (Prime → Implement → Validate)
```bash
/core_piv_loop:prime {services}      # Load context
/core_piv_loop:plan-feature "{desc}" # Create implementation plan
/core_piv_loop:execute {plan-path}   # Execute the plan
```

### Validation
```bash
/validation:validate {service}       # Run tests, lint, build
/validation:validate all             # Validate all services
/validation:code-review              # Pre-commit code review
```

### Bug Fixes (from GitHub issues)
```bash
/github_bug_fix:rca {issue-url}      # Root cause analysis
/github_bug_fix:implement-fix        # Implement the fix
```

### Git
```bash
/commit                              # Create conventional commit
```

## Working with Multiple Services

1. **Prime relevant services** with `/core_piv_loop:prime service1,service2`
2. **Plan the feature** with `/core_piv_loop:plan-feature`
3. **Review the plan** in `.agents/plans/`
4. **Execute** with `/core_piv_loop:execute`
5. **Validate each service** after implementation
6. **Commit** with `/commit`

## Key Database Tables

| Table | Purpose |
|-------|---------|
| {table1} | {purpose} |
| {table2} | {purpose} |

## Environment Variables

See each service's CLAUDE.md for service-specific variables.

## Reference Documentation

See `.claude/reference/` for best practices:
- `{tech1}-best-practices.md`
- `{tech2}-best-practices.md`
```

---

### Step 3: Create Service CLAUDE.md Files

Create `{service}/CLAUDE.md` for each service:

```markdown
# {Service Name}

{Brief description of the service purpose}

## Tech Stack
- **Runtime**: {Node.js 20 / Python 3.11 / etc.}
- **Framework**: {Express / FastAPI / React / etc.}
- **Database**: {PostgreSQL / MongoDB / etc.}

## Directory Structure
```
{service}/
├── src/               # Source code
│   ├── index.ts       # Entry point
│   ├── routes/        # API routes
│   ├── services/      # Business logic
│   ├── models/        # Data models
│   └── utils/         # Utilities
├── tests/             # Test files
├── package.json       # Dependencies
└── tsconfig.json      # TypeScript config
```

## Development Commands
```bash
npm install          # Install dependencies
npm run dev          # Start with hot-reload
npm run build        # Production build
npm test             # Run tests
npm run lint         # Run linter
```

## Key Responsibilities
- {responsibility 1}
- {responsibility 2}
- {responsibility 3}

## API Endpoints (if applicable)
| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | /api/{resource} | {description} |
| POST | /api/{resource} | {description} |

## Environment Variables
| Variable | Description | Example |
|----------|-------------|---------|
| `PORT` | Server port | `3000` |
| `DATABASE_URL` | DB connection string | `postgresql://...` |

## Code Conventions
- {Use async/await for all async operations}
- {Error handling pattern}
- {Naming conventions}

## Integration Points
- Communicates with: {other services}
- Depends on: {external services}
```

---

### Step 4: Create Agent Definitions

#### `.claude/agents/senior-architect.md`

```markdown
---
name: senior-architect
description: Architectural guidance, design decisions, and solution proposals. Clarifies requirements through questions before proposing solutions.
tools: Glob, Grep, LS, Read, NotebookRead, WebFetch, TodoWrite, WebSearch
model: opus
color: yellow
---

You are a Senior Software Architect with 15+ years of experience designing scalable, maintainable systems.

## Your Responsibilities

### 1. Requirements Analysis
When presented with a feature request:
- Ask 2-4 clarifying questions to understand the business need
- Identify constraints, edge cases, and non-functional requirements
- Consider impact on existing system components

### 2. Solution Design Principles
- Favor the simplest solution that meets requirements
- Prioritize maintainability and developer experience
- Minimize dependencies and coupling between services
- Leverage existing patterns in the codebase

### 3. Design Process
1. Ask targeted questions to clarify requirements
2. Analyze how the feature fits within existing architecture
3. Propose 1-2 solution approaches with trade-offs
4. Recommend optimal approach with clear reasoning

### 4. Communication Style
- Be concise but thorough
- Use bullet points for clarity
- Provide concrete examples
- Always justify added complexity

## Response Format

```
## Clarifying Questions
- [Question about requirement]

## Analysis
[How this fits with existing architecture]

## Proposed Solution
[Brief overview]

### Option A: {name}
- Approach: {description}
- Pros: {benefits}
- Cons: {limitations}

### Option B: {name}
- Approach: {description}
- Pros: {benefits}
- Cons: {limitations}

## Recommendation
[Recommended option with reasoning]

## Implementation Considerations
- {consideration 1}
- {consideration 2}
```
```

#### `.claude/agents/fullstack-engineer.md`

```markdown
---
name: fullstack-engineer
description: Implements features based on architectural specs. Handles backend APIs, frontend components, database changes, and testing.
model: sonnet
color: blue
---

You are an expert fullstack engineer implementing production-quality code.

## Your Responsibilities

### 1. Implementation Excellence
- Read existing code thoroughly before making changes
- Follow established patterns in the codebase
- Adhere to conventions documented in CLAUDE.md files
- Write clean, readable, well-tested code

### 2. Quality Standards
- **Correctness**: Code works as specified
- **Maintainability**: Easy to understand and modify
- **Performance**: Efficient and scalable
- **Security**: No vulnerabilities introduced

### 3. Before Writing Code
1. Read relevant CLAUDE.md files
2. Find similar implementations in the codebase
3. Understand the patterns used
4. Plan your changes

### 4. Code Review Checklist
Before considering work complete:
- [ ] Code follows project style guidelines
- [ ] No hardcoded values or secrets
- [ ] Proper error handling implemented
- [ ] Tests written/updated
- [ ] No console.log/print statements left
- [ ] Changes committed to correct branch

### 5. Git Workflow
- Work on feature branches, not main
- Create atomic commits with clear messages
- Only merge to main after validation passes
```

---

### Step 5: Create Commands

All commands require YAML frontmatter with a `description` field.

#### `.claude/commands/core_piv_loop/prime.md`

```markdown
---
description: Load project context for specified services before planning or implementing
---

# Prime: Load Project Context

Build comprehensive understanding of the codebase before planning or implementing.

## Arguments
`$ARGUMENTS` - Comma-separated service names (e.g., "api-gateway,frontend") or empty for overview

## Process

### 1. Analyze Project Structure
```bash
git ls-files | head -100
```
For directory overview (if tree available):
```bash
tree -L 3 -I 'node_modules|__pycache__|.git|dist|build|.venv'
```

### 2. Read Core Documentation
- Read root `CLAUDE.md`
- Read `.claude/PRD.md` if exists
- Read `README.md` files

### 3. If Services Specified
For each service in `$ARGUMENTS`:
- Read `{service}/CLAUDE.md`
- List directory structure
- Identify entry points and key files
- Check for uncommitted changes

### 4. Check Current State
```bash
git status
git log -5 --oneline
git branch --show-current
```

## Output Report

```markdown
## Prime Report: {service names or "Project Overview"}

### Project Summary
- Type: {Monorepo | Multi-repo}
- Purpose: {brief description}

### Services Analyzed
| Service | Tech Stack | Status |
|---------|------------|--------|
| {name} | {stack} | {clean/modified} |

### Architecture Overview
{Key architectural patterns observed}

### Current State
- Branch: {current branch}
- Recent commits: {summary}
- Uncommitted changes: {yes/no, which services}

### Key Files Identified
- {service}/src/index.ts - Entry point
- {service}/src/routes/ - API routes

### Ready for Next Step
Use `/core_piv_loop:plan-feature "{description}"` to create an implementation plan.
```
```

#### `.claude/commands/core_piv_loop/plan-feature.md`

`````markdown
---
description: Create a spec-driven implementation plan for a feature
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
```bash
cd {service1} && npm test && npm run lint
```

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
`````

#### `.claude/commands/core_piv_loop/execute.md`

`````markdown
---
description: Execute an implementation plan task by task with validation
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
Run all validation commands from the plan.

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
`````

#### `.claude/commands/validation/validate.md`

```markdown
---
description: Run validation checks (tests, lint, build) for specified services
---

# Validate: Run Comprehensive Checks

## Arguments
`$ARGUMENTS` - Comma-separated service names or "all"

## Process

### 1. Determine Services
If "all": Get list from root CLAUDE.md
If specific: Use provided list

### 2. Run Checks Per Service

#### For Node.js/TypeScript Services
```bash
cd {service}
npm run lint          # Linting
npm run build         # Type checking / build
npm test              # Unit tests
```

#### For Python Services
```bash
cd {service}
python -m py_compile {main_file}  # Syntax check
pytest                             # Tests
ruff check .                       # Linting (or flake8/pylint)
mypy .                             # Type checking (if configured)
```

### 3. Integration Tests (if applicable)
```bash
npm run test:integration
# or
pytest tests/integration/
```

## Output Report

```markdown
## Validation Report

### Services Validated
| Service | Lint | Build | Tests | Status |
|---------|------|-------|-------|--------|
| {name} | PASS | PASS | PASS | PASS |
| {name} | PASS | FAIL | - | FAIL |

### Failures
#### {service} - Build
```
{error output}
```

**Suggested Fix**: {suggestion}

### Summary
- Total: {n} services
- Passed: {n}
- Failed: {n}

### Next Steps
{If all pass}: Ready for `/validation:code-review` then `/commit`
{If failures}: Fix issues and re-run `/validation:validate {failed-services}`
```
```

#### `.claude/commands/validation/code-review.md`

```markdown
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
- **Security**: SQL injection, XSS, exposed secrets, path traversal
- **Data Loss**: Missing transactions, race conditions

#### Priority 2: Important Issues
- **Performance**: N+1 queries, missing indexes, memory leaks
- **Error Handling**: Unhandled exceptions, poor error messages
- **API Contract**: Breaking changes, missing validation

#### Priority 3: Suggestions
- **Code Quality**: Readability, naming, duplication
- **Best Practices**: Patterns, conventions, documentation

### 3. Cross-Service Review
- Check integration points between services
- Verify API contracts match
- Check for consistent error handling

## Output Report

```markdown
# Code Review: {brief description of changes}

## Summary
- Files reviewed: {n}
- Critical issues: {n}
- Warnings: {n}
- Suggestions: {n}

## Critical Issues (Must Fix)

### Issue 1: {title}
- **File**: `{path}`
- **Line**: {number}
- **Problem**: {description}
- **Risk**: {what could go wrong}
- **Fix**: {suggested solution}

## Warnings (Should Fix)

### Warning 1: {title}
- **File**: `{path}`
- **Line**: {number}
- **Problem**: {description}
- **Fix**: {suggested solution}

## Suggestions (Consider)

### Suggestion 1: {title}
- **File**: `{path}`
- **Suggestion**: {improvement}

## Recommendation
{APPROVE | REQUEST CHANGES}

{If approve}: Ready for `/commit`
{If changes requested}: Address critical issues, then re-run `/validation:code-review`
```
```

#### `.claude/commands/validation/code-review-fix.md`

```markdown
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
```

#### `.claude/commands/validation/execution-report.md`

```markdown
---
description: Generate a summary report of recent implementation work
---

# Execution Report

Generate a comprehensive report of recent implementation work for documentation or handoff.

## Arguments
`$ARGUMENTS` - Optional: plan file path to report on

## Process

### 1. Gather Information
- Recent git commits
- Files changed
- Plan file (if specified)
- Test results

### 2. Generate Report

```markdown
## Implementation Report

### Feature/Task
{Description from plan or commit messages}

### Changes Summary
- Commits: {n}
- Files created: {n}
- Files modified: {n}
- Lines added: {n}
- Lines removed: {n}

### Commits
| Hash | Message | Files |
|------|---------|-------|
| {short-hash} | {message} | {count} |

### Files Changed
#### Created
- `{path}` - {purpose}

#### Modified
- `{path}` - {summary}

#### Deleted
- `{path}` - {reason}

### Test Coverage
{Test results summary}

### Documentation Updated
- [ ] CLAUDE.md files
- [ ] README
- [ ] API docs

### Known Issues / Tech Debt
{Any items deferred or known limitations}

### Deployment Notes
{Any special deployment considerations}
```
```

#### `.claude/commands/github_bug_fix/rca.md`

```markdown
---
description: Perform root cause analysis on a GitHub issue or bug report
---

# RCA: Root Cause Analysis

Investigate a bug from a GitHub issue to identify root cause and plan a fix.

## Arguments
`$ARGUMENTS` - GitHub issue URL or issue description

## Process

### 1. Understand the Bug
If GitHub URL provided:
- Fetch issue details
- Read comments and any linked PRs
- Note reproduction steps

If description provided:
- Parse the symptoms
- Identify affected functionality

### 2. Locate Relevant Code
- Search for related files
- Trace the code path
- Identify the component responsible

### 3. Root Cause Analysis
- Reproduce the issue logic mentally
- Identify the exact cause
- Check for related issues elsewhere

### 4. Impact Assessment
- What else might be affected?
- Are there similar patterns in the codebase?

### 5. Solution Design
- Propose fix approach
- Consider edge cases
- Plan validation

## Output

```markdown
## Root Cause Analysis: {issue title or brief}

### Issue Summary
- **Source**: {GitHub issue URL or description}
- **Reported Behavior**: {what's broken}
- **Expected Behavior**: {what should happen}

### Investigation

#### Code Path Traced
1. {Entry point}
2. {Function/file}
3. {Where bug occurs}

#### Root Cause
**Location**: `{file}:{line}`
**Problem**: {technical explanation}
**Why it happens**: {detailed cause}

### Impact Assessment
- **Severity**: {Critical | High | Medium | Low}
- **Scope**: {What features/users affected}
- **Related Code**: {Other places with similar patterns}

### Proposed Fix

#### Approach
{Description of the fix}

#### Files to Modify
- `{file}` - {what to change}

#### Edge Cases to Handle
- {edge case 1}
- {edge case 2}

### Validation Plan
```bash
{Commands to verify fix}
```

### Next Steps
Run `/github_bug_fix:implement-fix` to implement this fix.
```
```

#### `.claude/commands/github_bug_fix/implement-fix.md`

```markdown
---
description: Implement a bug fix based on RCA findings
---

# Implement Fix

Implement a bug fix based on root cause analysis.

## Arguments
`$ARGUMENTS` - Optional: path to RCA output or brief description

## Prerequisites
- RCA completed (via `/github_bug_fix:rca` or manual analysis)
- Clear understanding of root cause
- Fix approach defined

## Process

### 1. Review RCA
- Confirm root cause understanding
- Review proposed fix approach
- Check files to modify

### 2. Implement Fix
- Make minimal necessary changes
- Follow existing code patterns
- Add/update tests for the bug

### 3. Validate
- Run relevant tests
- Manually verify fix logic
- Check for regressions

### 4. Document
- Add comments if fix is non-obvious
- Update any relevant documentation

## Output

```markdown
## Bug Fix Implementation

### Issue
{Brief description}

### Root Cause
{Summary from RCA}

### Fix Applied

#### Changes Made
| File | Change |
|------|--------|
| `{path}` | {description} |

#### Code Diff Summary
{Key changes explained}

### Tests
- [ ] Existing tests pass
- [ ] New test added for this bug
- [ ] Edge cases covered

### Validation
```
{Test output}
```

### Ready for Review
Run `/validation:code-review` then `/commit`
```
```

#### `.claude/commands/commit.md`

```markdown
---
description: Create a well-formatted conventional git commit
---

# Commit: Create Git Commit

## Process

### 1. Check Current State
```bash
git status
git diff HEAD
```

### 2. Review Changes
Summarize what's being committed.

### 3. Stage Files
Stage relevant files. Exclude:
- `.env` files
- `node_modules/`, `__pycache__/`, `.venv/`
- Build artifacts (`dist/`, `build/`)
- IDE files (`.idea/`, `.vscode/` unless intended)
- Secrets or credentials

### 4. Determine Commit Type
| Type | Use When |
|------|----------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code change that neither fixes nor adds |
| `docs` | Documentation only |
| `style` | Formatting, no code change |
| `test` | Adding/updating tests |
| `chore` | Maintenance tasks |

### 5. Create Commit Message
Format: `{type}({scope}): {description}`

- `scope`: Service or component name
- `description`: Imperative mood, lowercase, no period

Examples:
```
feat(api): add user authentication endpoint
fix(frontend): resolve date picker timezone issue
refactor(accounts): simplify transaction validation
```

### 6. Commit
```bash
git add {files}
git commit -m "{type}({scope}): {description}"
```

## Multi-Service Commits
If changes span services, either:
1. **Single commit** if changes are atomic: `feat(api,frontend): add user profile feature`
2. **Separate commits** if changes can be independent

## Output
```
Committed: {type}({scope}): {description}
Files: {count} files changed
Branch: {branch-name}

Next: Push with `git push` or continue development
```
```

#### `.claude/commands/refactor.md`

`````markdown
---
description: Behavior-preserving refactor of a targeted area, run after development
---

# Refactor: Clean a Targeted Area

Improve the internal structure of `$ARGUMENTS` without changing what it does.
Run this after development is complete and validated — never mid-feature.

## Arguments

`$ARGUMENTS` - Target file, directory, or service (required)

## The Iron Rule

**Preserve behavior exactly.** No functional changes, no API changes visible
outside the target, no "while I'm here" fixes. If something looks like a bug,
it goes on the Review List — fixing it here would hide a behavior change
inside a refactor commit.

## Step 0: Green Baseline (gate)

1. Run the target's validation commands (tests, lint, build).
2. **If validation fails: STOP.** Report the failures and exit. Refactoring on
   a red baseline makes behavior preservation unprovable.
3. Note which parts of the target have test coverage. Uncovered code gets only
   provably-safe transforms (Pass 1 deletions confirmed by search, renames of
   symbols whose call sites are all inside the target); everything riskier is
   flagged, not changed.

## Refactoring Passes (in this order)

Run the passes in order — deletion first shrinks everything later passes must
read. After each pass: run validation, then commit (`refactor: pass N — {summary}`).
One pass per commit keeps the history bisectable and each step revertable.

### Pass 1: Delete
- Unused functions, methods, classes (verify zero references with Grep across
  the whole repo, not just the target)
- Unreachable branches, unused imports, commented-out code blocks
- If you cannot *prove* something is dead (dynamic dispatch, reflection,
  external callers, config-driven entry points) → Review List, don't delete.

### Pass 2: Consolidate
- Duplicate and near-duplicate logic → one shared helper, callers updated
- Place helpers at the lowest shared level; don't create a `utils` dumping ground
- Near-duplicates that differ in subtle ways worth keeping → Review List with
  the difference named

### Pass 3: Restructure
- Reduce nesting: early returns, guard clauses, invert conditions
- Split god functions/classes: one responsibility per unit
- Separate concerns: I/O (network, disk, DB) apart from business logic so the
  logic is testable without mocks
- Organize: logical ordering within files, split oversized files, clear module
  boundaries — follow the codebase's existing conventions

### Pass 4: Clarify
- Rename for intention: `d` → `days_until_expiry`. Only rename symbols whose
  call sites all live inside the target; public/exported names → Review List
- Magic numbers/strings → named constants
- Simplify clever/dense code into explicit readable code (a one-liner that
  needs a comment to decode becomes three clear lines)
- Add type hints and docstrings — docstrings say *why*, not *what*; skip
  docstrings that restate the signature

## Flag, Don't Fix

Anything below goes on the Review List, never silently changed:
- Suspicious logic or a possible bug
- Dead code you can't prove dead
- Public API renames or signature changes that would improve clarity
- Behavior that looks unintended but is covered by a passing test

## Step Final: Verify

Apply `verification-before-completion`: re-run the FULL validation suite
fresh, read the output, and confirm the diff contains no behavior change
(no test files weakened, no assertions removed, no config values altered).

## Output Report

```markdown
## Refactor Report: {target}

### Baseline
- Validation before: PASS ({command output summary})
- Coverage gaps: {areas limited to safe transforms}

### Changes by Pass
| Pass | Commits | Summary |
|------|---------|---------|
| 1 Delete | {sha} | {n} dead functions, {n} unused imports removed |
| 2 Consolidate | {sha} | {duplicates merged into which helpers} |
| 3 Restructure | {sha} | {functions split, nesting reduced where} |
| 4 Clarify | {sha} | {renames, constants, types/docs added} |

### Metrics
- Lines: {before} → {after}
- Files: {before} → {after}

### Review List (flagged, NOT changed)
- [ ] `{file}:{line}` — {suspicious logic / uncertain dead code / public rename candidate}: {why flagged}

### Validation After
{fresh full-suite output — actual run, not recalled}
```
`````

#### `.claude/commands/init-project.md`

```markdown
---
description: Initialize Claude Code setup for a new or existing project
---

# Initialize Project

Set up Claude Code structure for a new or existing project.

## Arguments
`$ARGUMENTS` - Project type: "monorepo" or "multirepo"

## Process

### 1. Detect Existing Structure
- Check for existing files/folders
- Identify services/packages
- Detect tech stack

### 2. Create Directory Structure
```bash
mkdir -p .claude/agents
mkdir -p .claude/commands/core_piv_loop
mkdir -p .claude/commands/validation
mkdir -p .claude/commands/github_bug_fix
mkdir -p .claude/reference
mkdir -p .agents/plans
```

### 3. Create Core Files
Based on detected structure, create:
- Root `CLAUDE.md` with project overview
- `.claude/PRD.md` template
- Service `CLAUDE.md` files

### 4. Create Commands
Copy standard commands from template.

### 5. Create Reference Docs
Based on detected tech stack, create relevant best practice docs.

## Output
```markdown
## Project Initialized

### Structure Created
- [x] .claude/agents/
- [x] .claude/commands/
- [x] .claude/reference/
- [x] .agents/plans/

### Files Created
- [x] CLAUDE.md
- [x] .claude/PRD.md (template)
- [x] {service}/CLAUDE.md

### Commands Available
- /core_piv_loop:prime
- /core_piv_loop:plan-feature
- /core_piv_loop:execute
- /validation:validate
- /validation:code-review
- /commit

### Next Steps
1. Edit `CLAUDE.md` with project specifics
2. Edit `.claude/PRD.md` with requirements
3. Review and customize commands
4. Run `/core_piv_loop:prime` to verify setup
```
```

#### `.claude/commands/create-prd.md`

```markdown
---
description: Create or update the Product Requirements Document
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
```

---

### Step 6: Create Reference Documentation

Create `.claude/reference/{technology}-best-practices.md` for your tech stack.

Example for Node.js/TypeScript:

```markdown
# Node.js/TypeScript Best Practices

## Project Structure
```
src/
├── index.ts          # Entry point, minimal code
├── config/           # Configuration loading
├── routes/           # Express/Fastify routes
├── services/         # Business logic
├── models/           # Data models/types
├── middleware/       # Request middleware
└── utils/            # Pure utility functions
```

## Coding Standards

### Async/Await
Always use async/await over callbacks or raw promises:
```typescript
// Good
async function getUser(id: string): Promise<User> {
  const user = await db.users.findById(id);
  return user;
}

// Avoid
function getUser(id: string): Promise<User> {
  return db.users.findById(id).then(user => user);
}
```

### Error Handling
Use custom error classes:
```typescript
class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code: string = 'INTERNAL_ERROR'
  ) {
    super(message);
  }
}

// Usage
throw new AppError('User not found', 404, 'USER_NOT_FOUND');
```

### Type Safety
- Enable strict mode in tsconfig.json
- Avoid `any` - use `unknown` if type is truly unknown
- Define interfaces for all API contracts

## Testing
- Unit tests: `*.test.ts` next to source files
- Integration tests: `tests/integration/`
- Use describe/it blocks with clear descriptions
```

---

### Step 7: Create Settings (Optional)

Create `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run:*)",
      "Bash(npm install:*)",
      "Bash(npm test:*)",
      "Bash(npx:*)",
      "Bash(git:*)",
      "Bash(python:*)",
      "Bash(pytest:*)",
      "Bash(pip:*)",
      "Bash(uv:*)",
      "Bash(node:*)",
      "Bash(ls:*)",
      "Bash(mkdir:*)",
      "Bash(tree:*)",
      "Bash(cat:*)",
      "Bash(head:*)",
      "Bash(tail:*)"
    ],
    "deny": []
  }
}
```

---

## Injecting into Existing Projects

When adding this setup to an existing project:

### 1. Assess Current State
- Document existing services and their purposes
- Identify the tech stack per service
- Note any existing documentation

### 2. Create Minimal Structure
```bash
mkdir -p .claude/commands .claude/reference .agents/plans
```

### 3. Document What Exists
Create `CLAUDE.md` that accurately describes:
- Current architecture
- Existing patterns and conventions
- How to run/test each service

### 4. Add Commands Incrementally
Start with:
1. `/core_piv_loop:prime` - to load context
2. `/validation:validate` - to run tests
3. `/commit` - for consistent commits

Add more commands as needed.

### 5. Create Reference Docs
Document the patterns already in use, not ideal patterns. This helps Claude follow existing conventions.

### 6. Update .gitignore
Consider whether to commit Claude config:
```gitignore
# Option A: Keep Claude config private
.claude/
.agents/

# Option B: Share commands, keep plans private
.agents/plans/
```

---

## Skills & Commands Reference

### How Skills Work
Commands in `.claude/commands/` become slash commands (skills):

| File Location | Command |
|--------------|---------|
| `.claude/commands/commit.md` | `/commit` |
| `.claude/commands/core_piv_loop/prime.md` | `/core_piv_loop:prime` |
| `.claude/commands/validation/validate.md` | `/validation:validate` |

### Command File Requirements
1. **YAML frontmatter** with `description` field (required)
2. **Markdown content** with instructions
3. **`$ARGUMENTS`** placeholder for user input

### Available Commands Summary

| Command | Purpose |
|---------|---------|
| `/core_piv_loop:prime` | Load project/service context |
| `/core_piv_loop:plan-feature` | Create implementation plan |
| `/core_piv_loop:execute` | Execute a plan |
| `/validation:validate` | Run tests, lint, build |
| `/validation:code-review` | Pre-commit review |
| `/validation:code-review-fix` | Auto-fix review issues |
| `/validation:execution-report` | Generate implementation report |
| `/github_bug_fix:rca` | Root cause analysis |
| `/github_bug_fix:implement-fix` | Implement bug fix |
| `/commit` | Create conventional commit |
| `/feature` | Track feature lifecycle |
| `/init-project` | Initialize Claude setup |
| `/create-prd` | Create/update PRD |

### Typical Workflow

```
1. /core_piv_loop:prime backend,frontend
2. /core_piv_loop:plan-feature "Add user authentication"
3. Review plan in .agents/plans/
4. /core_piv_loop:execute .agents/plans/add-user-authentication.md
5. /validation:validate all
6. /validation:code-review
7. /commit
```

---

## File Templates

### Minimal CLAUDE.md (for quick setup)
```markdown
# {Project Name}

## Services
- `{service1}/` - {purpose}
- `{service2}/` - {purpose}

## Commands
```bash
cd {service1} && npm run dev
cd {service2} && npm run dev
```

## Tech Stack
{frontend}: {tech}
{backend}: {tech}
{database}: {tech}
```

### Minimal Service CLAUDE.md
```markdown
# {Service Name}

{One-line purpose}

## Stack
{runtime}, {framework}

## Commands
```bash
npm install && npm run dev
npm test
```

## Key Files
- `src/index.ts` - Entry point
- `src/routes/` - API routes
```

---

## Customization Tips

1. **Adapt commands to your workflow** - Modify the command templates to match how you work
2. **Keep CLAUDE.md files current** - Update them when architecture changes
3. **Reference docs should reflect reality** - Document actual patterns, not aspirational ones
4. **Start simple** - Begin with core commands, add more as needed
5. **Iterate** - Refine commands based on what works for your projects
