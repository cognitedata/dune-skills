---
name: dependencies-audit
description: "MUST be used whenever auditing dependencies for a Dune app, checking for vulnerabilities, outdated packages, or supply-chain risks. Do NOT skip any step — run the full audit when the user asks for a dependency review, package audit, security scan, CVE check, or supply-chain review. Triggers: dependencies, packages, dependency audit, package audit, npm audit, pnpm audit, CVE, vulnerability, outdated, deprecated, supply chain, license, bundle size, node_modules."
allowed-tools: Read, Glob, Grep, Shell, Write
metadata:
  argument-hint: "[path to package.json, or leave blank to audit the root package.json]"
---

# Dependencies Audit

Audit all dependencies in **$ARGUMENTS** (or the root `package.json` if no argument is given) for health, security, and maintainability. This skill produces the `review-packages.md` artifact required by the Dune app review process.

---

## Step 1 — Read and list all dependencies

```bash
# List all dependencies and devDependencies
node -e "
  const pkg = require('./package.json');
  console.log('=== Dependencies ===');
  Object.entries(pkg.dependencies || {}).forEach(([name, ver]) => console.log(name + ' @ ' + ver));
  console.log('\\n=== Dev Dependencies ===');
  Object.entries(pkg.devDependencies || {}).forEach(([name, ver]) => console.log(name + ' @ ' + ver));
"
```

Record the total count of dependencies and devDependencies.

---

## Step 2 — Look up npm metadata for each package

For each package, gather:
- **Latest version** on npm
- **Weekly downloads**
- **Last publish date**
- **Deprecated** flag

```bash
# Batch lookup — run for each package (example for a single package)
npm view <package-name> --json 2>/dev/null | node -e "
  const data = JSON.parse(require('fs').readFileSync('/dev/stdin','utf8'));
  console.log(JSON.stringify({
    name: data.name,
    latest: data['dist-tags']?.latest,
    modified: data.time?.modified,
    deprecated: data.deprecated || false,
  }));
"

# For weekly downloads, use the npm API
curl -s "https://api.npmjs.org/downloads/point/last-week/<package-name>" | node -e "
  const data = JSON.parse(require('fs').readFileSync('/dev/stdin','utf8'));
  console.log(data.downloads);
"
```

For efficiency, batch multiple lookups. If the project has many dependencies, use a script:

```bash
node -e "
  const { execSync } = require('child_process');
  const pkg = require('./package.json');
  const allDeps = { ...pkg.dependencies, ...pkg.devDependencies };

  for (const [name, usedVersion] of Object.entries(allDeps)) {
    try {
      const info = JSON.parse(execSync('npm view ' + name + ' --json 2>/dev/null', { encoding: 'utf8' }));
      const latest = info['dist-tags']?.latest || 'unknown';
      const modified = info.time?.modified || 'unknown';
      const deprecated = info.deprecated ? 'YES' : 'No';
      console.log([name, usedVersion, latest, modified, deprecated].join(' | '));
    } catch {
      console.log(name + ' | ' + usedVersion + ' | LOOKUP FAILED');
    }
  }
"
```

---

## Step 3 — Run security audit

```bash
# Run audit with the project's package manager
pnpm audit --json 2>/dev/null || npm audit --json 2>/dev/null

# Also run production-only audit (what ships to users)
pnpm audit --prod --json 2>/dev/null || npm audit --production --json 2>/dev/null
```

Parse the JSON output for:
- Severity counts (critical, high, moderate, low)
- Per-vulnerability details (package, severity, title, patched version, advisory URL)

Any package with a known CVE is an automatic **Fail** in the health column.

---

## Step 4 — Assign health scores

For each package, assign a health indicator:

| Health | Criteria |
|--------|----------|
| **Pass** | >100k weekly downloads AND updated within last 12 months AND not deprecated AND version is current or near-current (within 1 major) |
| **Warn** | 10k–100k weekly downloads OR >12 months since last publish OR >1 major version behind |
| **Fail** | <10k weekly downloads OR no update in 2+ years OR deprecated OR known CVE |

Edge cases:
- `@cognite/*` packages: trust Cognite-internal packages even if download counts are low
- `@types/*` packages: trust DefinitelyTyped packages; focus on whether the version matches the main package
- Newly published packages (<6 months old): flag as **Warn** for review, not auto-Fail on low downloads

---

## Step 5 — Check for supply-chain risks

```bash
# Check for install scripts (preinstall, postinstall, prepare)
node -e "
  const { execSync } = require('child_process');
  const pkg = require('./package.json');
  const allDeps = Object.keys({ ...pkg.dependencies, ...pkg.devDependencies });

  for (const name of allDeps) {
    try {
      const info = JSON.parse(execSync('npm view ' + name + ' --json 2>/dev/null', { encoding: 'utf8' }));
      const scripts = info.scripts || {};
      const risky = ['preinstall', 'install', 'postinstall'].filter(s => scripts[s]);
      if (risky.length > 0) {
        console.log('INSTALL SCRIPT: ' + name + ' — ' + risky.join(', '));
      }
    } catch {}
  }
"

# Check for packages with very few maintainers (single point of failure)
# This is informational, not blocking
```

Flag any dependency with install scripts for manual review. Install scripts can execute arbitrary code during `npm install`.

---

## Step 6 — Check license compatibility

```bash
# List all licenses
npx license-checker --summary 2>/dev/null || node -e "
  const { execSync } = require('child_process');
  const pkg = require('./package.json');
  const allDeps = Object.keys({ ...pkg.dependencies, ...pkg.devDependencies });

  for (const name of allDeps) {
    try {
      const info = JSON.parse(execSync('npm view ' + name + ' --json 2>/dev/null', { encoding: 'utf8' }));
      console.log(name + ': ' + (info.license || 'UNKNOWN'));
    } catch {}
  }
"
```

Acceptable licenses for Dune apps (commercial distribution):
- MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC, 0BSD, Unlicense, CC0-1.0

Licenses that need legal review:
- GPL-2.0, GPL-3.0, LGPL-2.1, LGPL-3.0, AGPL-3.0, MPL-2.0, EUPL-1.1
- Any "UNKNOWN" or missing license

Flag any copyleft or unknown license as **Warn** or **Fail** depending on the license type.

---

## Step 7 — Generate the review-packages.md artifact

Produce the output in the format required by the Dune app review process:

```markdown
## Package audit: [app name]

### Dependencies

| Package | Used version | Latest | Weekly downloads | Last published | Deprecated | CVEs | Health |
| ------- | ------------ | ------ | ---------------- | -------------- | ---------- | ---- | ------ |
| react | ^18.2.0 | 18.3.1 | 25M | 2024-04-26 | No | 0 | Pass |
| some-old-lib | ^1.0.0 | 1.0.3 | 5k | 2021-03-15 | No | 0 | Fail |

### Dev Dependencies

| Package | Used version | Latest | Weekly downloads | Last published | Deprecated | CVEs | Health |
| ------- | ------------ | ------ | ---------------- | -------------- | ---------- | ---- | ------ |
| vitest | ^1.6.0 | 2.0.1 | 8M | 2024-07-01 | No | 0 | Pass |

### Security audit

| Severity | Count |
| -------- | ----- |
| Critical | 0 |
| High | 0 |
| Moderate | 0 |
| Low | 0 |

#### Vulnerabilities

| Package | Severity | Title | Patched in | Advisory |
| ------- | -------- | ----- | ---------- | -------- |
| (none found) | — | — | — | — |

### License summary

| License | Count | Packages |
| ------- | ----- | -------- |
| MIT | 45 | react, react-dom, ... |
| Apache-2.0 | 3 | ... |

### Supply-chain flags

| Package | Risk | Details |
| ------- | ---- | ------- |
| (none found) | — | — |
```

---

## Step 8 — Report summary

Summarize findings:

| Category | Count |
|----------|-------|
| Total dependencies | N |
| Total devDependencies | N |
| Health: Pass | N |
| Health: Warn | N |
| Health: Fail | N |
| CVEs (critical + high) | N |
| CVEs (moderate + low) | N |
| License concerns | N |
| Install script flags | N |

List **must fix** items first:
- Any dependency with a critical or high CVE
- Any deprecated dependency in production (not devDependencies)
- Any dependency with a copyleft license that hasn't been approved

Then **should fix**:
- Outdated packages (>1 major version behind)
- Packages with <10k weekly downloads
- Packages not updated in >2 years

Then **nice to fix**:
- Minor version bumps
- Switching to lighter alternatives

---

## Done

State the overall health verdict: how many Pass/Warn/Fail, whether there are blocking CVEs, and the recommended next steps for the app author.
