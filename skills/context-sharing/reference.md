# Reproducibility standards

Use these standards when generating commit context windows.

## Evidence hierarchy

Prefer evidence in this order:

1. Repository state (`git status`, `git diff`, `git log`)
2. Executed validation commands and outputs
3. User instructions in current session
4. Agent memory of prior steps

If two sources conflict, choose the higher-priority source and note the conflict briefly.

## Decision recording standard

Each non-trivial decision should have:

- Decision statement
- Alternatives considered
- Constraint that drove the choice
- Revisit trigger

Avoid "chose X because better"; provide a concrete constraint (time, compatibility, risk, performance, maintainability).

## Validation standard

Separate:

- **Verified:** checks that were actually run
- **Assumed:** checks not run but expected to pass

Never present assumptions as verified outcomes.

## Reproduction standard

Reproduction steps should:

- Be ordered and minimal
- Use commands that work from repo root
- Include setup commands if required
- Include expected observable outcome at least once

## Handoff standard

A collaborator should be able to continue work without chat history by reading:

- Objective
- Decisions and rationale
- Risks
- Open questions
- Next suggested actions
