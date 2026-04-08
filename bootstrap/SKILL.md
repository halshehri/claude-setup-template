---
name: claude-setup-bootstrap
description: Initialize Claude Code setup for a new or existing project. Use when setting up Claude Code configuration, creating project structure, or when user says "set up Claude Code", "initialize project", or "bootstrap".
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebFetch, AskUserQuestion
---

# Claude Code Project Bootstrap

You are initializing a Claude Code project setup. This skill guides you through collecting requirements and generating a customized configuration.

## Overview

This bootstrap process will:
1. Ask questions about the project
2. Analyze existing code (if brownfield)
3. Generate customized Skills, Agents, Commands, and CLAUDE.md files

## Process

### Phase 1: Project Discovery

First, determine if this is a new or existing project:

```
Is this a new (greenfield) project or an existing (brownfield) project?
```

**If Brownfield:**
- Run analysis first: `git ls-files`, check for package.json, pyproject.toml, etc.
- Identify existing services/packages
- Detect tech stack automatically
- Ask user to confirm/correct your findings
- Plan injection points (don't overwrite existing files)

**If Greenfield:**
- Proceed directly to questions

### Phase 2: Collect Requirements

Use the AskUserQuestion tool to gather information. See [questions.md](questions.md) for the complete question flow.

**Required Information:**
1. Project type (Monorepo / Multi-repo microservices)
2. Project name
3. Services/packages (names and purposes)
4. Tech stack per service
5. Database(s) used
6. Any specific patterns or conventions to follow

### Phase 3: Generate Configuration

Based on collected information, generate files from templates in `bootstrap/templates/`.

#### Directory Structure to Create

```
{project-root}/
├── .agents/
│   └── plans/
├── .claude/
│   ├── agents/
│   │   ├── senior-architect.md
│   │   └── fullstack-engineer.md
│   ├── commands/
│   │   ├── core_piv_loop/
│   │   │   ├── prime.md
│   │   │   ├── plan-feature.md
│   │   │   └── execute.md
│   │   ├── validation/
│   │   │   ├── validate.md
│   │   │   ├── code-review.md
│   │   │   ├── code-review-fix.md
│   │   │   └── execution-report.md
│   │   ├── github_bug_fix/
│   │   │   ├── rca.md
│   │   │   └── implement-fix.md
│   │   ├── commit.md
│   │   ├── feature.md
│   │   ├── init-project.md
│   │   └── create-prd.md
│   ├── skills/
│   │   └── {generated based on tech stack}
│   ├── reference/
│   │   ├── {generated based on tech stack}
│   │   └── adk/              # (if Google ADK selected)
│   └── PRD.md
├── CLAUDE.md
└── {service}/
    └── CLAUDE.md
```

#### File Generation Rules

**Skills** - Generate based on tech stack:
| Tech Stack | Skills to Generate |
|------------|-------------------|
| Node.js/TypeScript | `nodejs-coding` |
| Python | `python-coding` |
| React/Vue/Frontend | `frontend-development` |
| PostgreSQL | `postgres` |
| MongoDB | `mongodb` |
| Google ADK | `google-adk` |
| Always | `solution-architect` |

**Reference Files** - For Google ADK, the deep reference docs ship inside the skill folder (`templates/skills/google-adk/reference/`) and are copied with the skill — no separate handling needed.

**Agents** - Always generate:
- `senior-architect.md` (Opus model, assigned: solution-architect skill + relevant tech skills)
- `fullstack-engineer.md` (Sonnet model, assigned: all tech skills for the project)

**Commands** - Always generate the full set from templates.

**CLAUDE.md files**:
- Root: Use monorepo or multirepo template based on project type
- Services: Generate one per service with detected/provided info

### Phase 4: Brownfield Injection

For existing projects, follow these rules:

1. **Never overwrite** existing files without asking
2. **Check for conflicts**:
   - If `.claude/` exists, ask before modifying
   - If `CLAUDE.md` exists, offer to merge or backup
3. **Preserve existing structure** - add files alongside, don't reorganize
4. **Add to .gitignore** if needed:
   ```
   .agents/plans/
   ```

### Phase 5: Verification

After generating files:

1. List all created files
2. Show summary of configuration
3. Suggest next steps:
   ```
   ## Setup Complete!

   ### Files Created
   - .claude/agents/senior-architect.md
   - .claude/agents/fullstack-engineer.md
   - .claude/skills/{skills}
   - .claude/commands/{commands}
   - CLAUDE.md
   - {service}/CLAUDE.md

   ### Next Steps
   1. Review and customize CLAUDE.md files
   2. Edit .claude/PRD.md with your requirements
   3. Test with: /core_piv_loop:prime

   ### Available Commands
   - /core_piv_loop:prime - Load project context
   - /core_piv_loop:plan-feature - Plan a feature
   - /validation:validate - Run validation
   - /commit - Create git commit
   ```

## Template Locations

All templates are in `bootstrap/templates/`:

- `skills/` - Folder-based skills: `skills/{name}/SKILL.template.md` with `allowed-tools` frontmatter (generated to `.claude/skills/{name}/SKILL.md`)
- `agents/` - Agent templates
- `commands/` - Command templates
- `claude-md/` - CLAUDE.md templates
- `reference/` - Reference documentation templates (e.g., `reference/adk/`)
- `settings/` - settings.json template (permissions, env, model)
- `hooks/` - Hook fragments to merge into settings.json (format-on-edit, block-secrets, session-start-context, auto-validate, stop-execution-report)
- `mcp/` - `.mcp.json` template with commented filesystem/github/postgres/sentry servers

## Plugin Distribution

This repo is also packaged as a Claude Code plugin via `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`. Users can install it with `claude plugin install` instead of cloning.

## Phase 3b: Harness Configuration

After generating skills/agents/commands, also offer to install:
1. **settings.json** — copy from `templates/settings/settings.local.template.json`, substitute `{{TEST_COMMAND}}`, `{{BUILD_COMMAND}}`, `{{LINT_COMMAND}}`
2. **Hooks** — ask which hooks to enable; merge selected fragments from `templates/hooks/` into `settings.json` under the `hooks` key
3. **MCP servers** — ask which external systems to wire (github, postgres, sentry, filesystem); copy `templates/mcp/.mcp.template.json` to `.mcp.json` and uncomment chosen servers

## Variable Substitution

When generating from templates, replace these variables:

| Variable | Description |
|----------|-------------|
| `{{PROJECT_NAME}}` | Project name |
| `{{PROJECT_TYPE}}` | "Monorepo" or "Multi-repo Microservices" |
| `{{SERVICES_TABLE}}` | Generated services table |
| `{{TECH_STACK}}` | Tech stack summary |
| `{{SERVICE_NAME}}` | Individual service name |
| `{{SERVICE_PURPOSE}}` | Service description |
| `{{SERVICE_TECH}}` | Service tech stack |
| `{{SKILLS_LIST}}` | Comma-separated skills for agents |

## Error Handling

- If template file not found, use inline defaults
- If user cancels, save progress and offer to resume
- If file write fails, report error and continue with others
