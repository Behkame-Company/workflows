# AI Usage Policy

> A comprehensive, ready-to-customize policy framework governing the use of AI-assisted development tools across your organization. This document establishes clear boundaries, responsibilities, and processes for teams adopting AI coding assistants like GitHub Copilot, Claude Code, and other generative AI tools.

---

## Why You Need an AI Usage Policy

AI coding tools are transforming software development, but without clear policies, teams face risks around data exposure, code quality, intellectual property, and regulatory compliance. A well-crafted AI usage policy:

- **Reduces risk** by establishing clear boundaries for AI tool usage
- **Accelerates adoption** by removing uncertainty about what's allowed
- **Ensures consistency** across teams and projects
- **Satisfies compliance** requirements for audits and regulations
- **Protects the organization** from legal and security exposure

---

## 1. Scope

### 1.1 Covered Tools

This policy applies to all AI-assisted development tools, including but not limited to:

| Category | Tools | Policy Status |
|----------|-------|---------------|
| Code completion | GitHub Copilot, Tabnine, Amazon CodeWhisperer | Covered |
| AI chat assistants | Claude Code, ChatGPT, Gemini | Covered |
| Code review AI | Copilot code review, CodeRabbit, Sourcery | Covered |
| AI-powered IDEs | Cursor, Windsurf, Cody | Covered |
| Custom/internal AI | Internal fine-tuned models, RAG pipelines | Covered |
| General AI chatbots | Web-based ChatGPT, Claude.ai for code questions | Covered |

### 1.2 Covered Projects

This policy applies to:

- ✅ All production codebases
- ✅ Internal tools and utilities
- ✅ Open-source projects maintained by the organization
- ✅ Proof-of-concept and prototype work
- ✅ Documentation and technical writing
- ✅ Configuration and infrastructure-as-code
- ❌ Personal projects on personal time (outside organizational scope)
- ❌ Learning and training exercises using public/synthetic data only

### 1.3 Covered Personnel

- All full-time and part-time engineering staff
- Contractors and consultants with codebase access
- Technical leads and engineering managers using AI tools
- DevOps and SRE teams using AI for infrastructure
- QA engineers using AI for test generation

---

## 2. Approved Tools

### 2.1 Tier 1 — Approved for General Use

These tools have been vetted and are approved for use across all non-restricted projects:

```
┌─────────────────────────────────────────────────────────┐
│  TIER 1: General Use — No Additional Approval Needed    │
├─────────────────────────────────────────────────────────┤
│  • GitHub Copilot (Business/Enterprise)                 │
│  • GitHub Copilot Chat                                  │
│  • Claude Code (with organizational API key)            │
│  • GitHub Copilot Code Review                           │
└─────────────────────────────────────────────────────────┘
```

**Requirements for Tier 1 tools:**
- Must use organizational accounts (not personal)
- Must have telemetry/training opt-out enabled where available
- Must be configured per organizational standards

### 2.2 Tier 2 — Approved with Restrictions

These tools are approved for specific use cases with additional safeguards:

| Tool | Allowed Use Cases | Restrictions |
|------|-------------------|--------------|
| ChatGPT (Team/Enterprise) | General coding questions, algorithm design | No proprietary code in prompts |
| Amazon CodeWhisperer | AWS-specific development | AWS projects only |
| Tabnine (Enterprise) | Code completion | On-premise deployment required |
| Cursor / Windsurf | Development IDE | Must use approved model backends |

### 2.3 Tier 3 — Requires Explicit Approval

These tools require written approval from engineering leadership before use:

- Custom fine-tuned models on organizational data
- Self-hosted open-source models (LLaMA, CodeLlama, etc.)
- New or untested AI tools not yet evaluated
- AI tools from vendors without enterprise agreements

### 2.4 Not Approved

- ❌ Free-tier AI services without enterprise data agreements
- ❌ AI tools that store/train on input data without opt-out
- ❌ Browser extensions from unverified sources claiming AI features
- ❌ AI tools requiring upload of full repositories to third-party servers

---

## 3. Prohibited Uses

### 3.1 Security-Critical Code

AI tools **must not** be the primary author of security-critical components without expert human review:

❌ **Never do this:**
```python
# Don't ask AI to generate cryptographic implementations
def custom_encrypt(data, key):
    # AI-generated AES implementation
    # This could have subtle vulnerabilities
    ...
```

✅ **Do this instead:**
```python
# Use well-established libraries; AI can help with correct usage
from cryptography.fernet import Fernet

def encrypt_data(data: bytes, key: bytes) -> bytes:
    """Encrypt data using Fernet symmetric encryption.
    AI assisted with API usage — verified against library docs."""
    f = Fernet(key)
    return f.encrypt(data)
```

**Specifically prohibited without expert security review:**
- Custom cryptographic algorithms or protocols
- Authentication and authorization logic (core flows)
- Token generation, validation, or session management
- Certificate handling and TLS configuration
- Security middleware and access control enforcement

### 3.2 Personal and Sensitive Data in Prompts

❌ **Never include in AI prompts:**
```
# BAD: Real customer data in prompt
"Fix this query that returns wrong results for customer 
John Smith (john.smith@example.com, SSN: 123-45-6789)"
```

✅ **Use synthetic data instead:**
```
# GOOD: Anonymized/synthetic data in prompt
"Fix this query that returns wrong results for a customer 
record like {name: 'Test User', email: 'test@example.com', id: 12345}"
```

### 3.3 Accepting Code Without Review

Every piece of AI-generated code **must** be reviewed before committing:

❌ **Never do this:**
```bash
# Blindly accepting all Copilot suggestions
# Tab-Tab-Tab-Tab without reading
```

✅ **Always do this:**
```bash
# Review each suggestion for:
# 1. Correctness — Does it do what you intend?
# 2. Security — Are there injection points, exposed data, or vulnerabilities?
# 3. Performance — Are there unnecessary allocations, N+1 queries?
# 4. Style — Does it match team conventions?
# 5. Dependencies — Does it import unexpected libraries?
```

### 3.4 Additional Prohibited Uses

| Prohibited Use | Reason | Alternative |
|----------------|--------|-------------|
| Generating fake test data with real patterns | PII risk | Use faker libraries with seeds |
| Asking AI to bypass security controls | Policy violation | File a security exception request |
| Using AI to write compliance documentation you haven't verified | Audit risk | Draft with AI, verify every claim |
| Pasting error logs containing user data into AI | Data exposure | Sanitize logs before sharing |
| Using AI to reverse-engineer proprietary third-party code | Legal risk | Read official documentation |

---

## 4. Data Handling

### 4.1 Data Classification for AI Tool Usage

```
┌──────────────────────────────────────────────────────────────┐
│                    DATA CLASSIFICATION MATRIX                │
├──────────────┬───────────────┬────────────────┬──────────────┤
│  Category    │  Tier 1 Tools │  Tier 2 Tools  │  Tier 3 Tools│
├──────────────┼───────────────┼────────────────┼──────────────┤
│  Public      │  ✅ Allowed   │  ✅ Allowed    │  ✅ Allowed  │
│  Internal    │  ✅ Allowed   │  ⚠️  Caution   │  ❌ No       │
│  Confidential│  ⚠️  Caution  │  ❌ No         │  ❌ No       │
│  Restricted  │  ❌ No        │  ❌ No         │  ❌ No       │
└──────────────┴───────────────┴────────────────┴──────────────┘
```

### 4.2 What CAN Be Sent to AI Tools

✅ **Acceptable to include in prompts and context:**

- Open-source code and public documentation references
- General coding patterns and algorithm questions
- Code you've written that contains no sensitive data
- Error messages with sensitive information redacted
- Architecture questions using generic terms
- Test code using synthetic/fake data
- Configuration templates with placeholder values

### 4.3 What CANNOT Be Sent to AI Tools

❌ **Never include in prompts or allow in AI context:**

- API keys, tokens, passwords, or any credentials
- Customer PII (names, emails, addresses, phone numbers, SSNs)
- Payment card data (PCI-DSS scope)
- Protected health information (HIPAA scope)
- Internal infrastructure details (IP addresses, hostnames, network topology)
- Proprietary algorithms that constitute trade secrets
- Data subject to NDA or contractual restrictions
- Unreleased product features or roadmap details

### 4.4 Practical Data Handling Examples

❌ **Bad — Secret in context:**
```yaml
# Don't let AI see this file as context
database:
  host: prod-db-east.internal.company.com
  password: SuperSecret123!
  connection_string: postgresql://admin:SuperSecret123!@prod-db-east...
```

✅ **Good — Use environment variable references:**
```yaml
# Safe for AI to see as context
database:
  host: ${DB_HOST}
  password: ${DB_PASSWORD}
  connection_string: ${DATABASE_URL}
```

❌ **Bad — Real customer data in test fixtures:**
```json
{
  "customers": [
    {"name": "Jane Doe", "email": "jane.doe@realcompany.com", "ssn": "123-45-6789"}
  ]
}
```

✅ **Good — Synthetic test fixtures:**
```json
{
  "customers": [
    {"name": "Test User Alpha", "email": "alpha@test.example.com", "id": "test-001"}
  ]
}
```

---

## 5. Attribution and Disclosure

### 5.1 When to Disclose AI Use

| Scenario | Disclosure Required? | How |
|----------|---------------------|-----|
| AI autocompleted a few lines | No | Standard workflow |
| AI generated a significant function/module | Yes | Code comment or PR description |
| AI designed an architecture approach | Yes | Design document notation |
| AI wrote initial test cases | Yes | PR description |
| AI helped debug an issue | No | Standard workflow |
| AI generated documentation | Yes | Document footer notation |
| AI wrote commit messages | No | Standard workflow |

### 5.2 Attribution Format

When disclosure is warranted, use a consistent format:

```python
# AI-assisted: Initial implementation generated with GitHub Copilot.
# Reviewed and modified by [developer] on [date].
def complex_algorithm(data: list[int]) -> list[int]:
    ...
```

In pull request descriptions:

```markdown
## AI Assistance Disclosure
- **Tool:** GitHub Copilot / Claude Code
- **Scope:** AI assisted with [initial scaffolding / test generation / algorithm implementation]
- **Review:** All generated code was reviewed, tested, and modified as needed
```

### 5.3 Open-Source Contributions

When contributing AI-assisted code to open-source projects:

✅ **Do:**
- Check the project's policy on AI-generated contributions
- Disclose AI assistance in the PR description if the project requires it
- Ensure generated code doesn't introduce license conflicts
- Use Copilot's code reference feature to check for matches

❌ **Don't:**
- Submit large blocks of unmodified AI output to license-restricted projects
- Claim sole authorship of substantially AI-generated contributions if the project policy requires disclosure
- Ignore project-specific AI contribution policies

---

## 6. Accountability

### 6.1 Core Principle

> **The developer who commits code is responsible for that code, regardless of whether it was written by hand, generated by AI, or copied from any source.**

This means:

- **You own the bugs.** If AI-generated code introduces a defect, the committing developer is responsible for the fix.
- **You own the security.** If AI-generated code creates a vulnerability, the committing developer is accountable.
- **You own the quality.** If AI-generated code doesn't meet standards, the committing developer must bring it up to standard.
- **You own the understanding.** You must understand what the code does — "the AI wrote it" is not an acceptable explanation in code review.

### 6.2 Accountability Matrix

| Role | Responsibilities |
|------|------------------|
| **Individual Developer** | Review all AI output before committing; understand every line; ensure compliance with this policy |
| **Code Reviewer** | Apply same review standards to AI-assisted code; flag potential AI-generated issues; verify the author understands the code |
| **Tech Lead** | Ensure team follows AI usage policy; review AI tool configurations; escalate policy violations |
| **Engineering Manager** | Monitor AI tool adoption and compliance; ensure training is completed; report on policy adherence |
| **Security Team** | Audit AI tool data flows; review AI-related incidents; update policy based on emerging threats |

### 6.3 What "Understanding Your Code" Means

✅ **Acceptable — Developer understands the code:**
```
Reviewer: "Why did you use a trie here instead of a hash map?"
Developer: "The trie gives us prefix matching in O(m) time where m is 
the key length. Copilot suggested it, and I evaluated it against our 
use case — we need prefix search for the autocomplete feature, so a 
trie is the right choice."
```

❌ **Not acceptable — Developer doesn't understand the code:**
```
Reviewer: "Why did you use a trie here instead of a hash map?"
Developer: "I'm not sure, Copilot suggested it and it passed the tests."
```

---

## 7. Exceptions Process

### 7.1 When to Request an Exception

Exceptions are needed when:

- Using a tool not on the approved list
- Working with restricted data that needs AI assistance
- A project has unique requirements not covered by this policy
- A new AI tool needs evaluation for potential adoption
- Security-critical code benefits from AI assistance with additional review

### 7.2 Exception Request Process

```
┌─────────────────────────────────────────────────────────────┐
│                   EXCEPTION REQUEST FLOW                    │
│                                                             │
│  1. Developer submits exception request form                │
│     ↓                                                       │
│  2. Tech lead reviews and adds context                      │
│     ↓                                                       │
│  3. Security team assesses risk (if data/security related)  │
│     ↓                                                       │
│  4. Engineering leadership approves or denies               │
│     ↓                                                       │
│  5. If approved: document scope, duration, and conditions   │
│     ↓                                                       │
│  6. Review exception at expiration (default: 90 days)       │
└─────────────────────────────────────────────────────────────┘
```

### 7.3 Exception Request Template

```markdown
## AI Usage Policy Exception Request

**Requester:** [Name]
**Date:** [Date]
**Team:** [Team name]

### What exception is needed?
[Describe the specific policy section and what you need to do differently]

### Why is this exception needed?
[Business justification]

### What is the scope?
- **Project(s):** [Specific repos/projects]
- **Duration:** [Requested timeframe]
- **Team members:** [Who needs this exception]

### Risk assessment
- **Data sensitivity:** [What data types are involved]
- **Security impact:** [Potential security implications]
- **Mitigation:** [What additional controls will be in place]

### Approval
- [ ] Tech Lead: _______________
- [ ] Security Team: _______________
- [ ] Engineering Leadership: _______________
```

---

## Complete AI Usage Policy Template

Below is a complete, customizable policy template your organization can adopt:

```markdown
# [Organization Name] AI-Assisted Development Policy

**Version:** 1.0
**Effective Date:** [Date]
**Last Reviewed:** [Date]
**Owner:** [Engineering Leadership / CTO]
**Next Review:** [Date + 6 months]

## 1. Purpose
This policy establishes guidelines for the responsible use of AI-assisted 
development tools within [Organization Name].

## 2. Scope
This policy applies to all personnel who write, review, or deploy code 
for [Organization Name] projects.

## 3. Approved Tools
[List your approved tools by tier — see Section 2 above]

## 4. Permitted Uses
- Code completion and suggestion
- Code explanation and documentation
- Test generation (with review)
- Debugging assistance
- Refactoring suggestions
- Architecture discussion (without sensitive details)

## 5. Prohibited Uses
- Generating security-critical cryptographic code
- Including PII, credentials, or restricted data in prompts
- Accepting generated code without review
- Using unapproved AI tools on company projects
- Bypassing security controls with AI assistance

## 6. Data Handling
- Classify data before including in AI context
- Never send restricted or confidential data to AI tools
- Use .copilotignore and secret scanning
- Sanitize logs and error messages before sharing

## 7. Attribution
- Disclose significant AI contributions in PRs
- Follow project-specific disclosure requirements
- Maintain consistent attribution format

## 8. Accountability
- Developers are responsible for all committed code
- Code review standards apply equally to AI-generated code
- "The AI wrote it" is not an acceptable justification

## 9. Training
- All developers must complete AI tool usage training
- Annual refresher training is required
- New tool onboarding includes policy review

## 10. Compliance
- This policy will be reviewed every 6 months
- Violations are handled through standard HR processes
- Audit trails must be maintained per compliance requirements

## 11. Exceptions
- Follow the exception request process
- Exceptions expire after 90 days by default
- All exceptions must be documented

## 12. Contact
- Policy questions: [email/channel]
- Exception requests: [email/channel]
- Security concerns: [email/channel]
```

---

## Policy Enforcement Checklist

Use this checklist for periodic compliance reviews:

- [ ] All team members have acknowledged the AI usage policy
- [ ] Approved tool list is current and reflects actual usage
- [ ] `.copilotignore` files are configured in all repositories
- [ ] Secret scanning is enabled and functioning
- [ ] Code review processes explicitly address AI-generated code
- [ ] Exception requests are documented and within expiration dates
- [ ] Incident response plan includes AI-related scenarios
- [ ] Training records are up to date
- [ ] Data classification guidelines are applied to AI contexts
- [ ] Audit logs are being collected per compliance requirements
