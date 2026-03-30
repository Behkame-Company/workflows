# Static Analysis for AI-Generated Code

> Automated scanning is the most reliable guardrail because it doesn't depend on the AI following instructions — it catches mistakes regardless of how the code was generated.

---

## Why Static Analysis Matters for AI Code

AI-generated code looks clean, passes surface-level review, and often introduces subtle vulnerabilities that humans miss during review. Static Application Security Testing (SAST) provides an **enforceable** safety net:

- ✅ Runs on every commit — no human fatigue
- ✅ Catches known vulnerability patterns deterministically
- ✅ Doesn't care if the AI "forgot" your security instructions
- ✅ Provides instant feedback before code reaches production

**Key insight:** Instructions tell AI what to do. Static analysis verifies what it actually did. You need both.

---

## Tools by Language

| Language | Tool | What It Catches |
|----------|------|----------------|
| JavaScript/TypeScript | ESLint + `eslint-plugin-security` | Injection, eval, non-literal regex |
| JavaScript/TypeScript | `eslint-plugin-no-secrets` | Hardcoded credentials |
| Python | Bandit | SQL injection, shell injection, weak crypto |
| Python | Pylint + security plugins | Unsafe deserialization, exec usage |
| Go | gosec | SQL injection, crypto misuse, file permissions |
| Multi-language | Semgrep | Custom rules for any pattern |
| Multi-language | SonarQube | Broad vulnerability + code smell detection |
| Multi-language | CodeQL | Deep semantic analysis via GitHub |

---

## Setting Up Language-Specific Tools

### ESLint Security for JavaScript/TypeScript

```bash
npm install --save-dev eslint eslint-plugin-security eslint-plugin-no-secrets
```

```json
// .eslintrc.json
{
  "plugins": ["security", "no-secrets"],
  "extends": ["plugin:security/recommended-legacy"],
  "rules": {
    "security/detect-object-injection": "warn",
    "security/detect-non-literal-regexp": "warn",
    "security/detect-non-literal-fs-filename": "warn",
    "security/detect-eval-with-expression": "error",
    "security/detect-no-csrf-before-method-override": "error",
    "security/detect-possible-timing-attacks": "warn",
    "no-secrets/no-secrets": "error"
  }
}
```

### Bandit for Python

```bash
pip install bandit
```

```yaml
# .bandit.yaml
skips: []
tests:
  - B101  # assert used
  - B102  # exec used
  - B103  # set bad file permissions
  - B104  # binding to all interfaces
  - B105  # hardcoded password string
  - B106  # hardcoded password in function argument
  - B107  # hardcoded password default
  - B108  # hardcoded tmp directory
  - B110  # try-except-pass
  - B301  # pickle usage
  - B302  # marshal usage
  - B303  # MD5/SHA1 usage
  - B304  # insecure cipher
  - B305  # insecure cipher mode
  - B306  # mktemp usage
  - B307  # eval usage
  - B308  # mark_safe usage
  - B310  # urllib urlopen
  - B311  # random for crypto
  - B312  # telnetlib
  - B321  # FTP
  - B323  # unverified SSL
  - B501  # requests with verify=False
  - B601  # paramiko shell
  - B602  # subprocess with shell=True
  - B603  # subprocess without shell
  - B604  # function call with shell=True
  - B605  # os.system
  - B608  # SQL injection via string
  - B609  # wildcard injection
```

### gosec for Go

```bash
go install github.com/securego/gosec/v2/cmd/gosec@latest
```

```bash
# Run on all packages
gosec ./...

# Generate SARIF output for GitHub integration
gosec -fmt sarif -out results.sarif ./...
```

---

## Custom Semgrep Rules for AI Code Patterns

Semgrep is particularly powerful because you can write rules targeting the exact patterns AI tends to produce.

```yaml
# .semgrep/ai-code-patterns.yml
rules:
  - id: ai-sql-string-concatenation
    patterns:
      - pattern-either:
          - pattern: |
              $QUERY = f"...SELECT...{$VAR}..."
          - pattern: |
              $QUERY = "...SELECT..." + $VAR
          - pattern: |
              $QUERY = f"...INSERT...{$VAR}..."
          - pattern: |
              $QUERY = f"...UPDATE...{$VAR}..."
          - pattern: |
              $QUERY = f"...DELETE...{$VAR}..."
    message: >
      SQL query uses string interpolation — use parameterized queries instead.
      AI models frequently generate this pattern.
    severity: ERROR
    languages: [python]

  - id: ai-os-system-usage
    pattern: os.system(...)
    message: >
      os.system() is vulnerable to shell injection. Use subprocess.run()
      with a list of arguments instead. AI commonly generates os.system().
    severity: ERROR
    languages: [python]

  - id: ai-hardcoded-credentials
    patterns:
      - pattern-either:
          - pattern: |
              $KEY = "..."
          - pattern: |
              $SECRET = "..."
          - pattern: |
              $PASSWORD = "..."
          - pattern: |
              $TOKEN = "..."
    pattern-where-python: >
      any(kw in str(vars.get("$KEY", "")).lower() or
          kw in str(vars.get("$SECRET", "")).lower() or
          kw in str(vars.get("$PASSWORD", "")).lower() or
          kw in str(vars.get("$TOKEN", "")).lower()
          for kw in ["password", "secret", "api_key", "apikey", "token", "credential"])
    message: >
      Possible hardcoded credential. Use environment variables or a secret manager.
    severity: WARNING
    languages: [python, javascript, typescript]

  - id: ai-innerhtml-xss
    pattern-either:
      - pattern: $EL.innerHTML = ...
      - pattern: document.write(...)
    message: >
      innerHTML/document.write is vulnerable to XSS. Use textContent
      or a sanitization library. AI frequently generates innerHTML.
    severity: ERROR
    languages: [javascript, typescript]

  - id: ai-eval-usage
    pattern-either:
      - pattern: eval(...)
      - pattern: Function(...)
    message: >
      eval() and Function() can execute arbitrary code.
      AI sometimes generates these for dynamic behavior.
    severity: ERROR
    languages: [javascript, typescript, python]

  - id: ai-md5-password-hashing
    patterns:
      - pattern-either:
          - pattern: hashlib.md5(...)
          - pattern: hashlib.sha1(...)
    message: >
      MD5/SHA1 are not suitable for password hashing.
      Use bcrypt, scrypt, or argon2. AI commonly uses md5 from training data.
    severity: ERROR
    languages: [python]

  - id: ai-verbose-error-response
    patterns:
      - pattern: |
          res.status(500).json({..., stack: $ERR.stack, ...})
    message: >
      Stack traces in error responses leak internal details.
      Log errors server-side and return generic messages to clients.
    severity: WARNING
    languages: [javascript, typescript]
```

Run these rules:

```bash
# Install Semgrep
pip install semgrep

# Run custom rules
semgrep --config .semgrep/ai-code-patterns.yml .

# Run with Semgrep's built-in security rules too
semgrep --config auto --config .semgrep/ai-code-patterns.yml .
```

---

## GitHub Actions Workflow for AI PRs

This workflow runs SAST tools specifically on pull requests from AI agents:

```yaml
# .github/workflows/ai-code-security-scan.yml
name: Security Scan for AI-Generated Code

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  security-events: write
  pull-requests: write

jobs:
  detect-ai-pr:
    runs-on: ubuntu-latest
    outputs:
      is_ai: ${{ steps.check.outputs.is_ai }}
    steps:
      - id: check
        run: |
          author="${{ github.event.pull_request.user.login }}"
          if [[ "$author" == *"[bot]"* ]] || \
             [[ "$author" == "copilot" ]] || \
             [[ "$author" == "github-actions" ]]; then
            echo "is_ai=true" >> $GITHUB_OUTPUT
          else
            echo "is_ai=false" >> $GITHUB_OUTPUT
          fi

  security-scan:
    needs: detect-ai-pr
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Semgrep
        uses: semgrep/semgrep-action@v1
        with:
          config: >-
            auto
            .semgrep/ai-code-patterns.yml
          generateSarif: true

      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: semgrep.sarif

      - name: Run Bandit (Python)
        if: hashFiles('**/*.py') != ''
        run: |
          pip install bandit
          bandit -r . -f json -o bandit-results.json || true
          # Fail on high-severity issues
          bandit -r . -ll -iii

      - name: Run ESLint Security (JavaScript/TypeScript)
        if: hashFiles('**/*.js') != '' || hashFiles('**/*.ts') != ''
        run: |
          npm ci
          npx eslint --no-eslintrc \
            --plugin security \
            --rule '{"security/detect-eval-with-expression": "error"}' \
            --rule '{"security/detect-non-literal-fs-filename": "warn"}' \
            --rule '{"security/detect-possible-timing-attacks": "warn"}' \
            "**/*.{js,ts}" || true

      - name: Strict gate for AI PRs
        if: needs.detect-ai-pr.outputs.is_ai == 'true'
        run: |
          echo "::warning::This PR was authored by an AI agent — applying strict security gates"
          # Re-run Semgrep in strict mode (fail on any finding)
          semgrep --config auto --config .semgrep/ai-code-patterns.yml --error .
```

---

## Integration with AI Coding Agents

### Copilot Coding Agent — copilot-setup-steps.yml

Add SAST tools to the agent's setup so they run before the agent commits:

```yaml
# .github/copilot-setup-steps.yml
steps:
  - name: Install dependencies
    run: npm ci

  - name: Install security scanning tools
    run: |
      pip install semgrep bandit
      npm install -g eslint eslint-plugin-security

  - name: Verify security tools available
    run: |
      semgrep --version
      bandit --version
```

Then in your `copilot-instructions.md`:

```markdown
## Security Requirements

Before committing any code changes, run:
- `semgrep --config auto --config .semgrep/ai-code-patterns.yml --error .`
- `bandit -r src/ -ll` (for Python files)
- `npx eslint --plugin security src/` (for JS/TS files)

Fix all findings before creating a pull request.
```

### Claude Code — CLAUDE.md

```markdown
## CLAUDE.md — Security Scanning

After making code changes, always run the security linter:

```bash
# Run before committing
semgrep --config auto --config .semgrep/ai-code-patterns.yml --error .
```

If Semgrep reports findings, fix them before proceeding. Common fixes:
- SQL string interpolation → use parameterized queries
- os.system() → use subprocess.run() with argument list
- innerHTML → use textContent
- Hardcoded secrets → use environment variables
```

---

## CodeQL for Deep Semantic Analysis

CodeQL goes beyond pattern matching — it understands data flow and can trace tainted input from source to sink.

```yaml
# .github/workflows/codeql.yml
name: CodeQL Analysis

on:
  pull_request:
    branches: [main]

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    strategy:
      matrix:
        language: [javascript, python]
    steps:
      - uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          queries: security-extended

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform analysis
        uses: github/codeql-action/analyze@v3
```

---

## Comparison of SAST Tools

| Tool | Best For | Speed | Custom Rules | CI Integration |
|------|----------|-------|-------------|----------------|
| Semgrep | Custom AI patterns, multi-language | ⚡ Fast | YAML-based (easy) | Native GitHub Action |
| Bandit | Python-specific security | ⚡ Fast | Plugin-based | CLI in any CI |
| ESLint + plugins | JS/TS security patterns | ⚡ Fast | JS-based rules | Built into most JS CI |
| gosec | Go-specific security | ⚡ Fast | Limited | CLI in any CI |
| CodeQL | Deep data-flow analysis | 🐢 Slow | QL language (advanced) | Native GitHub integration |
| SonarQube | Broad quality + security | 🐢 Slow | Custom rules (complex) | Self-hosted or Cloud |

**Recommendation:** Start with **Semgrep** (custom AI rules) + **language-specific tool** (Bandit/ESLint/gosec) + **CodeQL** (data flow analysis). This covers pattern matching, language-specific issues, and deep semantic analysis.

---

## Tips

- ✅ Run SAST in CI on every PR — not just AI-authored PRs (humans make the same mistakes)
- ✅ Start with Semgrep custom rules targeting your top 5 AI-generated vulnerabilities
- ✅ Make security scans blocking — don't just warn, fail the build
- ✅ Keep custom rules in version control (`.semgrep/` directory)
- ❌ Don't rely solely on instructions to AI — models skip rules unpredictably
- ❌ Don't skip SAST because "the AI was told to write secure code"
- ❌ Don't treat SAST warnings as noise — investigate every security finding
