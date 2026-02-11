# {{PROJECT_NAME}}

This file provides guidance for Claude Code. For service-specific details, see the CLAUDE.md in each service directory.

## Project Type

Monorepo - Single repository containing all services.

## Quick Reference

### Services

{{SERVICES_TABLE}}

### Common Commands

```bash
# Development
{{DEV_COMMANDS}}

# Testing
{{TEST_COMMANDS}}

# Build
{{BUILD_COMMANDS}}
```

## Architecture Overview

```
{{ARCHITECTURE_DIAGRAM}}
```

### Data Flow

1. {{DATA_FLOW_STEP_1}}
2. {{DATA_FLOW_STEP_2}}
3. {{DATA_FLOW_STEP_3}}

## Tech Stack

{{TECH_STACK}}

## Claude Code Workflow

### PIV Loop (Prime → Implement → Validate)

```bash
/core_piv_loop:prime {{SERVICES}}     # Load context
/core_piv_loop:plan-feature "{desc}"  # Create implementation plan
/core_piv_loop:execute {plan-path}    # Execute the plan
```

### Validation

```bash
/validation:validate {service}        # Run tests, lint, build
/validation:validate all              # Validate all services
/validation:code-review               # Pre-commit code review
```

### Bug Fixes

```bash
/github_bug_fix:rca {issue-url}       # Root cause analysis
/github_bug_fix:implement-fix         # Implement the fix
```

### Git

```bash
/commit                               # Create conventional commit
```

## Working with Multiple Services

1. **Prime relevant services** with `/core_piv_loop:prime service1,service2`
2. **Plan the feature** with `/core_piv_loop:plan-feature`
3. **Review the plan** in `.agents/plans/`
4. **Execute** with `/core_piv_loop:execute`
5. **Validate each service** after implementation
6. **Commit** with `/commit`

## Key Database Tables

{{DATABASE_TABLES}}

## Environment Variables

See each service's CLAUDE.md for service-specific variables.

## Reference Documentation

See `.claude/reference/` for best practices:
{{REFERENCE_DOCS}}

## Service-Specific Documentation

{{SERVICE_CLAUDE_MD_LINKS}}
