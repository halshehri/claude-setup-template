---
name: bootstrap
description: Bootstrap a Claude Code project — generates skills, agents, commands, hooks, MCP, and settings tailored to the user's stack. Use when the user says "set up Claude Code", "initialize project", "bootstrap", or points Claude at this template repo.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebFetch, AskUserQuestion
---

# Claude Setup Bootstrap

Entry point for the claude-setup-template plugin. The orchestration logic, question flow,
and templates live outside this file — read them before doing anything.

## Resolve the template root first

This skill runs with the working directory set to the **user's project**, not to this
repo. Never read the paths below relative to the CWD — resolve them against the plugin
root first:

```bash
ROOT="${CLAUDE_PLUGIN_ROOT:-.}"   # set by Claude Code when installed as a plugin
ls "$ROOT/bootstrap/SKILL.md"     # sanity check before proceeding
```

If `CLAUDE_PLUGIN_ROOT` is unset, the user has cloned the repo and is running from inside
it, so `.` is correct. If neither resolves, ask the user where the template repo lives
rather than generating from memory.

## How to run

1. Read `$ROOT/bootstrap/SKILL.md` — the 5-phase process (Discovery → Requirements →
   Generation → Injection → Verification). It is the source of truth; follow it exactly.
2. Read `$ROOT/bootstrap/questions.md` — the question flow and auto-detection logic.
3. Generate from `$ROOT/bootstrap/templates/` — `skills/`, `agents/`, `commands/`,
   `claude-md/`, `hooks/`, `mcp/`, `settings/`, `gitignore.template`.

## Two rules that are easy to get wrong

- **Skills are folders.** Each generated skill goes to `.claude/skills/<name>/SKILL.md`.
  A flat `.claude/skills/<name>.md` is silently ignored by Claude Code — it never appears
  in the available-skills list, and agents naming it in `skills:` frontmatter point at
  nothing.
- **Substitute every `{{VARIABLE}}`.** An unsubstituted placeholder ships literal braces
  into the user's CLAUDE.md. See the Variable Substitution table in
  `$ROOT/bootstrap/SKILL.md`.

Both are re-checked in Phase 5 (Verification).
