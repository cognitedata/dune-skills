---
name: dune-app-review
description: >-
  Run a full Dune app platform review against a React/TypeScript CDF codebase,
  following the cognitedata/dune-app-reviews scoring criteria. Produces three
  artifacts: review-files.md (per-file inventory), review-packages.md (dependency
  audit), and review-report.md (scored report with must/should/nice-fix items).
  Use when the user asks for a Dune app review, code quality audit, CDF platform
  review, or "run dune-review" on a codebase.
allowed-tools: Read, Glob, Grep, Shell, Write
---

# Dune App Review

Fetch the official review command and follow it exactly:

```bash
gh api repos/cognitedata/dune-app-reviews/contents/.claude/commands/dune-review.md \
  --jq '.content' | base64 -d
```

Adapt it for a **local developer review**:
- Treat the **current workspace** as the app under review.
- Skip all ticket, PR, overview, submodule, and `reviews/<TICKET-ID>/...` setup steps.
- If the upstream command asks for Jira ticket or PR input, ignore that requirement and continue with the local codebase.
- Write the three artifacts to the **project root**: `review-files.md`, `review-packages.md`, and `review-report.md`.

After the review artifacts are written, fetch the official verification command and follow it too:

```bash
gh api repos/cognitedata/dune-app-reviews/contents/.claude/commands/dune-review-verify.md \
  --jq '.content' | base64 -d
```

Adapt verification the same way:
- Skip ticket and feedback-round lookup.
- Read the three root artifacts instead of `reviews/<TICKET-ID>/feedback-round-N/`.
- Verify the review against the local source code before declaring it complete.
