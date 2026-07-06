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
        ├── skills/                 # Folder-based skills (SKILL.template.md inside)
        │   ├── solution-architect/SKILL.template.md
        │   ├── verification-before-completion/SKILL.template.md
        │   ├── nodejs-coding/SKILL.template.md
        │   ├── python-coding/SKILL.template.md
        │   ├── react-frontend/SKILL.template.md
        │   ├── postgres/SKILL.template.md
        │   ├── mongodb/SKILL.template.md
        │   └── google-adk/
        │       ├── SKILL.template.md
        │       └── reference/      # 7 deep-dive files loaded on demand
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
        ├── settings/               # settings.json template
        │   └── settings.local.template.json
        ├── hooks/                  # Hook fragments to merge into settings.json
        │   ├── format-on-edit.template.json
        │   ├── block-secrets.template.json
        │   ├── session-start-context.template.json
        │   ├── auto-validate.template.json
        │   └── stop-execution-report.template.json
        ├── mcp/                    # .mcp.json template
        │   └── .mcp.template.json
        └── gitignore.template      # .gitignore template for target projects

.claude-plugin/                     # Plugin packaging (installable via `claude plugin install`)
├── plugin.json
└── marketplace.json
```

## How Bootstrap Works

The process is driven by `bootstrap/SKILL.md` (5 phases):

1. **Discovery** - Greenfield vs brownfield, auto-detect existing code
2. **Requirements** - Ask questions from `bootstrap/questions.md`
3. **Generation** - Create files from `bootstrap/templates/` with variable substitution
4. **Injection** - For brownfield: merge without overwriting existing files
5. **Verification** - List created files, suggest next steps

## Template Conventions

- **Skills** (`skills/{name}/SKILL.template.md`): folder-based, frontmatter `name` + `description` + `allowed-tools`; auto-applied by Claude when relevant. Drop additional `.md` files alongside for progressive disclosure.
- **Commands** (`commands/*.template.md`): YAML frontmatter with `description` (also supports `argument-hint`, `allowed-tools`, `model`); invoked via `/command-name`
- **Agents** (`agents/*.template.md`): YAML frontmatter with `name`, `description`, `model`, `tools`, optional `color`
- **Hooks** (`hooks/*.template.json`): JSON fragments merged into the target's `settings.json` under the `hooks` key
- **MCP** (`mcp/.mcp.template.json`): copied to target repo root as `.mcp.json`
- **Settings** (`settings/settings.local.template.json`): copied to `.claude/settings.local.json`
- **Reference files** (`reference/`): Not auto-loaded; read on-demand for deep knowledge
- All templates use `{{VARIABLE}}` syntax for substitution during generation

## Key Entry Points

- `bootstrap/SKILL.md` - Start here. Orchestrates the entire bootstrap process.
- `bootstrap/questions.md` - Question flow and auto-detection logic.
- `SETUP_TEMPLATE.md` - Complete manual reference with all file content inline.

## File Counts

- 13 command templates in `bootstrap/templates/commands/`
- 8 skill templates in `bootstrap/templates/skills/` (folder-based)
- 2 agent templates in `bootstrap/templates/agents/`
- 3 CLAUDE.md templates in `bootstrap/templates/claude-md/`
- 7 ADK reference files in `bootstrap/templates/skills/google-adk/reference/` (progressive disclosure)
- 5 hook templates in `bootstrap/templates/hooks/`
- 1 settings template in `bootstrap/templates/settings/`
- 1 MCP template in `bootstrap/templates/mcp/`
- 1 gitignore template
- Plugin manifest at `.claude-plugin/`
