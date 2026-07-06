# Claude Code Setup Template

A comprehensive template for setting up Claude Code in your projects. This repository provides a bootstrap system that generates customized Skills, Agents, Commands, and documentation based on your project's tech stack.

## Quick Start

### Option 1: Ask Claude to Set Up Your Project

In your project directory, tell Claude:

```
Connect to https://github.com/halshehri/claude-setup-template and set up Claude Code for this project.
```

Claude will:
1. Ask questions about your project (type, services, tech stack)
2. Analyze existing code (if brownfield)
3. Generate customized configuration files

### Option 2: Manual Setup

1. Clone this repository
2. Copy the relevant templates to your project
3. Customize the placeholders

## What Gets Generated

```
your-project/
├── .agents/
│   └── plans/                    # Implementation plans
├── .claude/
│   ├── agents/                   # AI agent definitions
│   │   ├── senior-architect.md   # Design & planning (Opus)
│   │   └── fullstack-engineer.md # Implementation (Sonnet)
│   ├── commands/                 # Slash commands
│   │   ├── core_piv_loop/        # Prime → Plan → Execute workflow
│   │   ├── validation/           # Testing & code review
│   │   ├── github_bug_fix/       # Bug investigation & fixes
│   │   └── commit.md             # Git commits
│   ├── skills/                   # Domain expertise (auto-applied)
│   │   ├── solution-architect/   # Architecture thinking
│   │   ├── verification-before-completion/ # Evidence before "done" claims
│   │   ├── nodejs-coding/        # Node.js patterns (if applicable)
│   │   ├── python-coding/        # Python patterns (if applicable)
│   │   ├── react-frontend/       # React patterns (if applicable)
│   │   ├── postgres/             # PostgreSQL patterns (if applicable)
│   │   └── mongodb/              # MongoDB patterns (if applicable)
│   ├── reference/                # Best practices docs
│   └── PRD.md                    # Product requirements
├── CLAUDE.md                     # Project overview
└── {service}/
    └── CLAUDE.md                 # Service-specific context
```

## Project Types Supported

### Monorepo
Single repository containing all services.

```
project/                  ← has .git
├── .claude/
├── CLAUDE.md
├── backend/
└── frontend/
```

### Multi-repo Microservices
Parent folder with separate repos per service.

```
org-parent/               ← NO .git
├── .claude/              ← shared config
├── CLAUDE.md
├── api-gateway/          ← has .git
├── user-service/         ← has .git
└── frontend/             ← has .git
```

## Available Commands

After setup, these slash commands are available:

| Command | Purpose |
|---------|---------|
| `/core_piv_loop:prime` | Load project/service context |
| `/core_piv_loop:plan-feature` | Create implementation plan |
| `/core_piv_loop:execute` | Execute a plan |
| `/validation:validate` | Run tests, lint, build |
| `/validation:code-review` | Pre-commit code review |
| `/github_bug_fix:rca` | Root cause analysis |
| `/github_bug_fix:implement-fix` | Implement bug fix |
| `/commit` | Create conventional commit |
| `/feature` | Track feature lifecycle (requirement → design → implementation → testing → merged) |

## Typical Workflow

```bash
# 1. Load context for services you'll work with
/core_piv_loop:prime backend,frontend

# 2. Plan your feature
/core_piv_loop:plan-feature "Add user authentication"

# 3. Review the plan
# (Check .agents/plans/add-user-authentication.md)

# 4. Execute the plan
/core_piv_loop:execute .agents/plans/add-user-authentication.md

# 5. Validate
/validation:validate all

# 6. Code review
/validation:code-review

# 7. Commit
/commit
```

## Skills vs Commands vs Agents

| Component | What It Is | When It's Used |
|-----------|------------|----------------|
| **Skills** | Domain expertise | Automatically when relevant |
| **Commands** | Explicit workflows | When you type `/command` |
| **Agents** | Specialized workers | Delegated by Claude |

### Skills (Auto-Applied)
Claude automatically applies skills when relevant:
- `solution-architect` - When designing systems
- `verification-before-completion` - Before any "done"/"passing" claim: run the command, read the output, then claim
- `nodejs-coding` - When writing Node.js code
- `postgres` - When working with PostgreSQL

### Commands (User-Invoked)
You explicitly invoke commands:
- `/core_piv_loop:prime` - Load context
- `/validation:validate` - Run checks

### Agents (Delegated)
Claude delegates to specialized agents:
- `senior-architect` (Opus) - Design decisions
- `fullstack-engineer` (Sonnet) - Implementation

## Tech Stack Templates

The bootstrap includes skill templates for:

- **Backend**: Node.js/TypeScript, Python/FastAPI
- **Frontend**: React, TypeScript
- **Database**: PostgreSQL, MongoDB
- **AI Agents**: Google ADK
- **Architecture**: Solution design patterns

## Customization

### Adding New Skills

Create `.claude/skills/{skill-name}/SKILL.md`:

```markdown
---
name: my-custom-skill
description: When to apply this skill
---

# My Custom Skill

{Instructions for Claude}
```

### Adding New Commands

Create `.claude/commands/{command-name}.md`:

```markdown
---
description: What this command does
---

# Command Name

## Arguments
`$ARGUMENTS` - Description

## Process
{Steps Claude should follow}
```

### Modifying Agents

Edit `.claude/agents/{agent-name}.md` to change:
- Model (opus/sonnet)
- Assigned skills
- Behavior instructions

## Repository Structure

```
claude-setup-template/
├── CLAUDE.md                 # Repo overview for Claude
├── README.md                 # This file
├── SETUP_TEMPLATE.md         # Detailed documentation
└── bootstrap/
    ├── SKILL.md              # Bootstrap orchestration skill
    ├── questions.md          # Question flow reference
    └── templates/
        ├── skills/           # Skill templates
        ├── agents/           # Agent templates
        ├── commands/         # Command templates
        ├── claude-md/        # CLAUDE.md templates
        └── reference/        # Reference documentation templates
            └── adk/          # Google ADK deep reference
```

## Contributing

Contributions welcome! Areas for improvement:
- Additional tech stack templates (Go, Rust, Vue, Angular)
- More specialized skills
- Better brownfield detection
- CI/CD integration commands

## License

MIT

---

Created by [@halshehri](https://github.com/halshehri)
