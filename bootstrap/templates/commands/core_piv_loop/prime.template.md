---
description: Load project context for specified services before planning or implementing
argument-hint: "[service-names]"
allowed-tools: Read, Glob, Grep, Bash(git status:*), Bash(git log:*)
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
