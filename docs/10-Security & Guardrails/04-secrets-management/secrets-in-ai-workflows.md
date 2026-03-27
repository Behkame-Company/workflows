# Secrets in AI Workflows

> Everything in the context window is sent to the model provider — understand how secrets flow through AI systems to prevent exposure.

---

## How AI Agents Encounter Secrets

AI coding agents aren't just reading your prompts — they're reading your files, running your commands, and interacting with your tools. At every step, they may encounter secrets:

```
Your codebase ──────────────┐
Your .env files ────────────┤
Your shell environment ─────┤──→ AI Context Window ──→ Model Provider
Your command output ────────┤       (Anthropic, OpenAI, GitHub)
Your MCP server responses ──┘
```

**The fundamental risk:** Anything the AI sees gets sent to the model provider's API. This includes code snippets, file contents, environment variables, and command output.

---

## Risk Scenarios

### 1. AI Reads .env and Embeds Secrets in Code

```python
# Developer has .env with DATABASE_URL=postgresql://admin:s3cr3tP@ss@prod.db.example.com/app

# AI reads the file to "understand the config" and generates:
DATABASE_URL = "postgresql://admin:s3cr3tP@ss@prod.db.example.com/app"  # From .env

# Now the production password is in source code and git history
```

### 2. AI Logs Secrets in Debug Output

```python
# AI adds debugging to help troubleshoot a connection issue:
import os
print(f"Connecting with: {os.environ['DATABASE_URL']}")
print(f"API Key: {os.environ['STRIPE_SECRET_KEY']}")
# These appear in logs, CI output, or terminal history
```

### 3. AI Commits Secrets to Git

```javascript
// AI generates a configuration file with real values:
module.exports = {
  stripe: {
    secretKey: 'sk_live_abc123def456...',  // Real key from context
  },
  database: {
    password: 'production_password_123',    // Real password
  }
};
// If committed, this is in git history forever (even after deletion)
```

### 4. AI Sends Secrets to MCP Servers

```
AI Agent: "I need to query the database to understand the schema"
    → Calls MCP database server with connection string
    → MCP server logs include the connection string
    → MCP server may be running on a third-party service
```

### 5. Secrets in Context Window Sent to LLM Provider

```
Developer: "Debug this database connection"
    → Copilot reads db.config.js (contains connection string)
    → Connection string with password becomes part of the prompt
    → Prompt sent to GitHub/OpenAI API
    → Now exists in API logs, potentially in training data
```

---

## What Counts as a Secret

Not all sensitive information is obvious:

| Category | Examples | Risk Level |
|----------|----------|-----------|
| **API keys** | `sk_live_*`, `AKIA*`, `ghp_*` | 🔴 Critical — direct access to services |
| **Database credentials** | Connection strings with passwords | 🔴 Critical — access to all data |
| **Private keys** | SSH keys, TLS certificates, signing keys | 🔴 Critical — identity impersonation |
| **Tokens** | JWT secrets, session secrets, OAuth tokens | 🔴 Critical — session hijacking |
| **Internal URLs** | Staging servers, admin panels, internal APIs | 🟠 High — reveals infrastructure |
| **PII** | Customer emails, addresses, phone numbers | 🟠 High — privacy/compliance violation |
| **Business data** | Revenue numbers, unreleased features, pricing | 🟡 Medium — competitive intelligence |
| **Infrastructure details** | IP ranges, server counts, architecture docs | 🟡 Medium — attack surface mapping |

---

## How Context Works by Tool

### GitHub Copilot (Code Completion)

- **What's sent:** Current file content, open tabs, neighboring files
- **Where it goes:** GitHub's Copilot API (backed by OpenAI/Anthropic)
- **Secret risk:** Secrets in open files or adjacent files appear in completions
- **Control:** `.copilotignore` to exclude files from context

### GitHub Copilot Chat / Copilot Coding Agent

- **What's sent:** Conversation history, referenced files, search results
- **Where it goes:** GitHub's Copilot API
- **Secret risk:** Files mentioned in chat or pulled by the agent
- **Control:** `.copilotignore`, don't reference files containing secrets

### Claude Code (Anthropic)

- **What's sent:** Full file contents read during the session, command output
- **Where it goes:** Anthropic's API
- **Secret risk:** Any file Claude reads, any command output Claude sees
- **Control:** `.claudeignore`, limit file access, use `--deny-tool` flags

### Copilot CLI

- **What's sent:** Conversation, command output from shell tool
- **Where it goes:** GitHub's Copilot API
- **Secret risk:** `env`, `printenv`, `cat .env` output visible to the agent
- **Control:** Limit shell access, keep secrets out of environment

---

## Risk Matrix

| Scenario | Likelihood | Impact | Risk Score | Primary Mitigation |
|----------|-----------|--------|------------|-------------------|
| AI reads `.env` file | 🔴 High | 🔴 Critical | **Critical** | `.copilotignore`, `.claudeignore` |
| AI logs secrets in debug code | 🟠 Medium | 🟠 High | **High** | Code review, SAST rules |
| AI commits secrets to git | 🟠 Medium | 🔴 Critical | **Critical** | Pre-commit hooks, secret scanning |
| AI sends secrets to MCP servers | 🟡 Low | 🟠 High | **Medium** | MCP server vetting, network controls |
| Secrets in context sent to LLM | 🔴 High | 🟡 Medium | **High** | File exclusion, secret-free dev environments |
| AI generates code with hardcoded secrets | 🟠 Medium | 🔴 Critical | **Critical** | SAST, pre-commit hooks |
| Secrets visible in AI-generated logs | 🟠 Medium | 🟠 High | **High** | Log scrubbing, structured logging |

---

## Defense Layers

```
Layer 1: Prevent secrets from reaching the AI
    ├── .copilotignore / .claudeignore
    ├── Minimal environment variables in AI sessions
    └── Secret-free development environments

Layer 2: Detect secrets in AI output
    ├── Pre-commit hooks (git-secrets, detect-secrets)
    ├── Static analysis (Semgrep rules for hardcoded creds)
    └── GitHub secret scanning

Layer 3: Limit blast radius
    ├── Short-lived tokens instead of long-lived keys
    ├── Least-privilege API keys
    ├── Secret rotation on exposure
    └── Separate dev/staging/production credentials
```

---

## Quick Wins

1. **Add `.copilotignore`** — exclude `.env`, `*.pem`, `*.key`, and config files with secrets
2. **Install pre-commit hooks** — `detect-secrets` catches secrets before they enter git
3. **Enable GitHub secret scanning** — catches secrets that slip through
4. **Use placeholder values** — keep `.env.example` with fake values in the repo
5. **Rotate exposed secrets immediately** — assume any secret the AI saw is compromised

---

## Tips

- ✅ Treat the AI context window as **semi-public** — anything the AI sees could be logged
- ✅ Use short-lived, scoped tokens instead of long-lived API keys where possible
- ✅ Keep development secrets separate from production secrets
- ✅ Audit what files AI agents access during a session
- ❌ Don't paste secrets into AI chat conversations
- ❌ Don't assume secrets in `.env` are "safe" — AI can read local files
- ❌ Don't trust AI-generated configuration files without checking for embedded secrets

---

## Next Steps

- 🔗 [Preventing Secret Leaks](preventing-leaks.md) — Practical guardrails and tooling
- 🔗 [Environment Variable Safety for AI Agents](environment-variables.md) — Secure env var handling
- 🔗 [Git Secrets and Pre-Commit Hooks](git-secrets.md) — Last line of defense before commits
- 🔗 [AI-Generated Code Risks](../03-code-safety/ai-generated-code-risks.md) — Broader security risks in AI code
