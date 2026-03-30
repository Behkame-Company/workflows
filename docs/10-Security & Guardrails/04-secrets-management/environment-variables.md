# Environment Variable Safety for AI Agents

> AI agents with shell access can read your environment variables — design your environment so that doesn't matter.

---

## The Risk

AI coding agents increasingly have shell access to help you build, test, and debug code. That shell access means they can run commands like:

```bash
env                           # Lists all environment variables
printenv                      # Same as env
echo $DATABASE_URL            # Prints a specific variable
cat .env                      # Reads your dotenv file
grep -r PASSWORD .            # Searches for passwords in files
set                           # PowerShell — lists all variables
Get-ChildItem Env:            # PowerShell — lists environment variables
```

**Every environment variable is potential context for the AI.** If `DATABASE_URL` contains a production password, and the AI runs `env`, that password is now in the context window — sent to the model provider.

---

## Which AI Tools Have Shell Access

| Tool | Shell Access | Can Read Env Vars | Control Mechanism |
|------|-------------|-------------------|-------------------|
| **Copilot CLI** | ✅ Yes — primary feature | ✅ Yes | `--deny-tool` flag |
| **Claude Code** | ✅ Yes — bash tool | ✅ Yes | `--deny-tool`, permission prompts |
| **Copilot Coding Agent** | ✅ Yes — runs in Actions | ✅ Limited to Actions env | GitHub Secrets (masked in logs) |
| **Copilot Chat (IDE)** | ❌ No direct shell | ❌ No (reads files only) | `.copilotignore` |
| **Cursor** | ✅ Yes — terminal tool | ✅ Yes | Permission prompts |

---

## Mitigation Strategies

### 1. Restrict Environment-Reading Commands

Use tool restrictions to prevent AI from running commands that dump environment variables:

```bash
# Copilot CLI — deny specific shell patterns
# (exact flag syntax depends on your Copilot CLI version)
copilot --deny-tool "bash(env)" \
        --deny-tool "bash(printenv)" \
        --deny-tool "bash(set)" \
        --deny-tool "bash(export)"
```

```markdown
<!-- CLAUDE.md — instruct Claude to avoid env reading -->
## Shell Restrictions

NEVER run these commands:
- `env`, `printenv`, `export` (without arguments), `set`
- `echo $VARIABLE_NAME` for any secret variable
- `cat .env` or `cat` on any file containing credentials
- `grep` or `find` searching for passwords, tokens, or keys

If you need to verify an environment variable exists, use:
```bash
# Check existence without reading value
[ -n "$DATABASE_URL" ] && echo "DATABASE_URL is set" || echo "DATABASE_URL is not set"
```
```

### 2. Run Agents in Containers with Minimal Env Vars

Don't give AI agents your full development environment:

```dockerfile
# Dockerfile.ai-agent — minimal environment for AI coding tasks
FROM node:20-slim

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY src/ ./src/
COPY tests/ ./tests/

# Only set non-secret environment variables
ENV NODE_ENV=development
ENV LOG_LEVEL=debug
ENV PORT=3000

# Secrets are NOT in the container — use a secret manager at runtime
# The AI agent can code and test without access to real credentials
```

```bash
# Run the AI session inside the container
docker run -it --rm \
  -v $(pwd)/src:/app/src \
  -v $(pwd)/tests:/app/tests \
  ai-dev-container bash
```

### 3. Use Secret Managers That Require Authentication

Instead of environment variables, use secret managers that require explicit authentication:

```python
# ❌ Secret in environment variable — AI can read it
import os
db_password = os.environ["DB_PASSWORD"]

# ✅ Secret from AWS Secrets Manager — requires IAM auth
import boto3
def get_db_password():
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId='prod/db/password')
    return response['SecretString']
```

```javascript
// ❌ Secret in environment variable
const apiKey = process.env.STRIPE_SECRET_KEY;

// ✅ Secret from 1Password CLI — requires authentication
const { execSync } = require('child_process');
function getApiKey() {
  return execSync('op read "op://Vault/Stripe/secret-key"').toString().trim();
}
```

The AI agent can see the *code* that fetches the secret, but can't access the secret itself without proper authentication.

### 4. Keep Secrets Out of Shell History and Startup Files

```bash
# ❌ Don't put secrets in shell startup files
# ~/.bashrc or ~/.zshrc
export API_KEY="sk_live_abc123..."  # AI can read this file

# ❌ Don't type secrets in the terminal
export SECRET="my-password"  # Now in shell history

# ✅ Use a .env file (excluded from AI context via .copilotignore)
# .env
API_KEY=sk_live_abc123...

# ✅ Use direnv for per-project secrets (file is gitignored)
# .envrc
export API_KEY=$(op read "op://Dev/API/key")
```

### 5. Separate Development and Production Credentials

Never use production secrets in development:

```bash
# .env.example (committed — shows required variables)
DATABASE_URL=postgresql://localhost:5432/myapp_dev
STRIPE_SECRET_KEY=sk_test_placeholder
JWT_SECRET=dev-only-not-a-real-secret
REDIS_URL=redis://localhost:6379

# .env (NOT committed — has real dev values)
DATABASE_URL=postgresql://localhost:5432/myapp_dev
STRIPE_SECRET_KEY=sk_test_abc123  # Test mode key — limited blast radius
JWT_SECRET=random-dev-secret-abc123
REDIS_URL=redis://localhost:6379
```

**Key principle:** If an AI agent leaks your development secrets, the damage should be minimal — test API keys, local database passwords, development-only tokens.

---

## Safe vs. Unsafe Environment Patterns

### ❌ Unsafe: Production Secrets in Dev Environment

```bash
# Developer's .env
DATABASE_URL=postgresql://admin:Pr0dP@ss!@prod-db.example.com:5432/production
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
STRIPE_SECRET_KEY=sk_live_51abc123def456...
```

If the AI reads these values, production systems are exposed.

### ✅ Safe: Local-Only Dev Secrets

```bash
# Developer's .env
DATABASE_URL=postgresql://dev:devpass@localhost:5432/myapp_dev
AWS_SECRET_ACCESS_KEY=test-key-no-real-access
STRIPE_SECRET_KEY=sk_test_abc123  # Stripe test mode — no real charges
```

Even if leaked, these secrets provide no access to production systems.

### ❌ Unsafe: Secrets in Docker Compose

```yaml
# docker-compose.yml — committed to git
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: "SuperSecretProdPassword123!"  # Hardcoded
```

### ✅ Safe: Secrets via .env Reference

```yaml
# docker-compose.yml — committed to git
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}  # From .env (gitignored)
```

### ❌ Unsafe: Secrets in Package Scripts

```json
{
  "scripts": {
    "deploy": "API_KEY=sk_live_abc123 ./deploy.sh"
  }
}
```

### ✅ Safe: Scripts Reference Env Vars

```json
{
  "scripts": {
    "deploy": "./deploy.sh"
  }
}
```

```bash
#!/bin/bash
# deploy.sh — reads secrets from environment at runtime
if [ -z "$API_KEY" ]; then
  echo "ERROR: API_KEY environment variable is required"
  exit 1
fi
```

---

## The .env.example Pattern

Always maintain a `.env.example` file that shows required variables without real values:

```bash
# .env.example — committed to git
# Copy this to .env and fill in real values

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/dbname

# Authentication
JWT_SECRET=generate-a-random-string-here
SESSION_SECRET=generate-another-random-string

# Third-Party APIs
STRIPE_SECRET_KEY=sk_test_your_test_key_here
SENDGRID_API_KEY=SG.your_key_here

# Storage
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
S3_BUCKET=your-bucket-name

# Optional
LOG_LEVEL=debug
PORT=3000
```

AI agents can read `.env.example` to understand what configuration is needed without seeing real values.

---

## Environment Variable Audit

Periodically review what's in your environment:

```bash
# List all env vars and check for sensitive ones
env | grep -iE 'key|secret|password|token|credential|auth' | \
  sed 's/=.*/=***REDACTED***/'

# Expected output:
# DATABASE_URL=***REDACTED***
# JWT_SECRET=***REDACTED***
# AWS_SECRET_ACCESS_KEY=***REDACTED***
```

If you see production credentials in your development environment, move them to a secret manager.

---

## Tips

- ✅ Use `.env.example` with placeholder values — commit this, not `.env`
- ✅ Use test/development credentials locally — never production secrets
- ✅ Run AI agents in containers with minimal environment variables
- ✅ Use secret managers for production — env vars are for development only
- ✅ Restrict AI tools from running `env`/`printenv` commands when possible
- ❌ Don't put secrets in shell startup files (`~/.bashrc`, `~/.zshrc`)
- ❌ Don't put secrets in `docker-compose.yml` or `package.json`
- ❌ Don't assume the AI won't run `env` — it might, even without being asked
