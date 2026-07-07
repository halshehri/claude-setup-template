---
description: Behavior-preserving refactor of a targeted area, run after development
argument-hint: "<file|directory|service>"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Refactor: Clean a Targeted Area

Improve the internal structure of `$ARGUMENTS` without changing what it does.
Run this after development is complete and validated — never mid-feature.

## Arguments

`$ARGUMENTS` - Target file, directory, or service (required)

## The Iron Rule

**Preserve behavior exactly.** No functional changes, no API changes visible
outside the target, no "while I'm here" fixes. If something looks like a bug,
it goes on the Review List — fixing it here would hide a behavior change
inside a refactor commit.

## Step 0: Green Baseline (gate)

1. Run the target's validation commands (tests, lint, build).
2. **If validation fails: STOP.** Report the failures and exit. Refactoring on
   a red baseline makes behavior preservation unprovable.
3. Note which parts of the target have test coverage. Uncovered code gets only
   provably-safe transforms (Pass 1 deletions confirmed by search, renames of
   symbols whose call sites are all inside the target); everything riskier is
   flagged, not changed.

## Refactoring Passes (in this order)

Run the passes in order — deletion first shrinks everything later passes must
read. After each pass: run validation, then commit (`refactor: pass N — {summary}`).
One pass per commit keeps the history bisectable and each step revertable.

### Pass 1: Delete
- Unused functions, methods, classes (verify zero references with Grep across
  the whole repo, not just the target)
- Unreachable branches, unused imports, commented-out code blocks
- If you cannot *prove* something is dead (dynamic dispatch, reflection,
  external callers, config-driven entry points) → Review List, don't delete.

### Pass 2: Consolidate
- Duplicate and near-duplicate logic → one shared helper, callers updated
- Place helpers at the lowest shared level; don't create a `utils` dumping ground
- Near-duplicates that differ in subtle ways worth keeping → Review List with
  the difference named

### Pass 3: Restructure
- Reduce nesting: early returns, guard clauses, invert conditions
- Split god functions/classes: one responsibility per unit
- Separate concerns: I/O (network, disk, DB) apart from business logic so the
  logic is testable without mocks
- Organize: logical ordering within files, split oversized files, clear module
  boundaries — follow the codebase's existing conventions

### Pass 4: Clarify
- Rename for intention: `d` → `days_until_expiry`. Only rename symbols whose
  call sites all live inside the target; public/exported names → Review List
- Magic numbers/strings → named constants
- Simplify clever/dense code into explicit readable code (a one-liner that
  needs a comment to decode becomes three clear lines)
- Add type hints and docstrings — docstrings say *why*, not *what*; skip
  docstrings that restate the signature

## Flag, Don't Fix

Anything below goes on the Review List, never silently changed:
- Suspicious logic or a possible bug
- Dead code you can't prove dead
- Public API renames or signature changes that would improve clarity
- Behavior that looks unintended but is covered by a passing test

## Step Final: Verify

Apply `verification-before-completion`: re-run the FULL validation suite
fresh, read the output, and confirm the diff contains no behavior change
(no test files weakened, no assertions removed, no config values altered).

## Output Report

```markdown
## Refactor Report: {target}

### Baseline
- Validation before: PASS ({command output summary})
- Coverage gaps: {areas limited to safe transforms}

### Changes by Pass
| Pass | Commits | Summary |
|------|---------|---------|
| 1 Delete | {sha} | {n} dead functions, {n} unused imports removed |
| 2 Consolidate | {sha} | {duplicates merged into which helpers} |
| 3 Restructure | {sha} | {functions split, nesting reduced where} |
| 4 Clarify | {sha} | {renames, constants, types/docs added} |

### Metrics
- Lines: {before} → {after}
- Files: {before} → {after}

### Review List (flagged, NOT changed)
- [ ] `{file}:{line}` — {suspicious logic / uncertain dead code / public rename candidate}: {why flagged}

### Validation After
{fresh full-suite output — actual run, not recalled}
```
