# Automated Security Gates

> CI/CD security checks specifically designed to catch vulnerabilities in AI-generated code — because instruction files are advisory but pipelines are enforceable.

---

## Why AI Code Needs Automated Gates

AI-generated code introduces unique risks that traditional code review alone can't reliably catch:

- **Volume**: AI produces code faster than humans can review it thoroughly
- **Plausibility**: AI-generated vulnerabilities look professional and pass casual review
- **Pattern repetition**: If AI learns a bad pattern, it repeats it across many files
- **Hallucinated dependencies**: AI may suggest packages that don't exist or are compromised

Automated security gates catch these issues before they reach production — every time, without fatigue.

```
Developer → AI generates code → Commit → Push
                                          ↓
                              ┌─────────────────────┐
                              │   SECURITY PIPELINE  │
                              ├─────────────────────┤
                              │ Gate 1: Secrets      │ → Block if found
                              │ Gate 2: SAST         │ → Block on high/critical
                              │ Gate 3: Dep Audit    │ → Block on known CVEs
                              │ Gate 4: Licenses     │ → Block on violations
                              │ Gate 5: DAST         │ → Block on findings
                              │ Gate 6: Human Review │ → Require approval
                              └─────────────────────┘
                                          ↓
                                    Merge / Deploy
```

---

## Gate 1: Secret Scanning

**Purpose**: Detect accidentally committed credentials, API keys, and tokens.

**Why it matters for AI**: AI frequently generates realistic-looking placeholder secrets or copies secrets from context windows into code.

### Tools

| Tool | Type | Notes |
|------|------|-------|
| **GitHub Secret Scanning** | SaaS (built-in) | Automatic on GitHub repos; supports partner patterns |
| **gitleaks** | Open source | Pre-commit and CI; highly configurable |
| **detect-secrets** | Open source (Yelp) | Baseline-aware; good for legacy repos |
| **git-secrets** | Open source (AWS) | Prevents commits with AWS-pattern secrets |

### gitleaks Configuration

```toml
# .gitleaks.toml
title = "AI-Aware Secret Detection"

[allowlist]
description = "Global allowlist"
paths = [
  '''\.gitleaks\.toml$''',
  '''\.env\.example$''',
  '''docs/.*\.md$''',
]

# Custom rule: catch AI-generated placeholder keys
[[rules]]
id = "ai-placeholder-key"
description = "AI-generated placeholder API key"
regex = '''(?i)(api[_-]?key|secret|password|token)\s*[:=]\s*["']?[a-z0-9]{20,}["']?'''
tags = ["ai", "placeholder"]

[[rules]]
id = "hardcoded-password"
description = "Hardcoded password assignment"
regex = '''(?i)(password|passwd|pwd)\s*[:=]\s*["'][^"']{3,}["']'''
tags = ["password"]
```

### Pre-commit Hook

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

---

## Gate 2: Static Application Security Testing (SAST)

**Purpose**: Scan source code for vulnerability patterns — SQL injection, XSS, insecure deserialization, etc.

**Why it matters for AI**: AI frequently generates code with injection vulnerabilities, insecure defaults, and missing input validation.

### Tools

| Tool | Type | Best For |
|------|------|----------|
| **CodeQL** | Open source (GitHub) | Deep semantic analysis; GitHub-native |
| **Semgrep** | Open source | Fast pattern matching; custom rules |
| **Bandit** | Open source | Python-specific security |
| **ESLint security plugins** | Open source | JavaScript/TypeScript |

### Semgrep Configuration for AI-Generated Code

```yaml
# .semgrep.yml
rules:
  - id: ai-sql-injection
    patterns:
      - pattern: |
          $QUERY = "..." + $INPUT + "..."
      - pattern-not: |
          $QUERY = "..." + str($INT) + "..."
    message: >
      Possible SQL injection via string concatenation.
      AI often generates this pattern. Use parameterized queries instead.
    severity: ERROR
    languages: [python, javascript, typescript, java]

  - id: ai-eval-usage
    pattern: eval($INPUT)
    message: >
      eval() with dynamic input is dangerous.
      AI sometimes suggests eval for flexibility — use a safer alternative.
    severity: ERROR
    languages: [python, javascript]

  - id: ai-hardcoded-secret
    pattern: |
      $VAR = "sk-..."
    message: >
      Possible hardcoded API key. AI may copy key patterns from training data.
      Use environment variables instead.
    severity: ERROR
    languages: [python, javascript, typescript, ruby, java, go]

  - id: ai-disabled-tls
    patterns:
      - pattern: verify=False
      - pattern: rejectUnauthorized: false
      - pattern: InsecureSkipVerify: true
    message: >
      TLS verification disabled. AI suggests this to "fix" SSL errors.
      Fix the certificate issue instead.
    severity: ERROR
    languages: [python, javascript, typescript, go]
```

---

## Gate 3: Dependency Audit

**Purpose**: Check AI-suggested packages for known vulnerabilities and supply chain risks.

**Why it matters for AI**: AI may suggest outdated, deprecated, or even non-existent packages. It can hallucinate package names that could be typosquatted.

### Tools

| Tool | Ecosystem | Notes |
|------|-----------|-------|
| **npm audit** | Node.js | Built-in; checks npm advisory database |
| **pip-audit** | Python | Google-maintained; checks OSV database |
| **Snyk** | Multi-language | Commercial; deep analysis with fix suggestions |
| **Socket.dev** | npm/PyPI | Detects supply chain attacks, not just CVEs |
| **Dependabot** | Multi-language | GitHub-native; auto-creates fix PRs |

### Dependency Check Commands

```bash
# Node.js
npm audit --audit-level=high

# Python
pip-audit --strict --desc

# Go
govulncheck ./...

# Ruby
bundle audit check --update

# .NET
dotnet list package --vulnerable --include-transitive
```

### Verifying AI-Suggested Packages

```bash
# Before installing an AI-suggested package, verify it exists and is legit
# Check: does it exist? Who maintains it? How many downloads?

# npm
npm view <package-name> description maintainers time.modified

# PyPI
pip index versions <package-name>
```

---

## Gate 4: License Compliance

**Purpose**: Ensure AI-suggested dependencies don't violate your project's license requirements.

**Why it matters for AI**: AI doesn't consider license compatibility. It may suggest GPL-licensed packages in proprietary projects or AGPL packages in SaaS products.

### Tools

| Tool | Type | Notes |
|------|------|-------|
| **FOSSA** | Commercial | Deep license analysis, policy enforcement |
| **license-checker** | Open source | Node.js; quick local checks |
| **pip-licenses** | Open source | Python; simple license listing |
| **go-licenses** | Open source | Go; identifies all dependency licenses |

### Quick License Check

```bash
# Node.js — check for problematic licenses
npx license-checker --failOn "GPL-2.0;GPL-3.0;AGPL-3.0"

# Python
pip-licenses --fail-on="GPL-2.0;GPL-3.0;AGPL-3.0"
```

---

## Gate 5: Dynamic Application Security Testing (DAST)

**Purpose**: Test the running application for vulnerabilities that static analysis can't catch.

**Why it matters for AI**: AI-generated API endpoints may have authorization bypasses, IDOR vulnerabilities, or CORS misconfigurations that only appear at runtime.

### Tools

| Tool | Type | Notes |
|------|------|-------|
| **OWASP ZAP** | Open source | Full-featured DAST scanner |
| **Nuclei** | Open source | Template-based; fast and extensible |
| **Burp Suite** | Commercial | Industry standard for web app testing |

```yaml
# Example: OWASP ZAP in CI (simplified)
- name: DAST Scan
  uses: zaproxy/action-full-scan@v0.10.0
  with:
    target: 'http://localhost:3000'
    rules_file_name: '.zap/rules.tsv'
    fail_action: 'warn'
```

---

## Gate 6: Human Code Review Requirement

**Purpose**: Ensure no AI-generated code merges without human review and approval.

**Why it matters for AI**: Automated agents (Copilot coding agent, Claude Code) can create PRs autonomously. Without branch protection, they could merge without review.

### GitHub Branch Protection

```json
// Settings → Branches → Branch protection rules
{
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true
  },
  "required_status_checks": {
    "strict": true,
    "contexts": [
      "security/secrets",
      "security/sast",
      "security/dependencies",
      "security/licenses"
    ]
  },
  "enforce_admins": true,
  "allow_force_pushes": false,
  "allow_deletions": false
}
```

### CODEOWNERS for Security-Sensitive Files

```gitignore
# .github/CODEOWNERS
# Security-critical paths require security team review

# Authentication & authorization
src/auth/           @security-team
src/middleware/auth* @security-team

# Cryptography
src/crypto/         @security-team
src/utils/hash*     @security-team

# Infrastructure & deployment
.github/workflows/  @security-team @devops
Dockerfile          @security-team @devops
docker-compose*     @security-team @devops

# Security configuration
.semgrep.yml        @security-team
.gitleaks.toml      @security-team
```

---

## Complete GitHub Actions Security Workflow

This workflow implements all six gates in a single pipeline:

```yaml
# .github/workflows/security-gates.yml
name: Security Gates

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

permissions:
  contents: read
  security-events: write
  pull-requests: read

jobs:
  # Gate 1: Secret Scanning
  secrets:
    name: "Gate 1: Secret Scanning"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  # Gate 2: SAST
  sast:
    name: "Gate 2: Static Analysis"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Semgrep
        uses: semgrep/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/owasp-top-ten
            p/secrets
            .semgrep.yml
        env:
          SEMGREP_APP_TOKEN: ${{ secrets.SEMGREP_APP_TOKEN }}

  # Gate 2b: CodeQL (deeper analysis)
  codeql:
    name: "Gate 2b: CodeQL Analysis"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript,python  # adjust for your stack

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3

  # Gate 3: Dependency Audit
  dependencies:
    name: "Gate 3: Dependency Audit"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Node.js dependencies
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: npm audit
        run: npm audit --audit-level=high

      # Python dependencies (if applicable)
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: pip-audit
        run: |
          pip install pip-audit
          pip-audit --strict --desc -r requirements.txt
        if: hashFiles('requirements.txt') != ''

  # Gate 4: License Compliance
  licenses:
    name: "Gate 4: License Compliance"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Check licenses
        run: |
          npx license-checker \
            --failOn "GPL-2.0-only;GPL-3.0-only;AGPL-3.0-only" \
            --excludePrivatePackages

  # Gate 5: DAST (on PRs to main only)
  dast:
    name: "Gate 5: Dynamic Testing"
    runs-on: ubuntu-latest
    if: github.base_ref == 'main'
    steps:
      - uses: actions/checkout@v4

      - name: Build and start application
        run: |
          npm ci
          npm run build
          npm start &
          sleep 10  # wait for server to start

      - name: OWASP ZAP Scan
        uses: zaproxy/action-baseline@v0.12.0
        with:
          target: 'http://localhost:3000'
          fail_action: 'warn'

  # Gate 6: Enforce human review (via branch protection — this job is a check gate)
  review-gate:
    name: "Gate 6: Review Required"
    runs-on: ubuntu-latest
    needs: [secrets, sast, codeql, dependencies, licenses]
    steps:
      - name: All security gates passed
        run: echo "✅ All automated security gates passed. Human review still required."
```

---

## Copilot Coding Agent Integration

When using GitHub's Copilot coding agent, the agent runs in a container and can execute setup steps before working. Use `copilot-setup-steps.yml` to pre-install security tools.

### copilot-setup-steps.yml with Security Tools

```yaml
# .github/copilot-setup-steps.yml
# Runs when Copilot coding agent starts working on an issue/PR
steps:
  - name: Install dependencies
    run: npm ci

  - name: Install security tools
    run: |
      # Secret scanner
      curl -sSfL https://github.com/gitleaks/gitleaks/releases/download/v8.18.0/gitleaks_8.18.0_linux_x64.tar.gz | tar xz
      mv gitleaks /usr/local/bin/

      # SAST scanner
      pip install semgrep

      # Dependency auditor
      pip install pip-audit

  - name: Configure pre-commit hooks
    run: |
      # Ensure the agent's commits go through secret scanning
      pip install pre-commit
      pre-commit install

  - name: Verify security tooling
    run: |
      gitleaks version
      semgrep --version
      echo "✅ Security tools installed and ready"
```

This ensures the Copilot coding agent has security scanning available in its environment, so any code it generates can be checked before committing.

---

## Tips

✅ **Do** run secret scanning on every commit — not just PRs
✅ **Do** configure gates to block on high/critical findings — warn on medium
✅ **Do** keep SAST rules updated as new AI vulnerability patterns emerge
✅ **Do** use `copilot-setup-steps.yml` to give AI agents access to security tools
✅ **Do** require human review for all AI-generated PRs via branch protection

❌ **Don't** set all gates to "warn only" — at least secret scanning and critical SAST should block
❌ **Don't** skip dependency auditing — AI hallucinated packages are a real supply chain risk
❌ **Don't** rely on a single tool — layer defenses (secrets + SAST + deps + review)
❌ **Don't** forget to test your pipeline — intentionally commit a test secret to verify detection
❌ **Don't** make security checks optional — they should be required status checks

---

## Next Steps

- [Security-First Instructions](./security-first-instructions.md) — instruction files that complement automated gates
- [Incident Response](./incident-response.md) — what to do when a gate fails to catch something
- [Security Templates](../11-practical/templates.md) — ready-to-use workflow files
- [Supply Chain Security](../08-supply-chain/) — deeper dive on dependency risks
