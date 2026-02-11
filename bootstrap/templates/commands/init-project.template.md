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
