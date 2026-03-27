# Preventing Secret Leaks

> Multiple overlapping guardrails — because any single defense will eventually fail.

---

## Strategy Overview

Secret leak prevention works in layers. No single tool catches everything, so you stack defenses:

```
1. Don't store secrets in code           (eliminate the source)
2. Exclude secrets from AI context       (.copilotignore, .claudeignore)
3. Block secrets at commit time          (pre-commit hooks)
4. Scan for secrets in CI/CD             (push protection, automated scanning)
5. Instruct AI to avoid secrets          (instruction files)
6. Restrict AI file access               (permissions, sandboxing)
7. Rotate secrets on detection           (incident response)
```

---

## 1. Never Store Secrets in Code

Use environment variables and secret managers — not config files committed to git.

### Environment Variables (Development)

```bash
# .env (local only — NEVER committed)
DATABASE_URL=postgresql://localhost:5432/myapp
STRIPE_SECRET_KEY=sk_test_abc123
JWT_SECRET=local-dev-secret-replace-in-prod
```

```python
# Access in code
import os
database_url = os.environ["DATABASE_URL"]
```

```javascript
// Access in code
const stripeKey = process.env.STRIPE_SECRET_KEY;
```

### Secret Managers (Production)

| Tool | Best For | Integration |
|------|----------|------------|
| **GitHub Actions Secrets** | CI/CD workflows | `${{ secrets.MY_SECRET }}` |
| **AWS Secrets Manager** | AWS-hosted apps | SDK/CLI access with IAM |
| **HashiCorp Vault** | Multi-cloud, self-hosted | API-based, dynamic secrets |
| **1Password CLI** | Developer workflows | `op run -- command` |
| **Azure Key Vault** | Azure-hosted apps | SDK/managed identity |
| **Google Secret Manager** | GCP-hosted apps | SDK/service account |

```bash
# 1Password CLI — inject secrets at runtime
op run --env-file=.env.1password -- npm start

# AWS Secrets Manager in code
import boto3
client = boto3.client('secretsmanager')
secret = client.get_secret_value(SecretId='prod/database/password')
```

---

## 2. Comprehensive .gitignore

Prevent secrets from entering the repository:

```gitignore
# .gitignore — Secrets and Credentials

# Environment files
.env
.env.local
.env.production
.env.*.local
*.env

# Private keys and certificates
*.pem
*.key
*.p12
*.pfx
*.jks
*.keystore
id_rsa
id_ed25519

# Cloud provider credentials
.aws/credentials
.azure/credentials
gcloud-service-key.json
*-service-account.json
*-credentials.json

# Application secrets
config/secrets.yml
config/secrets.json
config/master.key
config/credentials.yml.enc

# Database
*.sqlite3
*.db
dump.sql
*.dump

# IDE and OS files that may contain paths/tokens
.idea/
.vscode/settings.json
*.swp
.DS_Store
Thumbs.db

# Docker secrets
docker-compose.override.yml
.docker/config.json

# Terraform state (contains secrets)
*.tfstate
*.tfstate.backup
.terraform/
```

---

## 3. .copilotignore — Exclude from Copilot Context

Prevent GitHub Copilot from reading sensitive files:

```gitignore
# .copilotignore — Files excluded from Copilot's context

# Environment and secrets
.env
.env.*
*.env

# Credentials and keys
*.pem
*.key
*.p12
*.pfx
*.jks

# Cloud credentials
.aws/
.azure/
*-service-account.json
*-credentials.json

# Configuration with secrets
config/secrets.*
config/master.key
docker-compose.override.yml

# Terraform state
*.tfstate
*.tfstate.backup

# Database files
*.sqlite3
*.db

# Internal documentation with sensitive architecture details
docs/internal/
docs/infrastructure/
```

> **Note:** `.copilotignore` uses the same syntax as `.gitignore`. Place it in the repository root.

### .claudeignore for Claude Code

```gitignore
# .claudeignore — Files excluded from Claude Code's context

# Same patterns as .copilotignore
.env
.env.*
*.pem
*.key
*.p12
config/secrets.*
*-credentials.json
*-service-account.json
*.tfstate
```

---

## 4. Pre-Commit Hooks

Block secrets before they enter git history.

### git-secrets (AWS)

```bash
# Install
# macOS
brew install git-secrets

# Linux
git clone https://github.com/awslabs/git-secrets.git
cd git-secrets && make install

# Configure for the repository
cd your-repo
git secrets --install
git secrets --register-aws

# Add custom patterns
git secrets --add 'sk_live_[a-zA-Z0-9]{24,}'        # Stripe live keys
git secrets --add 'ghp_[a-zA-Z0-9]{36}'              # GitHub personal access tokens
git secrets --add 'AKIA[0-9A-Z]{16}'                  # AWS access key IDs
git secrets --add --literal 'password\s*=\s*"[^"]{8,}"' # Hardcoded passwords

# Test — scan existing commits
git secrets --scan-history
```

### detect-secrets (Yelp)

```bash
# Install
pip install detect-secrets

# Generate baseline (marks existing false positives)
detect-secrets scan > .secrets.baseline

# Install pre-commit hook
# In .pre-commit-config.yaml:
```

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
        exclude: package-lock.json|\.secrets\.baseline
```

```bash
# Install hooks
pip install pre-commit
pre-commit install

# Audit baseline (review flagged items)
detect-secrets audit .secrets.baseline

# Scan for new secrets not in baseline
detect-secrets scan --baseline .secrets.baseline
```

### truffleHog

```bash
# Install
pip install trufflehog

# Scan the repository
trufflehog git file://. --only-verified

# Scan for high-entropy strings
trufflehog filesystem . --only-verified
```

### Combined Pre-Commit Configuration

```yaml
# .pre-commit-config.yaml
repos:
  # detect-secrets
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']

  # gitleaks
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks

  # Custom patterns
  - repo: local
    hooks:
      - id: no-env-files
        name: Prevent .env files from being committed
        entry: bash -c 'echo "ERROR: .env files should not be committed" && exit 1'
        language: system
        files: '^\.env'
        types: [file]
```

---

## 5. CI/CD Secret Scanning

### GitHub Secret Scanning (Built-in)

GitHub secret scanning is **enabled by default on public repositories** and available for private repos on GitHub Advanced Security:

- **Secret scanning alerts** — Detects secrets already in the repository
- **Push protection** — Blocks pushes that contain detected secrets

```yaml
# Verify secret scanning is enabled:
# Repository → Settings → Code security and analysis → Secret scanning ✓
# Repository → Settings → Code security and analysis → Push protection ✓
```

Supported secret types include:
- GitHub tokens (PAT, OAuth, App)
- AWS, Azure, GCP credentials
- Stripe, Twilio, SendGrid keys
- Database connection strings
- And 200+ more patterns

### GitGuardian

```yaml
# .github/workflows/gitguardian.yml
name: GitGuardian Secret Scan

on:
  push:
    branches: ['*']
  pull_request:

jobs:
  scanning:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: GitGuardian scan
        uses: GitGuardian/ggshield-action@v1
        env:
          GITHUB_PUSH_BEFORE_SHA: ${{ github.event.before }}
          GITHUB_PUSH_BASE_SHA: ${{ github.event.base_ref }}
          GITHUB_DEFAULT_BRANCH: ${{ github.event.repository.default_branch }}
          GITGUARDIAN_API_KEY: ${{ secrets.GITGUARDIAN_API_KEY }}
```

### Gitleaks in GitHub Actions

```yaml
# .github/workflows/gitleaks.yml
name: Gitleaks Secret Scan

on:
  pull_request:
  push:
    branches: [main]

jobs:
  gitleaks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 6. Instruction File Rules

Tell AI agents to never handle secrets directly:

```markdown
<!-- In copilot-instructions.md, AGENTS.md, or CLAUDE.md -->

## Secrets Policy

- NEVER hardcode API keys, passwords, tokens, or connection strings
- ALWAYS read secrets from environment variables: process.env.KEY_NAME
- NEVER log, print, or include secrets in error messages
- NEVER include real secret values in code comments or documentation
- Use obviously fake values for examples: "your-api-key-here", "PLACEHOLDER"
- If a test needs a secret, use a dedicated test value that has no real access
- Do NOT read .env files directly — use a dotenv library that loads them into env vars
```

---

## 7. Restrict AI File Access

Prevent AI agents from accessing directories that contain secrets:

### Directory Structure

```
project/
├── src/              ← AI can access
├── tests/            ← AI can access
├── docs/             ← AI can access
├── config/
│   ├── default.json  ← AI can access (no secrets)
│   └── secrets.json  ← EXCLUDED from AI context
├── .env              ← EXCLUDED from AI context
├── deploy/
│   └── keys/         ← EXCLUDED from AI context
├── .copilotignore    ← Excludes sensitive paths
└── .claudeignore     ← Excludes sensitive paths
```

### Copilot CLI — deny-tool restrictions

```bash
# Prevent the AI from reading environment variables
copilot-cli --deny-tool "bash(env)" --deny-tool "bash(printenv)"
```

---

## Secret Exposure Response

When a secret is detected in code or git history:

```
1. ROTATE THE SECRET IMMEDIATELY
   └── Generate new credentials
   └── Update production configuration
   └── Revoke the old credentials

2. Remove from git history (if committed)
   └── git filter-branch or BFG Repo-Cleaner
   └── Force push (coordinate with team)

3. Audit access
   └── Check cloud provider logs for unauthorized usage
   └── Review API call logs during exposure window

4. Post-mortem
   └── How did the secret get committed?
   └── Which guardrail failed?
   └── Add the missing detection pattern
```

---

## Tips

- ✅ Layer your defenses — `.gitignore` + `.copilotignore` + pre-commit hooks + CI scanning
- ✅ Enable GitHub push protection — it blocks commits containing detected secrets
- ✅ Keep `.env.example` in the repo with placeholder values, never real secrets
- ✅ Rotate secrets immediately when exposed — don't wait to "assess the risk"
- ✅ Use short-lived, scoped tokens instead of long-lived API keys
- ❌ Don't rely on `.gitignore` alone — it doesn't prevent AI from reading files
- ❌ Don't assume deleting a committed secret removes it — it's still in git history
- ❌ Don't skip pre-commit hooks because they're "annoying" — they're your last defense

---

## Next Steps

- 🔗 [Secrets in AI Workflows](secrets-in-ai-workflows.md) — Understand how secrets flow through AI systems
- 🔗 [Environment Variable Safety for AI Agents](environment-variables.md) — Securing env vars from AI access
- 🔗 [Git Secrets and Pre-Commit Hooks](git-secrets.md) — Deep dive into commit-time scanning
- 🔗 [Static Analysis for AI-Generated Code](../03-code-safety/static-analysis.md) — Scan for hardcoded credentials in code
