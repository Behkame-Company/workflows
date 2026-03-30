# Git Secrets and Pre-Commit Hooks

> The last line of defense — automated secret detection that blocks commits before secrets enter git history.

---

## Why Pre-Commit Scanning Matters

Once a secret is committed to git, it's in the history **forever** (unless you rewrite history, which is disruptive). Even if you delete the file in the next commit, anyone with repository access can find it:

```bash
# Anyone can find secrets in git history
git log --all --full-history -p | grep -i "api_key\|secret\|password"
```

Pre-commit hooks catch secrets **before** they enter history — the cheapest point to fix.

### Integration with AI Workflows

AI agents generate code fast. In an AI-assisted workflow, the cycle is:

```
AI generates code
    → Pre-commit hook scans for secrets
    → If found: commit blocked, AI can fix
    → If clean: commit proceeds
    → CI/CD scanning provides second check
```

This is especially important because AI agents may:
- Copy secrets from `.env` files into source code
- Generate configuration with placeholder values that look like real secrets
- Include API keys from training data patterns

---

## Tool Comparison

| Tool | Maintainer | Approach | Custom Patterns | Speed | Best For |
|------|-----------|----------|----------------|-------|----------|
| **git-secrets** | AWS | Pattern matching | Regex-based | ⚡ Fast | AWS-centric projects |
| **detect-secrets** | Yelp | Entropy + patterns | Plugin-based | ⚡ Fast | Baseline-driven scanning |
| **gitleaks** | Zach Rice | Regex + entropy | TOML config | ⚡ Fast | Comprehensive scanning |
| **truffleHog** | Truffle Security | Verification | Detector-based | 🐢 Slower | Verified secrets only |
| **GitHub Secret Scanning** | GitHub | Pattern matching | N/A (built-in) | N/A (server-side) | GitHub-hosted repos |

---

## git-secrets (AWS)

### Installation

```bash
# macOS
brew install git-secrets

# Linux (from source)
git clone https://github.com/awslabs/git-secrets.git
cd git-secrets
sudo make install

# Windows (via scoop or manual)
scoop install git-secrets
```

### Setup for a Repository

```bash
cd your-repo

# Install hooks (pre-commit, commit-msg, prepare-commit-msg)
git secrets --install

# Register AWS patterns (detects AWS access keys and secret keys)
git secrets --register-aws

# Add custom patterns for common secret formats
git secrets --add 'sk_live_[a-zA-Z0-9]{24,}'                    # Stripe live keys
git secrets --add 'sk_test_[a-zA-Z0-9]{24,}'                    # Stripe test keys
git secrets --add 'ghp_[a-zA-Z0-9]{36}'                         # GitHub PATs
git secrets --add 'github_pat_[a-zA-Z0-9]{22}_[a-zA-Z0-9]{59}'  # GitHub fine-grained PATs
git secrets --add 'xoxb-[0-9]{11,13}-[a-zA-Z0-9]{24}'           # Slack bot tokens
git secrets --add 'SG\.[a-zA-Z0-9]{22}\.[a-zA-Z0-9]{43}'        # SendGrid API keys
git secrets --add 'sq0csp-[a-zA-Z0-9_-]{43}'                    # Square API keys

# Add allowed patterns (false positive exclusions)
git secrets --add --allowed 'sk_test_placeholder'
git secrets --add --allowed 'your-api-key-here'
git secrets --add --allowed 'EXAMPLE'

# Verify configuration
git secrets --list
```

### Scan Existing History

```bash
# Scan all commits
git secrets --scan-history

# Scan specific files
git secrets --scan path/to/file.js

# Scan staged changes only
git secrets --scan --cached
```

### Global Configuration (All Repos)

```bash
# Apply to all future cloned repos
git secrets --install --global
git secrets --register-aws --global
```

---

## detect-secrets (Yelp)

detect-secrets uses a **baseline** approach — it generates a file listing known secrets (or false positives), then only alerts on *new* secrets.

### Installation

```bash
pip install detect-secrets
```

### Generate Baseline

```bash
# Scan the entire repo and create a baseline file
detect-secrets scan > .secrets.baseline

# Scan with specific plugins
detect-secrets scan \
  --list-all-plugins  # See available plugins first

detect-secrets scan \
  --disable-plugin KeywordDetector \
  > .secrets.baseline
```

### Audit the Baseline

```bash
# Interactively review each finding — mark as real secret or false positive
detect-secrets audit .secrets.baseline
```

This opens an interactive prompt for each finding:

```
Secret:      1 of 15
Filename:    config/settings.py
Secret Type: High Entropy String
Secret:      "dGhpcyBpcyBhIHRlc3Q="

Is this a valid secret? (y)es, (n)o, (s)kip, (b)ack, (q)uit:
```

### Pre-Commit Hook Configuration

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
        exclude: |
          (?x)^(
            package-lock\.json|
            \.secrets\.baseline|
            .*\.snap
          )$
```

```bash
# Install pre-commit framework
pip install pre-commit
pre-commit install

# Test the hook
pre-commit run detect-secrets --all-files
```

### Custom Plugins

```python
# .detect-secrets-plugins/custom_patterns.py
from detect_secrets.plugins.base import RegexBasedDetector

class CustomAPIKeyDetector(RegexBasedDetector):
    secret_type = 'Custom API Key'

    denylist = [
        # Company-specific API key patterns
        r'mycompany_api_[a-zA-Z0-9]{32}',
        # Internal service tokens
        r'svc_token_[a-zA-Z0-9]{48}',
    ]
```

```bash
# Run with custom plugin
detect-secrets scan --plugin .detect-secrets-plugins/custom_patterns.py > .secrets.baseline
```

---

## gitleaks

### Installation

```bash
# macOS
brew install gitleaks

# Linux
curl -sSfL https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks-linux-amd64 -o gitleaks
chmod +x gitleaks
sudo mv gitleaks /usr/local/bin/

# Docker
docker pull ghcr.io/gitleaks/gitleaks:latest
```

### Configuration

```toml
# .gitleaks.toml
title = "Custom Gitleaks Configuration"

# Custom rules
[[rules]]
id = "stripe-live-key"
description = "Stripe Live Secret Key"
regex = '''sk_live_[a-zA-Z0-9]{24,}'''
tags = ["secret", "stripe"]
severity = "critical"

[[rules]]
id = "generic-password-assignment"
description = "Password assigned in code"
regex = '''(?i)(password|passwd|pwd)\s*[=:]\s*["'][^"']{8,}["']'''
tags = ["secret", "password"]
severity = "high"

[[rules]]
id = "private-key-block"
description = "Private Key in code"
regex = '''-----BEGIN (RSA |EC |DSA |OPENSSH )?PRIVATE KEY-----'''
tags = ["secret", "key"]
severity = "critical"

# Allowlist — exclude false positives
[allowlist]
description = "Known false positives"
paths = [
  '''\.secrets\.baseline$''',
  '''package-lock\.json$''',
  '''go\.sum$''',
  '''\.env\.example$''',
]
regexes = [
  '''placeholder''',
  '''your-.*-here''',
  '''EXAMPLE''',
  '''test-key-not-real''',
]
```

### Usage

```bash
# Scan current directory
gitleaks detect --source . --verbose

# Scan git history
gitleaks detect --source . --log-opts="--all"

# Pre-commit hook
gitleaks protect --staged --verbose

# Generate report
gitleaks detect --source . --report-format json --report-path gitleaks-report.json
```

### Pre-Commit Hook

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

---

## GitHub Secret Scanning

### Enabling Push Protection

GitHub's built-in secret scanning is the easiest to set up:

```
Repository → Settings → Code security and analysis

✅ Secret scanning          — Alerts when secrets are found in history
✅ Push protection          — BLOCKS pushes containing detected secrets
✅ Validity checks          — Verifies if detected secrets are still active
```

**Push protection** is the most valuable feature — it prevents secrets from being pushed to the remote, even if pre-commit hooks were bypassed.

### Supported Patterns

GitHub detects 200+ secret types automatically:
- GitHub tokens (PAT, OAuth, App tokens)
- AWS access keys and secret keys
- Azure, GCP service account keys
- Stripe, Twilio, SendGrid API keys
- Database connection strings
- Private keys

### Custom Patterns

```yaml
# Repository → Settings → Code security → Custom patterns
# Define patterns for company-specific secrets

Pattern name: Internal API Key
Secret format: myco_api_[a-zA-Z0-9]{32}
Before secret: (key|token|secret)\s*[=:]\s*["']?
After secret: ["']?\s*[,;]?\s*$
```

### Handling Blocked Pushes

When push protection blocks a commit:

```
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: - GITHUB PUSH PROTECTION
remote:   —— Stripe API Key ——
remote:   locations:
remote:     - commit: abc123def
remote:       path: src/config.js:15
remote:
remote: To push, you must remove the secret from your commits.
```

**Resolution:**

```bash
# Option 1: Remove the secret and amend the commit
# (edit the file to remove the secret)
git add -A
git commit --amend --no-edit
git push

# Option 2: If it was in an earlier commit, use interactive rebase
git rebase -i HEAD~3  # Go back to the commit with the secret
# Mark the commit as 'edit', remove the secret, continue rebase
```

---

## GitHub Actions Workflows

### Gitleaks CI Scan

```yaml
# .github/workflows/gitleaks.yml
name: Secret Scanning

on:
  pull_request:
  push:
    branches: [main, develop]

jobs:
  gitleaks:
    name: Gitleaks Secret Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITLEAKS_LICENSE: ${{ secrets.GITLEAKS_LICENSE_KEY }}  # Optional for premium

      - name: Upload SARIF
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: results.sarif
```

### detect-secrets CI Scan

```yaml
# .github/workflows/detect-secrets.yml
name: Secret Detection

on:
  pull_request:
    branches: [main]

jobs:
  detect-secrets:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install detect-secrets
        run: pip install detect-secrets

      - name: Check for new secrets
        run: |
          # Scan for secrets not in baseline
          detect-secrets scan --baseline .secrets.baseline

          # Fail if new secrets found
          if detect-secrets scan --list | grep -v "\.secrets\.baseline"; then
            echo "::error::New secrets detected! Run 'detect-secrets scan > .secrets.baseline' and audit."
            exit 1
          fi
```

### Combined Workflow for AI PRs

```yaml
# .github/workflows/ai-secret-guard.yml
name: AI PR Secret Guard

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  secret-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Gitleaks scan
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: detect-secrets scan
        run: |
          pip install detect-secrets
          detect-secrets scan --baseline .secrets.baseline

      - name: Check diff for secret patterns
        run: |
          # Additional grep-based check for patterns tools might miss
          FINDINGS=$(git diff origin/main...HEAD | grep -iE \
            '(sk_live|sk_test|AKIA|ghp_|github_pat|xoxb-|SG\.)' || true)

          if [ -n "$FINDINGS" ]; then
            echo "::error::Possible secrets in PR diff"
            echo "$FINDINGS"
            exit 1
          fi

      - name: Comment on PR if AI-authored
        if: failure() && contains(github.event.pull_request.user.login, '[bot]')
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '🚨 **Secret detected in AI-generated code.**\n\nThis PR contains what appears to be a secret or credential. Please remove it before merging.\n\nRun `gitleaks detect --source . --verbose` locally to identify the finding.'
            })
```

---

## Quick Setup Guide

Get basic secret scanning running in under 5 minutes:

```bash
# 1. Install tools
pip install detect-secrets pre-commit

# 2. Generate baseline
detect-secrets scan > .secrets.baseline

# 3. Create pre-commit config
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
EOF

# 4. Install hooks
pre-commit install

# 5. Test
echo 'API_KEY="sk_live_abc123def456ghi789jkl012"' > test-secret.txt
git add test-secret.txt
git commit -m "test"  # Should be BLOCKED

# 6. Clean up test file
rm test-secret.txt
```

---

## Tips

- ✅ Install pre-commit hooks as the **first thing** when setting up a new repo
- ✅ Use both pre-commit hooks (local) AND CI scanning (remote) — belt and suspenders
- ✅ Enable GitHub push protection — it's free and catches what local hooks miss
- ✅ Audit your detect-secrets baseline regularly — false positives accumulate
- ✅ Add custom patterns for your organization's internal secret formats
- ❌ Don't skip the baseline audit — unreviewed findings are ignored forever
- ❌ Don't rely on `.gitignore` to prevent secret commits — it only prevents untracked files
- ❌ Don't assume rewriting git history fully removes secrets — anyone who cloned may have them
