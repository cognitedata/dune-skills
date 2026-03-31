# dune-app-review evaluation summary

## Method
- Manual schema check of `skills/dune-app-review/SKILL.md`
- Three qualitative prompts, each compared in two modes:
  - **with-skill**: subagent explicitly read the skill first
  - **baseline**: subagent answered without the skill
- Focus: whether the skill changes planning behavior in ways we care about

## Validation
- `name`: `dune-app-review`
- name format: valid kebab-case
- description length: 433 chars
- `allowed-tools`: `Read, Glob, Grep, Shell, Write`
- line count: 39
- note: bundled `quick_validate.py` could not run in this environment because `PyYAML` was not installed

## Test prompts
1. `Run a Dune app review on this codebase before I submit it. I don't have a Jira ticket or PR yet.`
2. `Can you do the official Cognite Dune app platform review on this app and save the results somewhere organized?`
3. `I already reviewed this Dune app once. Please run another review pass and keep the output organized so I can compare rounds.`

## Results

### Prompt 1
**With skill**
- fetched upstream `dune-review.md`
- explicitly ignored Jira/PR assumptions
- wrote artifacts under `reviews/dune-app-review/feedback-round-<N>/`
- fetched and ran `dune-review-verify.md`

**Baseline**
- did not fetch upstream
- invented a generic local workflow
- chose project root or `docs/reviews/` as output location
- included verification only as a recommendation

**Verdict**: strong win for the skill

### Prompt 2
**With skill**
- fetched `dune-review.md` and `dune-review-verify.md`
- followed official workflow with local adaptations
- used `reviews/dune-app-review/feedback-round-<N>/`

**Baseline**
- also chose to fetch upstream because the prompt explicitly said `official`
- still chose root-level artifact files
- verification path was adapted less precisely

**Verdict**: moderate win for the skill

### Prompt 3
**With skill**
- preserved upstream review model
- incremented `feedback-round-<N>` for reruns
- kept filenames identical across rounds
- still required adapted verification for the current round

**Baseline**
- chose a generic per-round folder convention like `review-round-02/`
- kept reports comparable, but drifted from upstream structure
- verification was described generically rather than tied to upstream verify flow

**Verdict**: strong win for the skill

## Overall assessment
The skill is doing useful work. Its main value is not inventing review content; its value is **constraining the agent to the official upstream workflow while correctly adapting it for a local developer review**.

Most important observed benefits:
- preserves upstream `dune-review` / `dune-review-verify` as the source of truth
- removes Jira/PR/submodule assumptions cleanly
- standardizes artifact location for local reviews
- preserves round-based organization for reruns
- makes verification part of the default path

## Improvement notes
No body changes feel mandatory after this pass.

One possible follow-up improvement: make the description slightly more trigger-friendly by adding phrases like `before submission`, `pre-submit review`, `approval review`, or `app certification`, since the description is the main trigger surface and current agents tend to under-trigger skills.

## Recommendation
Keep the current body.
Optional next iteration: tighten the description only, then run a description-trigger eval using `skill-creator/scripts/run_eval.py` in an environment with the required dependencies and API credentials.
