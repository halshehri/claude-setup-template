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
│   ├── skills/                       # ONE FOLDER PER SKILL — never a flat .md
│   │   ├── solution-architect/
│   │   │   └── SKILL.md
│   │   └── {skill-name}/             # per tech stack
│   │       ├── SKILL.md
│   │       └── reference/            # optional (e.g. google-adk ships 7 files)
│   └── PRD.md
├── CLAUDE.md
└── {service}/
    └── CLAUDE.md
```

#### File Generation Rules

**Skills** - Generate based on tech stack:
| Tech Stack | Skill (template folder name) |
|------------|------------------------------|
| Node.js/TypeScript | `nodejs-coding` |
| Python | `python-coding` |
| React/Vue/Frontend | `react-frontend` |
| PostgreSQL | `postgres` |
| MongoDB | `mongodb` |
| Google ADK | `google-adk` |
| Always | `solution-architect` |

> **CRITICAL — skill file layout.** Claude Code only discovers a skill at
> `.claude/skills/<name>/SKILL.md`. A flat `.claude/skills/<name>.md` is silently
> ignored: it never appears in the available-skills list and any agent that names it
> in frontmatter points at nothing.
>
> For each selected skill, copy the whole template folder:
>
> ```
> bootstrap/templates/skills/<name>/SKILL.template.md  →  .claude/skills/<name>/SKILL.md
> bootstrap/templates/skills/<name>/reference/*        →  .claude/skills/<name>/reference/*
> ```
>
> Create the `<name>/` directory first. Never rename `SKILL.md`, never flatten the
> folder, and never drop the YAML frontmatter — a skill without `name` and
> `description` also fails to load.

**Reference Files** - For Google ADK, the deep reference docs ship inside the skill folder (`templates/skills/google-adk/reference/`) and are copied with the skill — no separate handling needed.

**Agents** - Always generate:
- `senior-architect.md` (Opus model, assigned: solution-architect skill + relevant tech skills)
- `fullstack-engineer.md` (Sonnet model, assigned: all tech skills for the project)

> The agents' `skills:` frontmatter must list **only skills actually generated in this
> run** (plus plugin skills whose plugin is installed). A name that does not resolve to
> a `.claude/skills/<name>/SKILL.md` — or to an installed plugin's `<plugin>:<skill>` —
> is dead weight in the agent definition. Substitute `{{TECH_SKILLS}}` with the
> comma-separated list of generated skill names.

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

1. **Verify skill discovery** — for every generated skill, confirm all three hold:
   - `.claude/skills/<name>/SKILL.md` exists (a flat `.claude/skills/<name>.md` means it will not load — move it into its folder)
   - its frontmatter has both `name:` and `description:`
   - every name in each agent's `skills:` frontmatter resolves to one of those folders

   ```bash
   ls .claude/skills/*.md 2>/dev/null && echo "BROKEN: flat skill files above — move each to .claude/skills/<name>/SKILL.md"
   for d in .claude/skills/*/; do
     [ -f "$d/SKILL.md" ] || echo "BROKEN: $d has no SKILL.md"
   done
   ```

2. List all created files
3. Show summary of configuration
4. Suggest next steps:
   ```
   ## Setup Complete!

   ### Files Created
   - .claude/agents/senior-architect.md
   - .claude/agents/fullstack-engineer.md
   - .claude/skills/{skill}/SKILL.md  (one folder per skill)
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
- `settings/` - settings.json template (permissions, env, model)
- `hooks/` - Hook fragments to merge into settings.json (format-on-edit, session-start-context, stop-execution-report)
- `mcp/` - `.mcp.json` template with commented filesystem/github/postgres/sentry servers

## Plugin Distribution

This repo is also packaged as a Claude Code plugin via `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`. Users can install it with `claude plugin install` instead of cloning.

## Phase 3b: Harness Configuration

After generating skills/agents/commands, also offer to install:
1. **settings.json** — copy from `templates/settings/settings.local.template.json`, substitute `{{TEST_COMMAND}}`, `{{BUILD_COMMAND}}`, `{{LINT_COMMAND}}`
2. **Hooks** — ask which hooks to enable; merge selected fragments from `templates/hooks/` into `settings.json` under the `hooks` key
3. **MCP servers** — ask which external systems to wire (github, postgres, sentry, filesystem); copy `templates/mcp/.mcp.template.json` to `.mcp.json` and uncomment chosen servers

## Variable Substitution

Every `{{VARIABLE}}` in a template MUST be substituted before the generated file is
written. An unsubstituted placeholder ships a literal `{{API_ENDPOINTS}}` into the user's
CLAUDE.md. If a value is genuinely unknown, write a short honest placeholder line
(e.g. `_TBD — not detected during bootstrap._`) rather than leaving the braces.

**Verify before finishing:**

```bash
grep -rn "{{[A-Z_]*}}" .claude CLAUDE.md */CLAUDE.md 2>/dev/null && echo "BROKEN: unsubstituted variables above"
```

### Shared

| Variable | Used in | Description |
|----------|---------|-------------|
| `{{PROJECT_NAME}}` | root CLAUDE.md, settings | Project name |
| `{{TECH_STACK}}` | root CLAUDE.md | Tech stack summary |
| `{{SERVICES_TABLE}}` | root CLAUDE.md | Markdown table of services |
| `{{SERVICES}}` | root CLAUDE.md | List of service names |
| `{{SERVICE_CLAUDE_MD_LINKS}}` | root CLAUDE.md | Links to each service CLAUDE.md |
| `{{ARCHITECTURE_DIAGRAM}}` | root CLAUDE.md | ASCII/mermaid architecture sketch |
| `{{DATABASE_TABLES}}` | root CLAUDE.md | Key tables/collections |
| `{{DEV_COMMANDS}}` / `{{BUILD_COMMANDS}}` / `{{TEST_COMMANDS}}` | root CLAUDE.md | Per-service command lists |
| `{{SKILLS_DOCS}}` | root CLAUDE.md | Bulleted list of the skills generated this run |
| `{{TECH_SKILLS}}` | both agents | Comma-separated names of the skills generated this run, for `skills:` frontmatter |

### Multi-repo only

| Variable | Description |
|----------|-------------|
| `{{GITHUB_ORG}}` | GitHub organization owning the service repos |
| `{{SERVICE_REPOS_TABLE}}` | Table mapping service → repo URL |

### Service CLAUDE.md

| Variable | Description |
|----------|-------------|
| `{{SERVICE_NAME}}` | Service name |
| `{{SERVICE_PURPOSE}}` | What the service does |
| `{{RESPONSIBILITIES}}` | Bulleted responsibilities |
| `{{LANGUAGE}}` / `{{RUNTIME}}` / `{{FRAMEWORK}}` / `{{DATABASE}}` | Stack details |
| `{{DEPENDENCIES}}` | Key dependencies |
| `{{INTEGRATIONS}}` | External systems it talks to |
| `{{DIRECTORY_STRUCTURE}}` | Tree of the service source layout |
| `{{KEY_FILES}}` | Notable entry-point files |
| `{{ROUTES_PATH}}` / `{{CONTROLLERS_PATH}}` / `{{SERVICES_PATH}}` / `{{MODELS_PATH}}` | Source subdirectories |
| `{{API_ENDPOINTS}}` | Endpoint list |
| `{{ENV_VARIABLES}}` | Required env vars |
| `{{CODE_CONVENTIONS}}` | Service-specific conventions |
| `{{INSTALL_COMMAND}}` / `{{DEV_COMMAND}}` / `{{BUILD_COMMAND}}` / `{{TEST_COMMAND}}` / `{{LINT_COMMAND}}` | Commands |
| `{{UNIT_TEST_COMMAND}}` / `{{INTEGRATION_TEST_COMMAND}}` | Narrower test commands |

### Harness

| Variable | Used in | Description |
|----------|---------|-------------|
| `{{PROJECT_NAME}}`, `{{BUILD_COMMAND}}`, `{{TEST_COMMAND}}`, `{{LINT_COMMAND}}` | `settings/settings.local.template.json` | Allowed commands |
| `{{FORMAT_COMMAND}}` | `hooks/format-on-edit.template.json` | Formatter binary (prettier, black, gofmt, rustfmt) |

## Error Handling

- If template file not found, use inline defaults
- If user cancels, save progress and offer to resume
- If file write fails, report error and continue with others
