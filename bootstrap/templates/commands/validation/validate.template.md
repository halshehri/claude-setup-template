---
description: Run validation checks (tests, lint, build) for specified services
---

# Validate: Run Comprehensive Checks

## Arguments

`$ARGUMENTS` - Comma-separated service names or "all"

## Process

### 1. Determine Services
- If "all": Get list from root CLAUDE.md
- If specific: Use provided list

### 2. Run Checks Per Service

#### For Node.js/TypeScript Services
```bash
cd {service}
npm run lint
npm run build
npm test
```

#### For Python Services
```bash
cd {service}
ruff check .
pytest
```

## Output Report

```markdown
## Validation Report

### Services Validated
| Service | Lint | Build | Tests | Status |
|---------|------|-------|-------|--------|
| {name} | PASS | PASS | PASS | PASS |

### Failures
#### {service} - {check}
```
{error output}
```
**Suggested Fix**: {suggestion}

### Summary
- Total: {n} services
- Passed: {n}
- Failed: {n}

### Next Steps
{If all pass}: Ready for `/validation:code-review` then `/commit`
{If failures}: Fix issues and re-run
```
