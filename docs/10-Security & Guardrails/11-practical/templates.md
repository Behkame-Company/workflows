# Security Templates & Checklists

> Ready-to-use templates for securing AI-assisted development workflows. Copy-paste these into your projects to establish security guardrails from day one.

---

## Overview

This document provides **11 production-ready templates** that cover the full spectrum of AI coding security — from instruction files that shape AI behavior, to CI/CD workflows that catch mistakes, to incident response playbooks for when things go wrong.

Each template is complete and self-contained. Copy the code block, paste it into the specified file in your project, and customize the marked sections for your environment.

---

## Common Security Rules (Template Header)

Templates 1–3 below share a core set of security rules. Define these once in your organization and reference them across instruction files. The common rules are:

```markdown
## Core Security Rules

### Forbidden Actions
- NEVER hardcode passwords, API keys, tokens, or secrets in source code
- NEVER disable TLS/SSL verification, authentication, CSRF protection, or CORS policies
- NEVER use `eval()`, `exec()`, `Function()`, or dynamic code execution on user input
- NEVER read/write files outside the project directory or follow escaping symlinks
- NEVER make outbound requests to unapproved domains

### Required Practices
- Use parameterized queries — no string concatenation in SQL
- Validate and sanitize ALL user input at system boundaries
- Encode output for its context (HTML, SQL, shell, URL)
- Reference secrets via environment variables or a secrets manager
- Handle errors explicitly — never expose stack traces to end users
- Pin dependency versions — no ranges, no `latest`
- Run `npm audit` / `pip audit` before committing new dependencies

### Sensitive Paths — Never Read or Reference
`.env`, `.env.*`, `secrets/`, `private/`, `*.pem`, `*.key`, `*.p12`,
`~/.ssh/`, `~/.aws/`, `~/.config/gcloud/`, `credentials*`, `token*`
```

Adapt the wording to each file format below, but don't repeat the rules themselves.

---

## Template 1 — Security-Focused `copilot-instructions.md`

Place at `.github/copilot-instructions.md`. Applies to all GitHub Copilot interactions. Include the **Core Security Rules** above, plus these Copilot-specific additions:

```markdown
# Copilot Instructions — Security Policy

<!-- Include Core Security Rules from your organization's template -->

## Copilot-Specific Rules
- Use placeholder values like `<YOUR_API_KEY>` or `process.env.API_KEY` in examples
- Never return more data than the caller needs — avoid `SELECT *` in production code
- Set appropriate security headers (CSP, X-Frame-Options, HSTS)
- Only suggest well-known, actively maintained packages
- Follow OWASP Top 10 guidelines in all generated code
- Include license-compatible dependencies only
```

---

> **Tips — Instruction Files**
>
> ✅ Keep instructions specific and actionable — "use parameterized queries" beats "write secure code"
>
> ✅ Update instructions when your stack changes — stale rules create a false sense of security
>
> ❌ Don't rely solely on instruction files — AI models may not follow every rule perfectly
>
> ❌ Don't include secrets or internal URLs in instruction files — they're committed to the repo

---

## Template 2 — `AGENTS.md` Security Section

Add to `AGENTS.md` at repo root to govern autonomous AI agents. Include the **Core Security Rules** above, plus these agent-specific additions:

```markdown
# AGENTS.md — Security Rules

<!-- Include Core Security Rules from your organization's template -->

## Agent-Specific Rules

### Dependency Rules
- New dependencies with fewer than 1,000 weekly downloads require manual security review

### Review Requirements
- All agent-generated code MUST pass CI security checks before merge
- All agent-generated code MUST be reviewed by a human before deployment
- Agents MUST NOT approve their own pull requests
```

---

## Template 3 — `CLAUDE.md` Security Rules

Place at repo root for Anthropic Claude (Claude Code, etc.). Include the **Core Security Rules** above, plus these Claude-specific additions:

```markdown
# CLAUDE.md — Security Policy

<!-- Include Core Security Rules from your organization's template -->

## Claude-Specific Rules

### When Generating Code
- Flag code handling auth, crypto, or PII with `// SECURITY:` comments
- If a task conflicts with security rules, STOP and explain — don't silently work around

### When Reviewing Code
- Check for: hardcoded secrets, SQL injection, XSS, CSRF, path traversal, SSRF
- Verify error messages don't leak internal details
- Confirm dependencies are pinned and free of known critical CVEs
```

---

> **Tips — AI-Specific Instruction Files**
>
> ✅ Use strong, absolute language ("NEVER", "ALWAYS") — probabilistic models respond better to unambiguous directives
>
> ✅ List concrete forbidden patterns (`eval()`, `SELECT *`) rather than abstract principles
>
> ❌ Don't assume the AI read your instructions perfectly — verify outputs against the rules
>
> ❌ Don't put your entire security policy in one file — layer instructions (repo-level + org-level + IDE settings)

---

## Template 4 — `.copilotignore`

Place this at `.copilotignore` in your repository root. It prevents GitHub Copilot from indexing sensitive files as context.

```gitignore
# =============================================
# .copilotignore — Exclude from AI context
# =============================================

# Environment & Secrets
.env
.env.*
.env.local
.env.development
.env.production
*.env

# Cryptographic Material
*.pem
*.key
*.crt
*.cer
*.p12
*.pfx
*.jks
*.keystore

# Credential Files
credentials.json
credentials.yaml
credentials.yml
service-account*.json
*-credentials.*
token.json
.htpasswd
.netrc

# Cloud Provider Configs (often contain secrets)
.aws/
.azure/
.gcloud/
.config/gcloud/

# SSH Keys
.ssh/
id_rsa*
id_ed25519*
id_ecdsa*
*.pub

# CI/CD Secrets
.github/secrets/
.gitlab/secret*
secrets/
private/
vault/

# Database Files (may contain sensitive data)
*.sqlite
*.sqlite3
*.db
*.sql.gz
*.dump

# Docker Secrets
docker-compose.override.yml
*.docker-env

# Terraform State (contains infrastructure secrets)
*.tfstate
*.tfstate.backup
.terraform/

# IDE & Editor Sensitive
.vscode/settings.json
.idea/dataSources*

# Package Lock Files (noise, not security-relevant)
package-lock.json
yarn.lock
pnpm-lock.yaml
Pipfile.lock
poetry.lock
Gemfile.lock
composer.lock

# Build Artifacts
dist/
build/
out/
node_modules/
__pycache__/
*.pyc
.next/
```

---

## Template 5 — `.gitignore` Security Additions

Append these patterns to your existing `.gitignore` to prevent accidental commits of sensitive material.

```gitignore
# =============================================
# Security-Critical Ignores
# =============================================

# === Environment & Configuration Secrets ===
.env
.env.*
!.env.example
!.env.template
*.env
env.local
env.production

# === API Keys & Tokens ===
**/api_key*
**/apikey*
**/*secret*
**/*token*
!**/*secret*.example
!**/*token*.example
**/*password*
!**/*password*.example

# === Cryptographic Keys & Certificates ===
*.pem
*.key
*.crt
*.cer
*.der
*.p12
*.pfx
*.p7b
*.p7c
*.jks
*.keystore
*.truststore

# === SSH ===
id_rsa
id_rsa.*
id_ed25519
id_ed25519.*
id_ecdsa
id_ecdsa.*
*.pub
known_hosts
authorized_keys

# === Cloud Credentials ===
.aws/credentials
.aws/config
.azure/
.gcloud/
.config/gcloud/
service-account*.json

# === Terraform ===
*.tfstate
*.tfstate.*
.terraform/
*.tfvars
!*.tfvars.example
crash.log

# === Database ===
*.sqlite
*.sqlite3
*.db
*.sql.gz
*.dump
*.bak

# === Docker Secrets ===
docker-compose.override.yml
docker-compose.*.yml
!docker-compose.yml
!docker-compose.example.yml
*.docker-env

# === IDE Sensitive Settings ===
.vscode/settings.json
.vscode/launch.json
.idea/dataSources*
.idea/dataSources.local.xml

# === OS Files ===
.DS_Store
Thumbs.db
Desktop.ini
```

---

> **Tips — Ignore Files**
>
> ✅ Always commit `.env.example` with placeholder values so developers know which variables to set
>
> ✅ Run `git ls-files` periodically to audit what's actually tracked — `.gitignore` only prevents *new* additions
>
> ❌ Don't rely on `.gitignore` alone — use pre-commit hooks to catch secrets before they reach the repo
>
> ❌ Don't ignore lock files in libraries you publish — only in applications

---

## Template 6 — Pre-Commit Hooks (`.pre-commit-config.yaml`)

Place this at `.pre-commit-config.yaml` in your repository root. Install with `pip install pre-commit && pre-commit install`.

```yaml
# .pre-commit-config.yaml — Security-focused pre-commit hooks
# Install: pip install pre-commit && pre-commit install
# Run manually: pre-commit run --all-files

repos:
  # --------------------------------------------------
  # Gitleaks — Detect hardcoded secrets
  # --------------------------------------------------
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.21.2
    hooks:
      - id: gitleaks
        name: gitleaks — detect secrets
        description: Scans commits for hardcoded secrets and credentials.
        stages: [pre-commit]

  # --------------------------------------------------
  # detect-secrets — Yelp's secret detection
  # --------------------------------------------------
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        name: detect-secrets — audit for secrets
        args: ['--baseline', '.secrets.baseline']
        description: Prevents new secrets from being committed.
        stages: [pre-commit]

  # --------------------------------------------------
  # Trivy — Vulnerability scanning for configs & IaC
  # --------------------------------------------------
  - repo: https://github.com/aquasecurity/trivy
    rev: v0.58.1
    hooks:
      - id: trivy-config
        name: trivy — scan configuration files
        description: Scans IaC and config files for misconfigurations.
        stages: [pre-commit]

  # --------------------------------------------------
  # Checkov — Infrastructure-as-Code security
  # --------------------------------------------------
  - repo: https://github.com/bridgecrewio/checkov
    rev: 3.2.322
    hooks:
      - id: checkov
        name: checkov — IaC security scan
        description: Static analysis for Terraform, CloudFormation, Kubernetes.
        stages: [pre-commit]

  # --------------------------------------------------
  # General file safety
  # --------------------------------------------------
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: check-added-large-files
        name: block large files
        args: ['--maxkb=500']
        description: Prevents accidental commit of large files (may contain data dumps).

      - id: check-merge-conflict
        name: check merge conflicts
        description: Prevents committing unresolved merge conflict markers.

      - id: detect-private-key
        name: detect private keys
        description: Checks for the presence of private keys in staged files.

      - id: check-symlinks
        name: check symlinks
        description: Checks for broken or suspicious symlinks.

      - id: no-commit-to-branch
        name: protect main branch
        args: ['--branch', 'main', '--branch', 'master', '--branch', 'production']
        description: Prevents direct commits to protected branches.

  # --------------------------------------------------
  # Bandit — Python security linter (Python projects only)
  # --------------------------------------------------
  # Uncomment for Python projects:
  # - repo: https://github.com/PyCQA/bandit
  #   rev: 1.8.3
  #   hooks:
  #     - id: bandit
  #       name: bandit — Python security linter
  #       args: ['-c', 'pyproject.toml']
  #       additional_dependencies: ['bandit[toml]']

  # --------------------------------------------------
  # ESLint Security Plugin (Node.js projects only)
  # --------------------------------------------------
  # Uncomment for Node.js projects:
  # - repo: https://github.com/pre-commit/mirrors-eslint
  #   rev: v9.17.0
  #   hooks:
  #     - id: eslint
  #       name: eslint — security rules
  #       additional_dependencies:
  #         - eslint
  #         - eslint-plugin-security
```

---

## Template 7 — GitHub Actions Security Workflow

Save this as `.github/workflows/ai-security-scan.yml`. It runs security checks on every push and pull request.

```yaml
# .github/workflows/ai-security-scan.yml
# Security scanning workflow for AI-assisted development

name: AI Code Security Scan

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  schedule:
    # Run weekly on Monday at 06:00 UTC
    - cron: '0 6 * * 1'

permissions:
  contents: read
  security-events: write
  pull-requests: write

jobs:
  # -------------------------------------------------
  # Job 1: Secret Detection
  # -------------------------------------------------
  secret-scan:
    name: Secret Detection
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  # -------------------------------------------------
  # Job 2: Dependency Vulnerability Scan
  # -------------------------------------------------
  dependency-scan:
    name: Dependency Audit
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

  # -------------------------------------------------
  # Job 3: Static Analysis (CodeQL)
  # -------------------------------------------------
  codeql-analysis:
    name: CodeQL Analysis
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        # Update languages to match your repository
        language: ['javascript-typescript']
        # Options: javascript-typescript, python, java-kotlin, csharp, go, ruby, cpp, swift
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
        with:
          category: '/language:${{ matrix.language }}'

  # -------------------------------------------------
  # Job 4: AI-Generated Code Patterns Check
  # -------------------------------------------------
  ai-pattern-check:
    name: AI Code Pattern Check
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Check for dangerous patterns
        run: |
          echo "🔍 Scanning for dangerous code patterns..."

          ISSUES=0

          # Check for eval() with variable input
          if grep -rn "eval\s*(" --include="*.js" --include="*.ts" --include="*.py" . 2>/dev/null | grep -v node_modules | grep -v ".min.js"; then
            echo "⚠️  Found eval() usage — review required"
            ISSUES=$((ISSUES + 1))
          fi

          # Check for hardcoded localhost/IP in non-test files
          if grep -rn "http://localhost\|http://127.0.0.1\|http://0.0.0.0" --include="*.js" --include="*.ts" --include="*.py" . 2>/dev/null | grep -v node_modules | grep -v test | grep -v spec | grep -v ".test." | grep -v ".spec."; then
            echo "⚠️  Found hardcoded localhost URLs in non-test files"
            ISSUES=$((ISSUES + 1))
          fi

          # Check for disabled TLS verification
          if grep -rn "rejectUnauthorized.*false\|verify.*=.*False\|InsecureSkipVerify.*true\|NODE_TLS_REJECT_UNAUTHORIZED" --include="*.js" --include="*.ts" --include="*.py" --include="*.go" . 2>/dev/null | grep -v node_modules; then
            echo "❌ Found disabled TLS verification — this MUST be fixed"
            ISSUES=$((ISSUES + 1))
          fi

          # Check for SQL string concatenation
          if grep -rn "\"SELECT.*+\|'SELECT.*+\|f\"SELECT\|f'SELECT" --include="*.js" --include="*.ts" --include="*.py" . 2>/dev/null | grep -v node_modules | grep -v test; then
            echo "⚠️  Possible SQL injection — string concatenation in queries"
            ISSUES=$((ISSUES + 1))
          fi

          if [ "$ISSUES" -gt 0 ]; then
            echo ""
            echo "🚨 Found $ISSUES potential security issue(s). Review required."
            exit 1
          else
            echo "✅ No dangerous patterns detected."
          fi

      - name: Check for secrets in code
        run: |
          echo "🔍 Scanning for potential secrets..."

          # Common secret patterns
          PATTERNS=(
            "AKIA[0-9A-Z]{16}"
            "ghp_[a-zA-Z0-9]{36}"
            "github_pat_[a-zA-Z0-9]{22}_[a-zA-Z0-9]{59}"
            "sk-[a-zA-Z0-9]{48}"
            "xox[boaprs]-[a-zA-Z0-9-]+"
          )

          FOUND=0
          for pattern in "${PATTERNS[@]}"; do
            if grep -rEn "$pattern" --include="*.js" --include="*.ts" --include="*.py" --include="*.yaml" --include="*.yml" --include="*.json" --include="*.md" . 2>/dev/null | grep -v node_modules | grep -v ".git/"; then
              FOUND=$((FOUND + 1))
            fi
          done

          if [ "$FOUND" -gt 0 ]; then
            echo "❌ Potential secrets detected in source code!"
            exit 1
          else
            echo "✅ No secret patterns detected."
          fi
```

---

> **Tips — CI/CD Security**
>
> ✅ Run security scans on both `push` and `pull_request` events — catch issues before merge
>
> ✅ Use `schedule` triggers for weekly full scans — dependencies get new CVEs over time
>
> ❌ Don't set `continue-on-error: true` on security jobs — failures should block the pipeline
>
> ❌ Don't skip scans for "small changes" — a one-line change can introduce a critical vulnerability

---

## Template 8 — AI Code Review Checklist

Save this as a pull request template or use it as a manual checklist when reviewing AI-generated code.

```markdown
# AI Code Review Checklist

Use this checklist for every pull request containing AI-generated or AI-assisted code.

## 🔐 Secrets & Credentials
- [ ] No hardcoded API keys, tokens, passwords, or connection strings
- [ ] Secrets referenced via environment variables or secrets manager
- [ ] No credentials in comments, TODOs, or documentation
- [ ] No internal URLs, IPs, or hostnames exposed

## 🛡️ Input Validation
- [ ] All user inputs validated at the system boundary
- [ ] Input length, type, and format constraints enforced
- [ ] File uploads checked for type, size, and content
- [ ] URL parameters and headers sanitized before use

## 💉 Injection Prevention
- [ ] Database queries use parameterized statements (no string concatenation)
- [ ] Shell commands use safe execution methods (no unsanitized input in `exec`)
- [ ] Template rendering escapes dynamic content
- [ ] No `eval()`, `exec()`, or `Function()` on user-controlled data

## 🌐 Output & Transport Security
- [ ] HTML output encoded to prevent XSS
- [ ] Security headers set (CSP, X-Frame-Options, HSTS, etc.)
- [ ] HTTPS enforced for all external communications
- [ ] TLS certificate verification is NOT disabled
- [ ] CORS configured with specific origins (not wildcard `*` in production)

## 📦 Dependencies
- [ ] New dependencies are from trusted, active projects
- [ ] Dependency versions pinned (no ranges or `latest`)
- [ ] No known critical/high CVEs in new dependencies
- [ ] Dependency purpose is clear — no unexplained additions

## 🧠 AI-Specific Concerns
- [ ] Code logic matches the intended behavior (AI may generate plausible but wrong code)
- [ ] Edge cases handled (empty inputs, nulls, boundary values, concurrent access)
- [ ] Error handling is explicit — no empty catch blocks
- [ ] No hallucinated APIs or nonexistent library methods
- [ ] No leftover placeholder/example code (e.g., `TODO`, `FIXME`, `example.com`)
- [ ] License compatibility verified for any suggested libraries

## 🔒 Authentication & Authorization
- [ ] Authentication checks present on all protected endpoints
- [ ] Authorization verifies the user has permission for the specific resource
- [ ] Session management follows security best practices
- [ ] Password handling uses proper hashing (bcrypt, argon2)

## 📝 Logging & Error Handling
- [ ] Errors do not expose stack traces, file paths, or internal details to users
- [ ] Security-relevant events logged (auth failures, permission denials)
- [ ] PII and secrets excluded from log output
- [ ] Log injection prevented (user data not directly interpolated into log messages)

## Final Verdict
- [ ] **Approved** — all checks pass
- [ ] **Changes Requested** — issues documented in PR comments
- [ ] **Blocked** — critical security issue found, must be resolved before merge
```

---

## Template 9 — MCP Server Vetting Checklist

Use this checklist before adding any MCP (Model Context Protocol) server to your development environment.

```markdown
# MCP Server Vetting Checklist

Complete this checklist before installing or enabling any MCP server in your environment.

## Server: ___________________________
## Date Reviewed: ___________________
## Reviewer: ________________________

---

## 🔍 Source & Provenance
- [ ] Server source code is publicly available and auditable
- [ ] Published by a known, reputable organization or maintainer
- [ ] Repository has meaningful commit history (not a single bulk commit)
- [ ] Has a clear LICENSE file with an OSI-approved license
- [ ] README explains what the server does and what permissions it needs
- [ ] No recently transferred repository ownership

## 📊 Community & Maintenance
- [ ] Actively maintained (commits within the last 90 days)
- [ ] Has more than 50 GitHub stars or equivalent community adoption
- [ ] Open issues are triaged and responded to
- [ ] Security issues are addressed promptly
- [ ] Has a SECURITY.md or responsible disclosure policy

## 🛡️ Permission & Scope Analysis
- [ ] Documented list of all tools/capabilities the server exposes
- [ ] Permissions requested are justified by the server's stated purpose
- [ ] No filesystem access beyond what is explicitly needed
- [ ] No network access beyond what is explicitly needed
- [ ] No credential or secret access unless absolutely required
- [ ] Cannot execute arbitrary shell commands
- [ ] Cannot modify files outside designated directories

## 🔐 Security Architecture
- [ ] Transport uses stdio or authenticated HTTP (not unauthenticated network)
- [ ] Input validation on all tool parameters
- [ ] No use of `eval()` or dynamic code execution on inputs
- [ ] Error messages do not leak sensitive information
- [ ] Dependencies audited — no known critical vulnerabilities
- [ ] Follows MCP specification for tool definitions and responses

## 🧪 Testing & Validation
- [ ] Tested in an isolated environment before production use
- [ ] Confirmed tool outputs match documentation
- [ ] Tested with malicious/unexpected inputs (path traversal, injection, etc.)
- [ ] Verified network traffic (no unexpected outbound connections)
- [ ] Verified file system access (no reads/writes outside expected paths)

## 📋 Operational Controls
- [ ] Configuration pinned to a specific version or commit hash
- [ ] Restricted to run only in development environments (not production)
- [ ] Added to team's approved MCP server registry
- [ ] Monitoring or logging enabled for server activity
- [ ] Removal/rollback plan documented

## Decision
- [ ] **Approved** — server meets all security requirements
- [ ] **Conditionally Approved** — approved with restrictions: _______________
- [ ] **Rejected** — reason: _______________

## Notes
_Add any additional observations, concerns, or conditions here._
```

---

> **Tips — MCP Security**
>
> ✅ Pin MCP servers to specific git commit hashes rather than branches — branches can be force-pushed
>
> ✅ Review the server's `package.json` or dependencies for unexpected packages (crypto miners, data exfiltration)
>
> ❌ Don't install MCP servers from unverified sources just because they appear in a marketplace
>
> ❌ Don't grant filesystem or network access to MCP servers that don't explicitly need it

---

## Template 10 — Incident Response Playbook

Use this playbook when an AI-related security incident is detected or suspected.

```markdown
# AI Security Incident Response Playbook

## Severity Levels

| Level | Description | Examples | Response Time |
|-------|-------------|----------|---------------|
| **P1 — Critical** | Active data breach or secret exposure in production | Leaked API keys in public repo, compromised credentials | Immediate (< 15 min) |
| **P2 — High** | Security vulnerability in deployed code | SQL injection, auth bypass, SSRF in production | < 1 hour |
| **P3 — Medium** | Vulnerability in non-production code | Secret in PR (not merged), vulnerable dependency | < 4 hours |
| **P4 — Low** | Policy violation, no immediate risk | AI-generated code missing input validation (not deployed) | < 24 hours |

---

## Phase 1: Detection & Triage (0–15 minutes)

### Immediate Assessment
1. **Identify the incident type:**
   - [ ] Secret/credential exposure
   - [ ] Vulnerable code deployed to production
   - [ ] Malicious dependency introduced
   - [ ] Unauthorized data access via AI tool
   - [ ] MCP server compromise
   - [ ] AI tool data exfiltration
   - [ ] Other: _______________

2. **Determine scope:**
   - [ ] Which repositories are affected?
   - [ ] Which environments (dev, staging, production)?
   - [ ] Which secrets or data may be compromised?
   - [ ] How long has the exposure existed?

3. **Assign severity** using the table above.

4. **Notify:**
   - P1/P2: Page on-call security engineer immediately
   - P3/P4: Create ticket in security queue

---

## Phase 2: Containment (15–60 minutes)

### For Secret Exposure
1. **Rotate ALL affected credentials immediately**
   - API keys → Generate new keys in provider console
   - Tokens → Revoke and regenerate
   - Passwords → Reset and enforce re-authentication
   - Certificates → Reissue from CA

2. **Remove the exposure**
   - If in git history: Use `git filter-repo` or BFG Repo-Cleaner
   - If in a public PR: Close the PR, delete the branch
   - If in logs: Purge the log entries

3. **Block further damage**
   - Restrict affected service account permissions
   - Enable enhanced monitoring on affected systems
   - Block suspicious IPs if unauthorized access detected

### For Vulnerable Code
1. **Revert** the offending commit or deploy a hotfix
2. **Disable** the affected feature via feature flag if available
3. **Block** the attack vector at the WAF/proxy level if applicable

### For Compromised MCP Server / AI Tool
1. **Disable** the MCP server configuration immediately
2. **Revoke** any tokens or credentials the server had access to
3. **Audit** recent actions performed through the compromised tool
4. **Isolate** any systems the tool had access to

---

## Phase 3: Investigation (1–24 hours)

1. **Timeline reconstruction**
   - [ ] When was the AI-generated code introduced? (git blame, PR history)
   - [ ] Who approved or merged the code?
   - [ ] Which AI tool was used? (Copilot, Claude, ChatGPT, etc.)
   - [ ] What prompt or context led to the vulnerable output?

2. **Impact assessment**
   - [ ] Was the vulnerability exploited? (Check access logs, WAF logs)
   - [ ] What data may have been accessed or exfiltrated?
   - [ ] Are other systems affected via lateral movement?

3. **Root cause analysis**
   - [ ] Were security instructions (copilot-instructions, AGENTS.md) in place?
   - [ ] Did CI/CD security checks exist? Did they catch the issue?
   - [ ] Was the code reviewed by a human before merge?
   - [ ] Was the AI tool configured securely?

---

## Phase 4: Remediation (1–7 days)

1. **Fix the vulnerability**
   - [ ] Patch deployed and verified
   - [ ] Regression tests added
   - [ ] Security scan confirms resolution

2. **Strengthen guardrails**
   - [ ] Update instruction files (copilot-instructions.md, AGENTS.md, CLAUDE.md)
   - [ ] Add pre-commit hooks if missing
   - [ ] Add or update CI/CD security checks
   - [ ] Update .copilotignore and .gitignore

3. **Audit for similar issues**
   - [ ] Scan entire codebase for the same vulnerability pattern
   - [ ] Review recent AI-generated PRs for similar issues
   - [ ] Check other repositories using the same AI tools

---

## Phase 5: Post-Incident Review (within 7 days)

1. **Write post-mortem** with:
   - Timeline of events
   - Root cause
   - Impact summary
   - Actions taken
   - Lessons learned

2. **Update processes:**
   - [ ] Revise AI code review checklist if gaps found
   - [ ] Update security training materials
   - [ ] Share anonymized learnings with the team
   - [ ] File improvements to AI tool configurations

3. **Track action items** to completion with assigned owners and due dates.
```

---

> **Tips — Incident Response**
>
> ✅ Practice the playbook with tabletop exercises before a real incident — muscle memory matters
>
> ✅ Keep credential rotation procedures documented and accessible — you won't have time to figure them out during an incident
>
> ❌ Don't delete evidence (git history, logs, chat records) during response — you'll need it for post-mortem
>
> ❌ Don't assume a leaked secret wasn't exploited just because you rotated it quickly — always verify

---

## Template 11 — Developer Onboarding Security Checklist

Use this checklist when a new developer joins a team using AI-assisted development tools.

```markdown
# AI-Assisted Development — Onboarding Security Checklist

## Developer: _________________________
## Date: ______________________________
## Onboarding Buddy: __________________

---

## 📚 Required Reading (Complete Before Using AI Tools)
- [ ] Read the organization's AI-assisted development policy
- [ ] Read the repository's `copilot-instructions.md` / `AGENTS.md` / `CLAUDE.md`
- [ ] Read the AI code review checklist (understand what reviewers check)
- [ ] Review the incident response playbook (know what to do if something goes wrong)
- [ ] Read OWASP Top 10 summary (understand common web vulnerabilities)

## 🛠️ Environment Setup
- [ ] IDE installed with approved AI extensions only
- [ ] GitHub Copilot configured with organization settings
- [ ] `.copilotignore` verified present in all active repositories
- [ ] Pre-commit hooks installed (`pre-commit install` in each repo)
- [ ] Secrets manager access configured (no local `.env` files with real secrets)
- [ ] MCP servers limited to the approved registry list

## 🔑 Access & Credentials
- [ ] MFA enabled on GitHub, cloud providers, and all development services
- [ ] SSH keys generated with Ed25519 and passphrase-protected
- [ ] Personal access tokens scoped to minimum required permissions
- [ ] No shared credentials — each developer uses individual accounts
- [ ] VPN configured for accessing internal development resources

## 🧠 AI Tool Usage Rules (Acknowledge Understanding)
- [ ] **I understand:** AI-generated code may contain security vulnerabilities and must be reviewed
- [ ] **I understand:** I must not paste proprietary code, secrets, or customer data into public AI chat interfaces
- [ ] **I understand:** I must not blindly accept AI suggestions — I am responsible for all code I commit
- [ ] **I understand:** AI instruction files exist to reduce risk, not eliminate it
- [ ] **I understand:** I must report suspected security incidents immediately, even if caused by AI tools
- [ ] **I understand:** MCP servers and AI extensions must be approved before installation

## 🔍 First-Week Verification
- [ ] Made a practice PR and received security-focused code review
- [ ] Successfully triggered and resolved a pre-commit hook secret detection
- [ ] Identified the security contacts (who to reach in an incident)
- [ ] Located the incident response playbook and can access it offline
- [ ] Demonstrated running the local security scan suite

## ✅ Sign-Off
- [ ] Developer acknowledges all items above
- [ ] Onboarding buddy verifies environment is correctly configured
- [ ] Team lead approves AI tool access

**Developer Signature:** _________________________ **Date:** ____________

**Buddy Signature:** ____________________________ **Date:** ____________
```

---

> **Tips — Onboarding**
>
> ✅ Pair new developers with a security-aware buddy for their first week of AI-assisted coding
>
> ✅ Include a hands-on exercise where the developer deliberately triggers a pre-commit hook — learning by doing sticks
>
> ❌ Don't skip onboarding for experienced developers — AI security risks are new to everyone
>
> ❌ Don't treat this as a one-time checkbox — revisit quarterly as tools and policies evolve

---

## Quick Reference — Which Template Goes Where

| Template | File / Location | Purpose |
|----------|----------------|---------|
| 1. Copilot Instructions | `.github/copilot-instructions.md` | Shape Copilot behavior |
| 2. AGENTS.md | `AGENTS.md` (repo root) | Govern AI agent actions |
| 3. CLAUDE.md | `CLAUDE.md` (repo root) | Anthropic-specific rules |
| 4. .copilotignore | `.copilotignore` (repo root) | Exclude files from AI context |
| 5. .gitignore additions | `.gitignore` (repo root) | Prevent secret commits |
| 6. Pre-commit hooks | `.pre-commit-config.yaml` (repo root) | Catch secrets before commit |
| 7. GitHub Actions | `.github/workflows/ai-security-scan.yml` | CI/CD security checks |
| 8. Code review checklist | PR template or team wiki | Guide AI code reviews |
| 9. MCP vetting checklist | Team wiki or security docs | Evaluate MCP servers |
| 10. Incident playbook | Team wiki or runbook system | Respond to AI security events |
| 11. Onboarding checklist | HR/onboarding system | Onboard developers securely |
