---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing — before committing, creating PRs, or reporting status. Requires running verification commands and reading their output before making any success claim. Evidence before assertions, always.
allowed-tools: Read, Bash, Grep
---

# Verification Before Completion

<!-- Adapted from obra/superpowers (MIT License, Copyright (c) 2025 Jesse Vincent) -->

**Core principle:** Evidence before claims, always.

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes. A previous run, "should pass", or confidence is not evidence.

## The Gate

Before claiming any status:

1. **IDENTIFY**: What command proves this claim?
2. **RUN**: Execute the FULL command — fresh, complete, now
3. **READ**: Full output, exit code, failure count
4. **VERIFY**: Does the output confirm the claim?
   - If NO: state the actual status with the evidence
   - If YES: state the claim WITH the evidence
5. **ONLY THEN**: make the claim

## What Each Claim Requires

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check |
| Build succeeds | Build command: exit 0 | Linter passing |
| Bug fixed | Original symptom re-tested: passes | Code changed, assumed fixed |
| Regression test works | Red-green verified (fails without fix, passes with) | Test passing once |
| Subagent completed | Diff inspected, changes verified | Agent reporting "success" |
| Requirements met | Line-by-line checklist against the plan | Tests passing |

## Red Flags — Stop and Verify

- "should", "probably", "seems to" in a status statement
- Expressing satisfaction before verification ("Great!", "Done!")
- About to commit, push, or PR without a fresh validation run
- Trusting a subagent's success report without checking the diff
- "Just this once" / tired and wanting the work over

## Why This Matters for Unattended Runs

In an autonomous loop there is no human to catch a false "done". The
execution report's `Validation Results` section must contain the actual
output of fresh command runs — it is the only witness. A loop that trusts
unverified success claims compounds errors run after run.
