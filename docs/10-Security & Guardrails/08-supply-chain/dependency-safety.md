# AI-Suggested Dependency Safety

> When AI assistants recommend packages and libraries, they may suggest dependencies that are
> outdated, vulnerable, unmaintained, or even entirely fabricated. This guide covers how to
> verify every AI-suggested dependency before it enters your project, protecting your software
> supply chain from hallucinated packages, typosquatting attacks, and known vulnerabilities.

---

## Why This Matters

AI coding assistants are trained on vast corpora of code, documentation, and community
discussions. While this makes them powerful, it also means they can:

- **Hallucinate packages** that don't actually exist on any registry
- **Suggest outdated versions** with known security vulnerabilities
- **Confuse similar package names**, inadvertently pointing you to typosquatted malware
- **Recommend abandoned projects** that no longer receive security patches
- **Mix up ecosystems**, suggesting a Python package name when you need a Node.js one

A single unverified dependency can compromise your entire application. Supply chain attacks
through malicious packages have affected thousands of projects — from `event-stream` in npm
to `ctx` in PyPI. When an AI suggests a package, treat it like you would treat advice from
a colleague who hasn't checked the registry recently: **verify before you trust**.

---

## The Core Problem

### AI Models Don't Browse Package Registries in Real Time

Large language models generate suggestions based on patterns in their training data. They do
not query npm, PyPI, or any other registry at inference time. This creates several failure
modes:

| Failure Mode | Description | Risk Level |
|---|---|---|
| **Hallucinated package** | AI invents a plausible-sounding name that doesn't exist | 🔴 Critical |
| **Typosquatted name** | AI suggests `colurs` instead of `colors` | 🔴 Critical |
| **Deprecated package** | Package exists but is officially deprecated | 🟡 Moderate |
| **Unmaintained package** | No updates or security patches in 2+ years | 🟡 Moderate |
| **Vulnerable version** | Suggested version has known CVEs | 🔴 Critical |
| **Wrong ecosystem** | Suggests a Rust crate name for a Python project | 🟢 Low |

### The Typosquatting Threat

Typosquatting is when attackers publish malicious packages with names similar to popular
ones. They rely on developers making typos or — increasingly — on AI models suggesting the
wrong name. Real-world examples:

- `crossenv` (malicious) vs `cross-env` (legitimate) in npm
- `python3-dateutil` (malicious) vs `python-dateutil` (legitimate) in PyPI
- `jeIlyfish` (with a capital I) vs `jellyfish` (legitimate) in PyPI

When an AI suggests a package name character-by-character, even a single wrong letter can
route you to a malicious package that exfiltrates environment variables, credentials, or
source code.

---

## The Five-Point Verification Checklist

Before adding any AI-suggested dependency, walk through these five checks. Each one catches
a different category of risk.

### 1. Does the Package Actually Exist?

The most basic check: confirm the package is real and published on the expected registry.

**For npm packages:**

```bash
# Check if the package exists on the npm registry
npm view <package-name>

# Quick existence check — returns metadata or an error
npm view <package-name> name version description
```

**For Python packages:**

```bash
# Check PyPI for the package
pip index versions <package-name>

# Or search directly
pip install <package-name>==99.99.99  # Will show available versions in the error
```

**For Go modules:**

```bash
# Check if the module is accessible
go list -m <module-path>@latest
```

**Red flags:**

- The `npm view` or `pip index` command returns a 404 or "not found" error
- The package exists but has a completely different description than expected
- The package was published very recently (less than a week ago)

### 2. Is It Actively Maintained?

An unmaintained package won't receive security patches, leaving your application exposed
to future vulnerabilities.

**Check these signals:**

```bash
# npm: Check last publish date and release frequency
npm view <package-name> time --json | head -20

# npm: Check the repository link
npm view <package-name> repository.url
```

**On GitHub, check:**

- When was the last commit? (> 2 years ago is a red flag)
- Are there open security issues being ignored?
- Has the maintainer responded to any issues recently?
- Is there a `SECURITY.md` or security policy?

**Maintenance indicators:**

| Signal | Healthy | Concerning |
|---|---|---|
| Last commit | Within 6 months | Over 2 years ago |
| Open issues response | Maintainer responds | Hundreds ignored |
| Release frequency | Regular releases | No releases in 2+ years |
| CI/CD status | Passing builds | Broken or absent |
| Dependency updates | Kept current | Severely outdated |

### 3. Are There Known Vulnerabilities?

Even well-maintained packages can have unpatched vulnerabilities. Always check before
adding a new dependency.

**Using npm audit:**

```bash
# After adding the package to package.json
npm audit

# Check a specific package before installing
npm audit --package-lock-only
```

**Using pip-audit:**

```bash
# Install pip-audit
pip install pip-audit

# Audit current environment
pip-audit

# Audit a specific package
pip-audit --requirement requirements.txt
```

**Using the GitHub Advisory Database:**

```bash
# Search advisories via the GitHub CLI
gh api graphql -f query='
{
  securityVulnerabilities(first: 5, ecosystem: NPM, package: "<package-name>") {
    nodes {
      advisory { summary severity }
      vulnerableVersionRange
      firstPatchedVersion { identifier }
    }
  }
}'
```

**Or visit directly:**

- [GitHub Advisory Database](https://github.com/advisories)
- [OSV (Open Source Vulnerabilities)](https://osv.dev)
- [Snyk Vulnerability Database](https://security.snyk.io)

### 4. Is the Name Correct (Not Typosquatted)?

This is the check most people skip — and the one most relevant to AI-suggested packages.

**Verification steps:**

```bash
# Compare the AI-suggested name against the official docs
# Visit the package's official website or GitHub repository

# npm: Check who published it
npm view <package-name> maintainers

# npm: Check download counts (low downloads on a "popular" package = red flag)
npm view <package-name> --json | Select-String '"weekly"'
```

**Cross-reference these sources:**

1. The official project website or documentation
2. The GitHub/GitLab repository README
3. The registry page (npmjs.com, pypi.org)
4. Stack Overflow answers from verified/high-reputation users

**Common typosquatting patterns to watch for:**

- Swapped characters: `axois` instead of `axios`
- Extra characters: `expresss` instead of `express`
- Missing hyphens: `react router` instead of `react-router`
- Scope confusion: `@my-scope/lodash` instead of `lodash`
- Letter substitution: `1odash` (with numeral one) instead of `lodash`
- Homoglyphs: `rеact` (Cyrillic 'е') instead of `react` (Latin 'e')

### 5. Is It Popular and Trusted Enough?

Popularity alone doesn't guarantee safety, but it does mean more eyes on the code and
faster vulnerability detection.

**npm download statistics:**

```bash
# Check weekly downloads
npm view <package-name> --json | Select-String "downloads"

# Or visit: https://www.npmjs.com/package/<package-name>
```

**Trust signals to evaluate:**

| Signal | Trustworthy | Risky |
|---|---|---|
| Weekly downloads | 100,000+ | Under 100 |
| GitHub stars | 1,000+ | Under 10 |
| Contributors | Multiple | Single anonymous |
| Used by | Known organizations | No notable users |
| Published by | Verified publisher | Unverified account |
| Package age | Years old | Days or weeks old |
| License | Standard OSS license | No license or unusual |

---

## Verification Tools

### npm audit

Built into npm. Scans your `node_modules` against the GitHub Advisory Database.

```bash
# Run a full audit
npm audit

# Get JSON output for CI integration
npm audit --json

# Auto-fix where possible
npm audit fix

# See what would change without applying
npm audit fix --dry-run
```

### pip-audit

The Python equivalent, maintained by the Python Packaging Authority (PyPA).

```bash
# Install
pip install pip-audit

# Audit installed packages
pip-audit

# Audit from requirements file
pip-audit -r requirements.txt

# Output in JSON for CI integration
pip-audit --format json
```

### Snyk

Commercial tool with a generous free tier. Provides deeper analysis than built-in tools.

```bash
# Install Snyk CLI
npm install -g snyk

# Authenticate
snyk auth

# Test a project
snyk test

# Monitor for new vulnerabilities
snyk monitor
```

### Socket.dev

Specializes in detecting supply chain attacks. Analyzes package behavior rather than just
known vulnerabilities.

**What Socket detects:**

- Install scripts that run arbitrary code
- Network access during installation
- Filesystem access outside the package directory
- Obfuscated or minified source code in packages
- Protestware and other unexpected behaviors

Visit [socket.dev](https://socket.dev) to integrate with your GitHub repositories.

### GitHub Advisory Database

The open, community-curated database that powers `npm audit`, `pip-audit`, and GitHub's
own Dependabot.

```bash
# Use the GitHub CLI to query advisories
gh advisory list --ecosystem npm

# Search for a specific package
gh advisory list --ecosystem pip --keyword <package-name>
```

---

## Practical Verification Workflow

Use this step-by-step workflow every time an AI suggests a new dependency.

### Step 1: Capture the Suggestion

When the AI suggests a package, note the exact name and version:

```
AI suggested: "You can use the `fast-xml-parser` package (v4.3.2) for XML parsing."
```

### Step 2: Verify Existence and Identity

```bash
# Confirm it exists and check metadata
npm view fast-xml-parser name version description author license

# Check the repository URL — does it lead to a real project?
npm view fast-xml-parser repository.url
```

### Step 3: Check for Vulnerabilities

```bash
# Create a temporary test to avoid polluting your project
mkdir dep-check && cd dep-check
npm init -y
npm install fast-xml-parser
npm audit

# Clean up
cd ..
Remove-Item -Recurse -Force dep-check
```

### Step 4: Assess Maintenance and Popularity

```bash
# Check publish history
npm view fast-xml-parser time --json

# Check download stats on npmjs.com
# Visit: https://www.npmjs.com/package/fast-xml-parser

# Check GitHub activity
# Visit the repository URL from Step 2
```

### Step 5: Cross-Reference the Name

```bash
# Search for the package to check for typosquatting
npm search fast-xml-parser

# Verify on the official project website
# Google: "fast-xml-parser npm official"
```

### Step 6: Make a Decision

Based on your findings, decide:

- ✅ **Install** — Package is verified, maintained, popular, and vulnerability-free
- ⚠️ **Investigate further** — Something looks off; ask a teammate for a second opinion
- ❌ **Reject** — Package is hallucinated, typosquatted, unmaintained, or vulnerable

---

## Instruction File Rules for Dependencies

Set clear rules in your project's AI instruction files to prevent unsafe dependency
additions.

### `.github/copilot-instructions.md` — Dependency Policy

```markdown
## Dependency Management Rules

### Adding New Dependencies
- Only suggest well-known, actively maintained packages with 10,000+ weekly downloads
- Never suggest packages you are unsure exist — state uncertainty explicitly
- Always include the exact package name as listed on the official registry
- Prefer packages from verified publishers or known organizations
- Do not suggest packages that have been deprecated

### Version Pinning
- Always suggest exact versions, not ranges (e.g., "4.3.2", not "^4.3.2")
- Suggest the latest stable version, never pre-release or beta versions
- If you are unsure of the latest version, say so rather than guessing

### Before Suggesting a Package
- Verify the name matches the official registry listing exactly
- Note the ecosystem (npm, PyPI, crates.io) explicitly
- Mention if the package has any known alternatives that are more popular

### Approval Process
- All new dependency additions require human review before installation
- Flag any dependency suggestion with: "⚠️ New dependency — please verify before installing"
```

### `.github/copilot-instructions.md` — Restricted Packages

```markdown
## Restricted and Banned Packages

### Banned Packages (Never Suggest)
- `request` (deprecated — use `node-fetch` or `axios` instead)
- `moment` (deprecated — use `date-fns` or `dayjs` instead)
- `uuid` versions below 7.0.0 (known vulnerabilities)

### Preferred Alternatives
| Instead of | Use |
|---|---|
| `request` | `node-fetch` or `undici` |
| `moment` | `date-fns` or `dayjs` |
| `underscore` | `lodash` (or native methods) |
| `body-parser` | Built-in `express.json()` |
| `node-uuid` | `uuid` (official package) |

### Internal Packages
- Prefer `@our-org/http-client` over third-party HTTP libraries
- Use `@our-org/logger` instead of `winston` or `pino`
- Authentication must use `@our-org/auth-sdk` only
```

### `.github/copilot-instructions.md` — CI Enforcement

```markdown
## CI/CD Dependency Checks

All pull requests that modify dependency files are automatically scanned:

1. `npm audit` runs on every PR touching `package.json` or `package-lock.json`
2. `pip-audit` runs on every PR touching `requirements.txt` or `pyproject.toml`
3. Socket.dev GitHub App analyzes new dependencies for supply chain risks
4. Dependabot alerts must be resolved before merging

### Lock File Requirements
- Always commit lock files (`package-lock.json`, `poetry.lock`, etc.)
- Never suggest `--no-package-lock` or equivalent flags
- Lock files must be regenerated if dependency files change
```

---

## Tips

### ✅ Do

- ✅ **Always verify the exact package name** on the official registry before installing
- ✅ **Run `npm audit` or `pip-audit`** immediately after adding any new dependency
- ✅ **Cross-reference the AI suggestion** with the project's official documentation
- ✅ **Check the package's GitHub repository** for recent activity and open issues
- ✅ **Use exact version pinning** (`4.3.2`) rather than ranges (`^4.3.2`) for new deps
- ✅ **Require human approval** for all new dependency additions in your workflow
- ✅ **Configure Dependabot or Renovate** to keep existing dependencies updated
- ✅ **Use Socket.dev or similar tools** to detect supply chain attack patterns
- ✅ **Keep an approved-packages list** for your project that AI assistants can reference
- ✅ **Document why each dependency was chosen** in a comment or ADR (Architecture Decision Record)
- ✅ **Prefer packages with TypeScript types included** — they tend to be better maintained
- ✅ **Check if the functionality already exists** in your current dependencies or the standard library

### ❌ Don't

- ❌ **Never install a package without checking it exists** on the expected registry
- ❌ **Never trust AI-suggested version numbers** — they are frequently outdated or wrong
- ❌ **Never add a dependency in a rush** without running through the verification checklist
- ❌ **Never ignore low download counts** on a package the AI claims is "popular"
- ❌ **Never skip the typosquatting check** — verify every character of the package name
- ❌ **Never install packages with `--ignore-scripts`** disabled in production pipelines
- ❌ **Never add a dependency when native functionality would suffice** (e.g., `is-odd` package)
- ❌ **Never blindly accept `npm audit fix --force`** — review major version bumps manually
- ❌ **Never assume the AI knows the current package landscape** — models have training cutoffs
- ❌ **Never add both a package and its fork** without understanding why both exist

---

## Real-World Scenarios

### Scenario 1: The Hallucinated Package

```
AI: "Use `react-form-validator-pro` to validate your forms."

Developer checks: npm view react-form-validator-pro
Result: 404 Not Found

Action: Package doesn't exist. The AI hallucinated a plausible name.
         Ask the AI to suggest an alternative, then verify that one too.
         Real alternatives: `react-hook-form`, `formik`, `zod`
```

### Scenario 2: The Typosquatted Suggestion

```
AI: "Install `electorn` for your desktop app framework."

Developer checks: npm view electorn
Result: Package exists... but with 12 weekly downloads and a suspicious author.

Action: The correct package is `electron` (200M+ downloads).
         The AI swapped two letters. Reject and use the correct name.
```

### Scenario 3: The Deprecated Package

```
AI: "Use `request` to make HTTP calls from Node.js."

Developer checks: npm view request
Result: Package exists but shows "DEPRECATED" banner.

Action: The `request` package was deprecated in 2020.
         Use `node-fetch`, `axios`, or `undici` instead.
         Update your instruction file to ban `request`.
```

### Scenario 4: The Vulnerable Version

```
AI: "Add `lodash@4.17.15` to your project."

Developer checks: npm audit after installing
Result: Prototype Pollution vulnerability (CVE-2020-8203)

Action: The AI suggested an outdated version. The fix is in 4.17.21+.
         Always install the latest patch version: npm install lodash@latest
```

---

## Automating Verification in CI/CD

### GitHub Actions Workflow

```yaml
name: Dependency Safety Check

on:
  pull_request:
    paths:
      - 'package.json'
      - 'package-lock.json'
      - 'requirements.txt'
      - 'pyproject.toml'

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Node.js Audit
        if: hashFiles('package.json') != ''
        run: |
          npm ci
          npm audit --audit-level=high

      - name: Python Audit
        if: hashFiles('requirements.txt') != ''
        run: |
          pip install pip-audit
          pip-audit -r requirements.txt

      - name: Check for new dependencies
        run: |
          git diff origin/main -- package.json | grep '"dependencies"' -A 50 || true
          echo "⚠️ Review any new dependencies above before approving."
```

---

## Summary

AI coding assistants are powerful tools, but they operate without real-time knowledge of
package registries. Every dependency suggestion should be treated as unverified until you
have confirmed it through the five-point checklist:

1. **Existence** — The package is real and published
2. **Maintenance** — The package is actively maintained
3. **Security** — No known vulnerabilities in the suggested version
4. **Identity** — The name is correct, not a typosquatted variant
5. **Trust** — The package has sufficient community adoption

Build these checks into your workflow, encode them in your instruction files, and automate
them in your CI/CD pipeline. The few minutes spent verifying a dependency can save weeks
of incident response.
