---
name: conventional-commits
description: "Writes and stages git commits using the Conventional Commits 1.0.0 format so PR history is scannable for humans and agents. Use when creating commits, amending messages, splitting commits, preparing a PR, or when the user asks for conventional commits, commit messages, or git hygiene on a Dune app. Triggers: commit, commit message, conventional commit, git log, squash, rebase message, changelog-friendly commits."
allowed-tools: Read, Glob, Grep, Shell, Write
metadata:
  argument-hint: "[optional: scope or area to emphasize, e.g. 'atlas' or 'auth']"
---

# Conventional Commits (Dune apps)

Follow **[Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)** for every commit. The goal is **frequent, focused commits** with messages that are **specific enough to review in a PR diff**, but **free of noise** (no essays, no redundant file lists, no vague “updates”).

`$ARGUMENTS` may hint a preferred **scope** (e.g. `atlas`, `cdf`, `auth`); otherwise infer scope from the changed files.

---

## When to commit

- After each **coherent unit of work** that leaves the repo in a sensible state (build/lint still passes when that is the project norm).
- Prefer **several small commits** over one large “WIP” commit, so reviewers (product or agents) can read the PR as a story.
- If many unrelated edits accumulated, **split** into multiple commits (by path or by concern) before opening the PR.

---

## Message structure (spec)

```
<type>[optional scope][optional !]: <description>

[optional body]

[optional footer(s)]
```

- **One blank line** between subject and body.
- **Description** (subject line): concise summary of the change; do not end with a period.
- **Breaking changes**: append `!` before the colon (e.g. `feat(api)!: remove legacy query param`) and/or a footer line `BREAKING CHANGE: <explanation>` per the spec.

---

## Types

Use the **smallest accurate type**. Common choices:

| type | Use for |
|------|---------|
| `feat` | New user-facing capability or API |
| `fix` | Bug fix or incorrect behavior |
| `docs` | Documentation only |
| `style` | Formatting, whitespace, semicolons — no logic change |
| `refactor` | Internal change, same outward behavior |
| `perf` | Performance improvement |
| `test` | Adding or correcting tests |
| `build` | Tooling, bundler, deps that affect builds |
| `ci` | CI configuration |
| `chore` | Maintenance that does not fit above (e.g. ignore files) |

If unsure between `feat` and `fix`, ask whether behavior **changed for users** in an intended way (`feat`) vs corrected (`fix`).

---

## Scope (optional but helpful)

Use **short, lowercase** scopes when they clarify the PR:

- Examples: `atlas`, `auth`, `cdf`, `dm`, `ui`, `chat`, `viewer`, `hooks`, `deps`
- Omit scope rather than invent a vague one.

---

## Subject line (description after `:`)

- **Imperative mood**, as if completing: “this commit will…” → *Add*, *Fix*, *Refactor*, not *Added* / *Adds*.
- Aim for **~50 characters or less** for the subject when practical (readability in `git log --oneline`).
- **No trailing period**; capitalize only proper nouns if needed.

---

## Body (optional)

Add a body when the subject alone would hide **why** or **non-obvious “what”**:

- 1–6 short lines: motivation, tradeoff, edge case, or risk.
- Wrap near **72 characters** per line.
- Do **not** paste full file paths or duplicate the diff; reviewers already have the diff.
- Do **not** restate the subject in other words unless it adds information.

Skip the body entirely for trivial, self-explanatory changes.

---

## Footers

Use when applicable:

- `BREAKING CHANGE: …` for incompatible API or behavior (and use `!` in the header when appropriate).
- Issue references if the project uses them: `Fixes #123`, `Refs #456`.

---

## Workflow before writing the message

1. Run `git status` and `git diff` (and `git diff --staged` if relevant) to see **exactly** what changed.
2. Choose **type** and **scope** from real changes, not intent from memory.
3. If staged files mix unrelated concerns, **unstage** and commit in smaller slices.
4. Write subject first; add body only if it earns its keep.

---

## Examples

**Good**

```
feat(chat): stream tool results as they arrive

Atlas previously buffered the full tool payload. Stream partial
JSON so the UI can render progressive status without blocking.
```

```
fix(auth): refresh token before expiry on idle tab

CDF sessions were expiring after background tab throttling skipped
the refresh timer. Align refresh with visibility and token TTL.
```

```
chore(deps): bump @cognite/sdk to 9.x
```

**Avoid**

- `fix: bug fix` — not specific.
- `update files` — missing type; vague.
- `feat: added some stuff and cleaned up` — multiple unrelated ideas; weak subject.

---

## PR discipline

For PR titles/descriptions aimed at product + engineering: keep the **same vocabulary** (type/scope meaning) in the PR description’s summary bullets so `git log` and the PR stay aligned. Do not replace conventional commits with only a PR title; **commits** should still follow this format.
