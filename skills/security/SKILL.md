---
name: security
description: "MUST be used whenever reviewing a Dune app for security issues, or before shipping any feature that handles credentials, user input, or external data. Do NOT skip this when the user asks for a security review, security audit, or vulnerability check — run every step in order. Triggers: security, security review, security audit, vulnerability, XSS, injection, credentials, secrets, auth, authentication, authorization, token, sensitive data, input validation, CORS, CSP, dependency audit."
allowed-tools: Read, Glob, Grep, Shell, Write
metadata:
  argument-hint: "[file or directory to audit, or leave blank to audit the whole app]"
---

# Security Audit

Perform a thorough security review of **$ARGUMENTS** (or the whole app if no argument is given). Work through every step below in order and report findings with file paths and line numbers.

---

## Step 1 — Map the attack surface

Read these files before checking anything:

- `src/main.tsx` / `src/App.tsx` — entry point, routing, auth gating
- `vite.config.ts` — dev server proxy, CORS, headers
- `package.json` — list of third-party dependencies
- Any file matching `**/auth*`, `**/login*`, `**/token*`, `**/credential*`

Identify:
- All pages/routes and whether each is behind an auth guard
- All places where external data enters the app (CDF SDK calls, `fetch`, user form input)
- All places where data is written back (CDF upsert, `fetch` POST/PUT/DELETE)

---

## Step 2 — Credential & secret hygiene

Search for hard-coded credentials and sensitive values:

```bash
# Look for anything that smells like a secret in source files
grep -rn --include="*.ts" --include="*.tsx" --include="*.js" \
  -E "(password|secret|apikey|api_key|token|bearer|private_key)\s*=\s*['\"]" src/
```

Flag any match. Secrets must come from environment variables (`import.meta.env.VITE_*`) or from the Dune auth flow — never hard-coded.

Also verify:
- `.env.example` does not contain real secrets (only placeholder values like `your-token-here`)
- `.gitignore` lists `.env` and `.env.local`
- No `console.log`, `console.error`, or similar calls that print a CDF token, user object, or API key

---

## Step 3 — Dangerous DOM APIs

Search for patterns that allow arbitrary script execution or HTML injection:

```bash
grep -rn --include="*.tsx" --include="*.ts" \
  -E "dangerouslySetInnerHTML|innerHTML\s*=|eval\(|new Function\(|setTimeout\(['\"]|setInterval\(['\"]" src/
```

For each hit:
- `dangerouslySetInnerHTML`: confirm the value is sanitized with DOMPurify or equivalent before use. If not, flag as **HIGH**.
- `eval` / `new Function`: flag as **HIGH** unconditionally — there is no safe use in a browser app.
- `setTimeout`/`setInterval` with a string argument: flag as **MEDIUM** (equivalent to `eval`).

---

## Step 4 — Authentication & authorization

Read the auth setup (likely `src/contexts/`, `src/hooks/`, or `setup-dune-auth` output):

- Every route that shows CDF data must be behind the Dune auth guard (`useCogniteClient` returns a non-null `sdk` before rendering).
- The CDF client must be initialized with short-lived OIDC tokens, not a static API key.
- User role/capability checks must happen server-side (CDF ACLs) — do not rely solely on hiding UI elements.

Check the `useAtlasChat` / Atlas agent integration:
- The `agentExternalId` must not be constructed from user-supplied input.
- Tool `execute` functions must not trust `args` blindly — validate or guard before using values in CDF queries.

---

## Step 5 — Input validation

Every value that comes from a form, URL param, or query string before it reaches a CDF call or is rendered to the DOM must be validated:

```bash
# Find useSearchParams, URLSearchParams, and form onChange handlers
grep -rn --include="*.tsx" --include="*.ts" \
  -E "useSearchParams|URLSearchParams|searchParams\.get|e\.target\.value" src/
```

For each hit, verify:
- The value is validated with Zod or a type guard before use.
- String values rendered in JSX are not concatenated into raw HTML.

---

## Step 6 — Vite / server configuration

Read `vite.config.ts` and any `server.ts` / `express.ts` files:

- Confirm `server.headers` includes at minimum:
  - `Content-Security-Policy` — restricts script sources
  - `X-Frame-Options: DENY` or `frame-ancestors 'none'`
  - `X-Content-Type-Options: nosniff`
- Confirm the dev proxy (`server.proxy`) does not expose internal endpoints in production builds.
- Confirm `define` does not embed raw secrets into the bundle (use `import.meta.env` instead).

---

## Step 7 — Dependency audit

```bash
pnpm audit --audit-level=high
```

List every high/critical vulnerability with its package name, severity, and the recommended fix. If no vulnerabilities are found at high/critical level, state that explicitly.

---

## Step 8 — Report findings

Produce a structured report grouped by severity:

| Severity | File | Line | Issue | Recommendation |
|----------|------|------|-------|----------------|
| HIGH | `src/...` | 42 | `eval()` call | Remove; use a data-driven approach |
| MEDIUM | ... | ... | ... | ... |
| LOW | ... | ... | ... | ... |
| INFO | ... | — | Dependency X has a known low-severity CVE | Run `pnpm update X` |

If no issues are found in a step, state "No issues found" for that step. Do not skip steps silently.

---

## Done

Summarize the total number of findings by severity and list any items that require immediate action before the next deployment.
