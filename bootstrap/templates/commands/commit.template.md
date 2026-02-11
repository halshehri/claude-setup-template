---
description: Create a well-formatted conventional git commit
---

# Commit: Create Git Commit

## Process

### 1. Check Current State
```bash
git status
git diff HEAD
```

### 2. Stage Files
Stage relevant files. Exclude:
- `.env` files
- `node_modules/`, `__pycache__/`, `.venv/`
- Build artifacts
- Secrets

### 3. Determine Commit Type
| Type | Use When |
|------|----------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code change that neither fixes nor adds |
| `docs` | Documentation only |
| `style` | Formatting, no code change |
| `test` | Adding/updating tests |
| `chore` | Maintenance tasks |

### 4. Create Commit Message
Format: `{type}({scope}): {description}`

Examples:
```
feat(api): add user authentication endpoint
fix(frontend): resolve date picker timezone issue
refactor(accounts): simplify transaction validation
```

### 5. Commit
```bash
git add {files}
git commit -m "{type}({scope}): {description}"
```

## Output
```
Committed: {type}({scope}): {description}
Files: {count} files changed
Branch: {branch-name}
```
