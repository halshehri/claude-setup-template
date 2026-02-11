# Claude Setup Template

This repository is a **template/tool**, not a runnable project. It bootstraps Claude Code configurations for any project.

## What This Repo Does

When a user points Claude Code at this repo (or its GitHub URL), the bootstrap skill:
1. Asks questions about the target project (type, services, tech stack)
2. Analyzes existing code if brownfield
3. Generates customized `.claude/` configuration files
4. Creates CLAUDE.md files, commands, skills, agents, and reference docs

## Repository Structure

```
claude-setup-template/
├── CLAUDE.md                       # This file
├── README.md                       # User-facing documentation
├── SETUP_TEMPLATE.md               # Detailed manual setup guide
├── .claude/
│   └── settings.local.json         # Template repo permissions
├── .gitignore
└── bootstrap/
    ├── SKILL.md                    # Bootstrap orchestration skill (entry point)
    ├── questions.md                # Question flow for project discovery
    └── templates/
        ├── skills/                 # Tech stack skill templates
        │   ├── solution-architect.template.md
        │   ├── nodejs-coding.template.md
        │   ├── python-coding.template.md
        │   ├── react-frontend.template.md
        │   ├── postgres.template.md
        │   ├── mongodb.template.md
        │   └── google-adk.template.md
        ├── agents/                 # Agent definition templates
        │   ├── senior-architect.template.md
        │   └── fullstack-engineer.template.md
        ├── commands/               # Command (slash command) templates
        │   ├── core_piv_loop/      # Prime → Plan → Execute workflow
        │   │   ├── prime.template.md
        │   │   ├── plan-feature.template.md
        │   │   └── execute.template.md
        │   ├── validation/         # Testing & review commands
        │   │   ├── validate.template.md
        │   │   ├── code-review.template.md
        │   │   ├── code-review-fix.template.md
        │   │   └── execution-report.template.md
        │   ├── github_bug_fix/     # Bug investigation & fix
        │   │   ├── rca.template.md
        │   │   └── implement-fix.template.md
        │   ├── commit.template.md
        │   ├── feature.template.md
        │   ├── init-project.template.md
        │   └── create-prd.template.md
        ├── claude-md/              # CLAUDE.md templates
        │   ├── root-monorepo.template.md
        │   ├── root-multirepo.template.md
        │   └── service.template.md
        ├── reference/              # Deep reference documentation
        │   └── adk/                # Google ADK reference files
        │       ├── adk-fundamentals.md
        │       ├── adk-agents.md
        │       └── STATUS.md
        └── gitignore.template      # .gitignore template for target projects
```

## How Bootstrap Works

The process is driven by `bootstrap/SKILL.md` (5 phases):

1. **Discovery** - Greenfield vs brownfield, auto-detect existing code
2. **Requirements** - Ask questions from `bootstrap/questions.md`
3. **Generation** - Create files from `bootstrap/templates/` with variable substitution
4. **Injection** - For brownfield: merge without overwriting existing files
5. **Verification** - List created files, suggest next steps

## Template Conventions

- **Skills** (`skills/*.template.md`): YAML frontmatter with `name` + `description`, auto-applied by Claude when relevant
- **Commands** (`commands/*.template.md`): YAML frontmatter with `description`, invoked via `/command-name`
- **Agents** (`agents/*.template.md`): YAML frontmatter with `name`, `description`, `model`, `color`
- **Reference files** (`reference/`): Not auto-loaded; read on-demand for deep knowledge
- All templates use `{{VARIABLE}}` syntax for substitution during generation

## Key Entry Points

- `bootstrap/SKILL.md` - Start here. Orchestrates the entire bootstrap process.
- `bootstrap/questions.md` - Question flow and auto-detection logic.
- `SETUP_TEMPLATE.md` - Complete manual reference with all file content inline.

## File Counts

- 13 command templates in `bootstrap/templates/commands/`
- 7 skill templates in `bootstrap/templates/skills/`
- 2 agent templates in `bootstrap/templates/agents/`
- 3 CLAUDE.md templates in `bootstrap/templates/claude-md/`
- 3 ADK reference files in `bootstrap/templates/reference/adk/`
- 1 gitignore template
