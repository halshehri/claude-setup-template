---
name: bootstrap
description: Bootstrap a Claude Code project — generates skills, agents, commands, hooks, MCP, and settings tailored to the user's stack. Use when the user says "set up Claude Code", "initialize project", "bootstrap", or points Claude at this template repo.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebFetch, AskUserQuestion
---

# Claude Setup Bootstrap

This skill is the entry point for the claude-setup-template plugin. The full orchestration logic, question flow, and template inventory live in `bootstrap/SKILL.md` at the repo root.

## How to run

1. Read `bootstrap/SKILL.md` for the 5-phase process (Discovery → Requirements → Generation → Injection → Verification).
2. Read `bootstrap/questions.md` for the question flow.
3. Use templates from `bootstrap/templates/` (skills, agents, commands, claude-md, hooks, mcp, settings, reference).

Follow `bootstrap/SKILL.md` exactly — it is the source of truth.
