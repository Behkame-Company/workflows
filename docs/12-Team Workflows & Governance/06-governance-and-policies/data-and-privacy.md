# Data and Privacy Considerations

> A practical guide to understanding, classifying, and protecting data when using AI-assisted development tools. This document covers what data gets sent to AI services, where the risks lie, and how to implement layered protections to keep sensitive information safe.

---

## Why Data Privacy Matters for AI Tools

When you use an AI coding assistant, data flows out of your development environment to external services. Unlike a local linter or compiler, AI tools transmit code context, file contents, and natural language prompts to remote model providers. Understanding and controlling this data flow is essential for:

- **Regulatory compliance** (GDPR, HIPAA, PCI-DSS, SOC 2)
- **Contractual obligations** (NDAs, customer data agreements)
- **Competitive advantage** (protecting trade secrets and proprietary algorithms)
- **Customer trust** (safeguarding personal and sensitive data)
- **Legal protection** (minimizing liability from data exposure)

---

## 1. What Gets Sent to AI Services

### 1.1 Code Completion (GitHub Copilot)

When you type in your editor, Copilot sends context to generate suggestions:

```
┌─────────────────────────────────────────────────────────┐
│            DATA SENT DURING CODE COMPLETION              │
├─────────────────────────────────────────────────────────┤
│  • Current file content (or a substantial portion)      │
│  • Open neighboring tabs (for additional context)       │
│  • File path and name                                   │
│  • Language identifier                                  │
│  • Cursor position                                      │
│  • Repository metadata (for Copilot Enterprise)         │
│  • Recent edits in the current session                  │
└─────────────────────────────────────────────────────────┘
```

✅ **Good to know:** GitHub Copilot Business and Enterprise do **not** use your code to train models. Individual plans may contribute telemetry unless opted out.

❌ **Risk:** If a file containing secrets is open in your editor, that content may be sent as context even if you're editing a different file.

### 1.2 Chat Assistants (Copilot Chat, Claude Code)

Chat interactions send more explicit data:

| Data Type | Sent? | Notes |
|-----------|-------|-------|
| Your prompt/question | ✅ Always | The full text of what you type |
| Selected code | ✅ When shared | Code you highlight or reference |
| Current file | ✅ Usually | As context for the conversation |
| Workspace files | ⚠️ Sometimes | When using @workspace or file references |
| Conversation history | ✅ Full session | All previous messages in the thread |
| Terminal output | ⚠️ Sometimes | When sharing error messages or logs |
| Git diff | ⚠️ Sometimes | When asking about changes |

### 1.3 Code Review AI

AI-powered code review tools typically receive:

- The full diff being reviewed
- File contents for changed files
- PR description and comments
- Repository metadata and file structure

---

## 2. Risk Areas

### 2.1 PII in Code and Comments

Personal Identifiable Information often hides in unexpected places:

❌ **Code comments with real data:**
```python
# Fix for Jane Smith's account (jane.smith@company.com)
# Her user ID is 847291 and she reported this on 2024-01-15
def fix_billing_calculation(user_id: int) -> Decimal:
    ...
```

✅ **Anonymized comments:**
```python
# Fix for billing calculation bug reported in JIRA-4521
# Affected accounts with quarterly billing cycle
def fix_billing_calculation(user_id: int) -> Decimal:
    ...
```

❌ **Real data in variable assignments:**
```javascript
// Test data — but it's real customer data!
const testUser = {
  name: "Sarah Johnson",
  email: "sarah.j@realclient.com",
  phone: "+1-555-0123",
  address: "123 Main St, Springfield, IL 62704"
};
```

✅ **Synthetic test data:**
```javascript
const testUser = {
  name: "Test User Alpha",
  email: "alpha@test.example.com",
  phone: "+1-555-0100",
  address: "100 Test Avenue, Testville, TS 00000"
};
```

### 2.2 API Keys and Secrets

Secrets in code are a risk even without AI tools, but AI amplifies the exposure:

❌ **Hardcoded secrets:**
```python
# This will be sent to the AI service as file context
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"
DATABASE_URL = "postgresql://admin:p4ssw0rd@prod-db.internal:5432/main"
STRIPE_SECRET_KEY = "sk_live_abc123..."
```

✅ **Environment variable references:**
```python
import os

AWS_ACCESS_KEY = os.environ["AWS_ACCESS_KEY_ID"]
DATABASE_URL = os.environ["DATABASE_URL"]
STRIPE_SECRET_KEY = os.environ["STRIPE_SECRET_KEY"]
```

### 2.3 Proprietary Algorithms

Trade secrets and proprietary logic deserve special consideration:

❌ **Exposing proprietary scoring algorithm:**
```python
def calculate_risk_score(applicant: dict) -> float:
    """Our proprietary risk scoring model — competitive advantage.
    
    This algorithm took 2 years to develop and is our key differentiator.
    Weights are trade secrets.
    """
    base_score = applicant["credit_score"] * 0.35
    income_factor = min(applicant["income"] / 150000, 1.0) * 0.25
    # ... 200 more lines of proprietary logic
```

✅ **If you must work with proprietary code, isolate it:**
```python
# .copilotignore — exclude proprietary algorithm files
src/scoring/proprietary_model.py
src/scoring/weights.json
src/pricing/dynamic_pricing_engine.py
```

### 2.4 Customer Data in Test Fixtures

Test fixtures and seed data frequently contain real or realistic data:

❌ **Database fixtures with real data:**
```sql
-- seed_data.sql — often copied from production
INSERT INTO customers (name, email, phone, ssn) VALUES
  ('Robert Chen', 'robert.chen@example.com', '555-0142', '321-54-9876'),
  ('Maria Garcia', 'maria.g@clientcorp.com', '555-0198', '654-32-1098');
```

✅ **Generated synthetic fixtures:**
```sql
-- seed_data.sql — generated with faker
INSERT INTO customers (name, email, phone, tax_id) VALUES
  ('Test Customer 001', 'tc001@test.example.com', '555-0001', 'XXX-XX-0001'),
  ('Test Customer 002', 'tc002@test.example.com', '555-0002', 'XXX-XX-0002');
```

### 2.5 Internal URLs and Infrastructure

Infrastructure details reveal your attack surface:

❌ **Internal infrastructure in code:**
```yaml
# docker-compose with real internal references
services:
  api:
    environment:
      - AUTH_SERVICE=https://auth.internal.company.com:8443
      - VAULT_ADDR=https://vault.prod.company.com:8200
      - SPLUNK_URL=https://splunk.corp.company.com:8089
      - JENKINS_URL=https://ci.internal.company.com/
```

✅ **Abstracted infrastructure references:**
```yaml
services:
  api:
    environment:
      - AUTH_SERVICE=${AUTH_SERVICE_URL}
      - VAULT_ADDR=${VAULT_ADDR}
      - SPLUNK_URL=${SPLUNK_URL}
      - JENKINS_URL=${CI_SERVER_URL}
```

---

## 3. Protection Layers

### 3.1 .copilotignore

The `.copilotignore` file prevents Copilot from reading specified files as context:

```gitignore
# .copilotignore — place in repository root

# Secrets and credentials
.env
.env.*
**/secrets/
**/credentials/
**/*.pem
**/*.key

# Proprietary algorithms
src/scoring/proprietary_*.py
src/pricing/engine_*.py

# Customer data fixtures
tests/fixtures/production_*.json
tests/fixtures/customer_*.sql
seeds/real_data/

# Infrastructure configuration
deploy/production/
k8s/production/
terraform/*.tfvars

# Compliance-sensitive files
docs/compliance/
docs/security-audit/
```

✅ **Best practice:** Add `.copilotignore` to every repository as part of initial setup.

❌ **Common mistake:** Assuming `.gitignore` protects you — `.gitignore` prevents files from being committed, but if they exist locally, AI tools can still read them as context.

### 3.2 Secret Scanning

Enable automated secret detection at multiple levels:

```yaml
# GitHub secret scanning — enable in repository settings
# Also consider pre-commit hooks:

# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks

# Or use GitHub's built-in secret scanning:
# Settings → Code security → Secret scanning → Enable
```

**Secret scanning integration points:**

| Layer | Tool | When It Runs |
|-------|------|-------------|
| IDE | GitHub Secret Scanning (push protection) | Before push |
| Pre-commit | gitleaks, detect-secrets | Before commit |
| CI/CD | GitHub Advanced Security | On push/PR |
| Repository | GitHub Secret Scanning | Continuous |
| Organization | Secret scanning alerts | Continuous |

### 3.3 Data Classification

Implement a clear data classification system:

```
┌──────────────────────────────────────────────────────────────────┐
│                    DATA CLASSIFICATION LEVELS                     │
├───────────────┬──────────────────────────────────────────────────┤
│  🟢 PUBLIC    │ Open-source code, public docs, marketing copy   │
│               │ → Safe for any AI tool                          │
├───────────────┼──────────────────────────────────────────────────┤
│  🔵 INTERNAL  │ Internal code, architecture docs, internal APIs │
│               │ → Tier 1 approved tools only                    │
├───────────────┼──────────────────────────────────────────────────┤
│  🟡 CONFID.   │ Proprietary algorithms, customer contracts,     │
│               │ unreleased features, financial data              │
│               │ → Tier 1 tools with extra caution               │
├───────────────┼──────────────────────────────────────────────────┤
│  🔴 RESTRICT. │ PII, PHI, payment data, credentials,            │
│               │ trade secrets, security audit results            │
│               │ → Never send to external AI tools               │
└───────────────┴──────────────────────────────────────────────────┘
```

### 3.4 Approved Model Providers

Not all AI providers handle data the same way:

| Provider | Data Retention | Training Opt-Out | Enterprise Agreement | SOC 2 |
|----------|---------------|-------------------|---------------------|-------|
| GitHub Copilot (Business) | Not retained for training | Default opt-out | Yes | Yes |
| GitHub Copilot (Enterprise) | Not retained for training | Default opt-out | Yes | Yes |
| Anthropic (Claude API) | Not used for training (API) | API default | Yes | Yes |
| OpenAI (API) | Not used for training (API) | API default | Yes | Yes |
| OpenAI (ChatGPT Free) | May be used for training | Opt-out available | No | No |
| Google (Gemini API) | Varies by tier | Varies | Yes | Yes |

---

## 4. GitHub Copilot Data Handling

### 4.1 Code Suggestions vs. Telemetry

**Code Suggestions (Completions):**
- Code context is sent to generate suggestions
- **Business/Enterprise:** Prompts and suggestions are **not retained** after suggestion is returned
- **Individual:** May retain prompts for service improvement (check current terms)

**Telemetry:**
- Usage metadata (accepted/rejected, latency, language)
- **Business/Enterprise:** Telemetry can be disabled by organization admins
- **Individual:** Basic telemetry collected; detailed telemetry can be opted out

### 4.2 Business vs. Individual Plans

```
┌─────────────────────────────────────────────────────────────┐
│  COPILOT BUSINESS / ENTERPRISE                              │
├─────────────────────────────────────────────────────────────┤
│  ✅ Code not used for training other users' models          │
│  ✅ Prompts not retained beyond request processing          │
│  ✅ Organization-level policy controls                      │
│  ✅ Audit logs available                                    │
│  ✅ Content exclusion (repository/organization level)       │
│  ✅ IP indemnification (Enterprise)                         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  COPILOT INDIVIDUAL                                         │
├─────────────────────────────────────────────────────────────┤
│  ⚠️  May retain code snippets for model improvement         │
│  ⚠️  Telemetry sharing enabled by default                   │
│  ❌ No organization-level controls                          │
│  ❌ No audit logs                                           │
│  ✅ Can opt out of code snippet collection in settings      │
└─────────────────────────────────────────────────────────────┘
```

✅ **Recommendation:** Always use Business or Enterprise plans for organizational work.

❌ **Risk:** Using a personal Copilot subscription on company code without verifying data handling settings.

### 4.3 Configuring Copilot Privacy Settings

```
# Organization-level settings (admin):
# GitHub → Organization → Settings → Copilot → Policies

# Recommended settings for organizational use:
# ✅ "Suggestions matching public code" → Block
# ✅ "Allow Copilot to access repositories" → Select specific repos
# ✅ "Copilot Chat in IDE" → Enabled for approved users
# ✅ "Copilot in CLI" → Enabled for approved users
```

---

## 5. Claude Code Data Handling

### 5.1 Conversation Data

When using Claude Code (Anthropic):

- **API usage:** Conversations are **not used for training** by default
- **Prompts and responses** are processed for the request and subject to Anthropic's data retention policy
- **Enterprise plans** offer additional data handling controls and shorter retention windows
- **File context** shared with Claude Code is sent as part of the conversation

### 5.2 Model Training Opt-Out

```
┌─────────────────────────────────────────────────────────────┐
│  CLAUDE CODE (API / Enterprise)                             │
├─────────────────────────────────────────────────────────────┤
│  ✅ API inputs not used for model training (default)        │
│  ✅ Enterprise: additional data controls available           │
│  ✅ Enterprise: custom data retention policies               │
│  ⚠️  Check current terms — policies evolve                   │
│  ⚠️  Conversation data retained for safety and abuse review  │
└─────────────────────────────────────────────────────────────┘
```

### 5.3 Practical Considerations for Claude Code

✅ **Do:**
```bash
# Use Claude Code with organizational API keys
# Configure ANTHROPIC_API_KEY as an organization-managed secret
export ANTHROPIC_API_KEY="sk-ant-..." # From organization key manager

# Use .claudeignore (if available) or .copilotignore patterns
# to exclude sensitive files from context
```

❌ **Don't:**
```bash
# Don't use personal API keys for organizational work
# Don't share conversation URLs containing proprietary code
# Don't paste production logs with customer data into prompts
```

---

## 6. Data Classification Matrix

Use this matrix to quickly determine what you can share with AI tools:

| Data Type | Classification | AI Tool Usage | Protection Required |
|-----------|---------------|---------------|-------------------|
| Public API schemas | 🟢 Public | Any tool | None |
| Internal library code | 🔵 Internal | Tier 1 tools | Standard |
| Business logic | 🔵 Internal | Tier 1 tools | Standard |
| Customer names/emails | 🔴 Restricted | Never | .copilotignore + scanning |
| API keys/tokens | 🔴 Restricted | Never | Secret scanning + .env |
| Health records (PHI) | 🔴 Restricted | Never | HIPAA controls |
| Payment card data | 🔴 Restricted | Never | PCI-DSS controls |
| Proprietary algorithms | 🟡 Confidential | Tier 1 with caution | .copilotignore |
| Architecture diagrams | 🔵 Internal | Tier 1 tools | Standard |
| Security audit reports | 🔴 Restricted | Never | Access controls |
| Test fixtures (synthetic) | 🟢 Public | Any tool | None |
| Test fixtures (real data) | 🔴 Restricted | Never | Data anonymization |
| Error logs (sanitized) | 🔵 Internal | Tier 1 tools | Log sanitization |
| Error logs (raw) | 🟡 Confidential | Tier 1 with caution | Log sanitization |
| Open-source dependencies | 🟢 Public | Any tool | None |
| Internal hostnames/IPs | 🟡 Confidential | Tier 1 with caution | Environment variables |
| Compliance documentation | 🟡 Confidential | Tier 1 with caution | Access controls |

---

## 7. Protection Strategies Summary

### Quick-Reference Decision Tree

```
Is the data you want to share with AI tools...

  Contains credentials or secrets?
    → ❌ STOP. Remove secrets, use environment variables.

  Contains PII (names, emails, SSN, etc.)?
    → ❌ STOP. Anonymize or use synthetic data.

  Contains PHI (health information)?
    → ❌ STOP. HIPAA prohibits this.

  Contains payment card data?
    → ❌ STOP. PCI-DSS prohibits this.

  Is it a proprietary/trade secret algorithm?
    → ⚠️  Use .copilotignore to exclude. Ask AI about patterns, not specifics.

  Is it internal but non-sensitive code?
    → ✅ Use Tier 1 approved tools with organizational accounts.

  Is it public/open-source code?
    → ✅ Use any approved tool.
```

### Layered Defense Checklist

- [ ] **Layer 1: Prevention** — `.copilotignore` configured in all repositories
- [ ] **Layer 2: Detection** — Secret scanning enabled (pre-commit + CI/CD + repository)
- [ ] **Layer 3: Classification** — Data classification labels applied to repositories and directories
- [ ] **Layer 4: Access Control** — AI tool access limited to approved personnel
- [ ] **Layer 5: Monitoring** — Audit logs enabled for AI tool usage
- [ ] **Layer 6: Response** — Incident response plan includes AI data exposure scenarios
- [ ] **Layer 7: Training** — All developers trained on data classification and AI privacy
