# Compliance and Audit for AI Development

> A comprehensive guide to meeting regulatory and organizational compliance requirements when using AI-assisted development tools. This document covers audit trails, compliance framework mappings, automated enforcement, and practical checklists for maintaining compliance in AI-augmented development workflows.

---

## Why Compliance Matters for AI Development

AI coding tools introduce new compliance dimensions that traditional development didn't face. Code is now being co-authored with external AI services, which means:

- **Data leaves your environment** — Prompts and code context are processed by third-party providers
- **New third-party dependencies** — AI services become part of your supply chain
- **Audit trail gaps** — Traditional logging doesn't capture AI interactions
- **Regulatory uncertainty** — Existing frameworks are being reinterpreted for AI usage
- **Shared responsibility** — AI providers and your organization both have compliance obligations

Organizations in regulated industries must proactively address these dimensions to maintain compliance certifications and pass audits.

---

## 1. Audit Trails

### 1.1 What to Log

Effective audit trails for AI-assisted development should capture:

| Data Point | Why It Matters | Storage |
|------------|---------------|---------|
| Tool used (Copilot, Claude Code, etc.) | Identifies third-party service involvement | Audit log |
| Model version (if available) | Reproducibility and accountability | Audit log |
| Timestamp | Chronological record | Audit log |
| User identity | Accountability and access control verification | Audit log |
| Action type (completion, chat, review) | Understanding scope of AI involvement | Audit log |
| Repository / project context | Scope of data exposure | Audit log |
| Acceptance/rejection of suggestions | Usage patterns and human oversight evidence | Telemetry |
| Significant AI contributions noted in PRs | Human review evidence | Git / PR system |

### 1.2 What NOT to Log

Equally important is knowing what should **not** be captured in audit logs:

❌ **Do not log:**
- Full prompt contents (may contain sensitive code or data)
- Complete AI responses (may contain generated code with embedded context)
- Code snippets in plain text in centralized logging systems
- Personal notes or conversations about specific bugs or customers
- Screenshots of AI interactions containing proprietary code

✅ **Instead, log metadata:**
```json
{
  "event": "ai_tool_interaction",
  "timestamp": "2024-11-15T14:23:01Z",
  "user": "developer@company.com",
  "tool": "github-copilot",
  "action": "chat_completion",
  "repository": "org/backend-api",
  "session_id": "abc-123-def",
  "data_classification": "internal",
  "suggestion_accepted": true,
  "review_completed": true
}
```

❌ **Not this:**
```json
{
  "event": "ai_tool_interaction",
  "prompt": "Fix the SQL query for customer John Smith account #847291 that returns wrong billing amount...",
  "response": "Here's the corrected query: SELECT * FROM billing WHERE customer_id = 847291 AND ...",
  "full_code_context": "... 500 lines of proprietary code ..."
}
```

### 1.3 Audit Log Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   AUDIT LOG ARCHITECTURE                     │
│                                                             │
│  Developer Environment                                      │
│  ├── IDE telemetry (tool-specific) ──┐                      │
│  ├── Git commit metadata ────────────┤                      │
│  └── PR descriptions ────────────────┤                      │
│                                      ▼                      │
│                              Log Aggregator                 │
│                              (SIEM / Logging)               │
│                                      │                      │
│                         ┌────────────┼────────────┐         │
│                         ▼            ▼            ▼         │
│                    Compliance    Security     Management     │
│                    Reports       Alerts       Dashboards    │
│                                                             │
│  AI Tool Provider                                           │
│  ├── Usage analytics ────────────────┐                      │
│  ├── Admin console logs ─────────────┤                      │
│  └── API usage records ──────────────┘                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Compliance Framework Mappings

### 2.1 SOC 2 — AI as a Third-Party Service

SOC 2 (System and Organization Controls 2) treats AI tools as third-party services in your software supply chain:

| SOC 2 Trust Principle | AI Development Impact | Compliance Action |
|----------------------|----------------------|-------------------|
| **Security** | AI tools process your code | Verify provider security certifications; use enterprise plans |
| **Availability** | AI tool outages affect development | Don't create hard dependencies on AI tools for critical paths |
| **Processing Integrity** | AI output may contain errors | Code review processes must catch AI-generated defects |
| **Confidentiality** | Code sent to AI providers | Data classification; .copilotignore; approved tools only |
| **Privacy** | PII may be in code context | Prevent PII from reaching AI tools; monitor and alert |

**SOC 2 Audit Evidence for AI Tool Usage:**

✅ **Evidence to prepare:**
```markdown
## SOC 2 AI Tool Evidence Package

1. VENDOR MANAGEMENT
   - [ ] AI tool vendor security assessments completed
   - [ ] Enterprise agreements with data handling clauses
   - [ ] Vendor SOC 2 reports reviewed (GitHub, Anthropic, etc.)
   - [ ] Data processing agreements (DPAs) in place

2. ACCESS CONTROLS
   - [ ] AI tools provisioned through organizational accounts
   - [ ] Access reviews conducted quarterly
   - [ ] Deprovisioning process for departing employees
   - [ ] MFA enabled for AI tool accounts

3. DATA PROTECTION
   - [ ] .copilotignore configured across repositories
   - [ ] Secret scanning enabled and monitored
   - [ ] Data classification policy applied to AI contexts
   - [ ] Training opt-out configured for all tools

4. MONITORING
   - [ ] AI tool usage logs collected
   - [ ] Anomalous usage patterns alerting configured
   - [ ] Periodic access reviews documented
   - [ ] Incident response plan includes AI scenarios
```

### 2.2 HIPAA — Protected Health Information in Prompts

For organizations handling Protected Health Information (PHI):

```
┌─────────────────────────────────────────────────────────────┐
│                    HIPAA + AI TOOLS                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🔴 CRITICAL: PHI must NEVER be sent to AI tools unless:    │
│     • A BAA (Business Associate Agreement) is in place      │
│       with the AI provider                                  │
│     • The AI tool is explicitly approved for PHI            │
│     • Data is encrypted in transit and at rest              │
│     • Access controls meet HIPAA requirements               │
│                                                             │
│  Most AI coding tools do NOT have BAAs and are NOT          │
│  approved for PHI processing.                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

❌ **HIPAA violation risk:**
```python
# Developer debugging a query with real patient data in prompt:
# "Fix this query — patient John Smith DOB 1985-03-15 
#  MRN 12345 shows wrong medication list"

# Even asking about this in an AI chat sends PHI to the provider
```

✅ **HIPAA-compliant approach:**
```python
# Developer debugging with synthetic data:
# "Fix this query — a patient record with a specific MRN 
#  shows an incorrect medication list. The query joins 
#  patients, prescriptions, and medications tables."

# Use synthetic MRN in test: "TEST-00001"
# Use synthetic names: "Test Patient Alpha"
```

**HIPAA Compliance Checklist for AI Tools:**

- [ ] No AI tool is used with PHI unless a BAA exists
- [ ] Developers are trained on PHI identification in code
- [ ] Code repositories with PHI are excluded from AI tool context
- [ ] Test fixtures use synthetic data exclusively
- [ ] Incident response plan covers PHI exposure via AI tools
- [ ] PHI-containing files are listed in `.copilotignore`
- [ ] Audit logs track AI tool access to PHI-adjacent repositories

### 2.3 GDPR — Personal Data Processing

The General Data Protection Regulation applies when AI tools process personal data of EU individuals:

| GDPR Requirement | AI Development Impact | Compliance Action |
|------------------|----------------------|-------------------|
| **Lawful basis** | Sending personal data to AI providers | Establish legitimate interest or get consent |
| **Data minimization** | AI context includes more than needed | Configure context limits; use .copilotignore |
| **Purpose limitation** | Data sent for coding assistance | Ensure AI providers don't use data for other purposes |
| **Storage limitation** | AI providers may retain data | Verify retention policies; use enterprise plans |
| **Data subject rights** | Right to erasure applies | Ensure ability to request data deletion from AI providers |
| **Data protection impact** | AI tools are new processing activity | Conduct DPIA for AI tool adoption |
| **Transfer mechanisms** | Data may cross borders | Verify AI providers' data transfer mechanisms (SCCs, etc.) |

✅ **GDPR compliance approach:**
```markdown
## Data Protection Impact Assessment (DPIA) — AI Development Tools

### Processing Activity
Use of AI-assisted development tools (GitHub Copilot, Claude Code) 
for software development.

### Personal Data Involved
- Code may contain personal data (variable names, comments, test data)
- Developer identities in usage logs

### Risks Identified
1. Personal data in code sent to AI providers
2. AI provider data retention beyond necessity
3. Cross-border data transfer to AI provider infrastructure

### Mitigations
1. .copilotignore excludes files likely to contain personal data
2. Enterprise plans with no-training and minimal retention policies
3. AI providers with EU data residency or adequate transfer mechanisms
4. Developer training on personal data in code
5. Regular audits of AI tool data flows
```

### 2.4 PCI-DSS — Payment Card Data

For organizations handling payment card data:

❌ **Never in AI context:**
```python
# NEVER include payment data in AI tool context
CARD_NUMBER = "4111-1111-1111-1111"
CVV = "123"
EXPIRY = "12/25"

# Even test card numbers in code that reaches AI tools
# could trigger PCI-DSS scope concerns
```

✅ **PCI-DSS compliant approach:**
```python
# Payment data handled through tokenized references
PAYMENT_TOKEN = os.environ["PAYMENT_TOKEN"]

# Use payment processor SDKs — never handle raw card data
stripe.Charge.create(
    amount=2000,
    currency="usd",
    source=token_from_client,  # Tokenized by Stripe.js
)
```

**PCI-DSS Compliance Requirements for AI Tools:**

- [ ] Cardholder data environment (CDE) code is excluded from AI context
- [ ] `.copilotignore` covers all payment processing code
- [ ] No payment card data appears in test fixtures accessible to AI tools
- [ ] AI tools are not used within the CDE network segment
- [ ] Penetration test scope includes AI tool data flows
- [ ] QSA (Qualified Security Assessor) has evaluated AI tool usage

### 2.5 ISO 27001 — Information Security Management

ISO 27001 requires systematic management of information security risks, including AI tools:

| ISO 27001 Control | AI Tool Application |
|-------------------|-------------------|
| A.5 — Information Security Policies | AI usage policy documented and communicated |
| A.6 — Organization of Information Security | AI tool responsibilities assigned |
| A.8 — Asset Management | AI tools listed in asset inventory |
| A.9 — Access Control | AI tool access follows least privilege |
| A.12 — Operations Security | AI tool monitoring and logging |
| A.13 — Communications Security | AI tool data transmission encrypted |
| A.14 — System Development Security | AI in SDLC documented and controlled |
| A.15 — Supplier Relationships | AI providers assessed as suppliers |
| A.18 — Compliance | AI tool compliance requirements documented |

---

## 3. Audit Checklist

### 3.1 Comprehensive AI Development Audit Checklist

Use this checklist for periodic compliance reviews:

#### Tool Governance
- [ ] All AI tools in use are on the approved tools list
- [ ] Tool versions and configurations are documented
- [ ] Enterprise agreements are in place for all tools
- [ ] No unapproved tools are being used (verify through access logs)
- [ ] Tool configurations match organizational policy settings

#### Access Controls
- [ ] AI tools are provisioned through organizational accounts only
- [ ] Individual (personal) AI accounts are not used for work
- [ ] Access reviews are conducted at least quarterly
- [ ] Departing employees are deprovisioned from AI tools promptly
- [ ] MFA is enabled for all AI tool accounts
- [ ] Role-based access is configured (admin vs. user)

#### Data Handling
- [ ] `.copilotignore` files are in place across all repositories
- [ ] Secret scanning is enabled and actively monitored
- [ ] Data classification policy is applied to AI tool contexts
- [ ] No restricted data is sent to AI tools (verified through reviews)
- [ ] AI tool training/telemetry opt-out is configured
- [ ] Data retention policies of AI providers are documented and acceptable

#### Incident Response
- [ ] Incident response plan includes AI-related scenarios
- [ ] Data exposure via AI tools is a defined incident type
- [ ] Response procedures for AI tool compromise are documented
- [ ] Contact information for AI provider security teams is on file
- [ ] Tabletop exercises have included AI-specific scenarios

#### Training and Awareness
- [ ] All developers have completed AI usage policy training
- [ ] Training records are maintained and current
- [ ] Annual refresher training is scheduled
- [ ] New employee onboarding includes AI policy acknowledgment
- [ ] Security awareness training covers AI-specific risks

#### Monitoring and Logging
- [ ] AI tool usage logs are collected centrally
- [ ] Anomalous usage patterns trigger alerts
- [ ] Log retention meets compliance requirements
- [ ] Logs are protected from tampering
- [ ] Regular log reviews are conducted

---

## 4. Automated Compliance

### 4.1 CI/CD Gates

Implement automated compliance checks in your CI/CD pipeline:

```yaml
# .github/workflows/compliance-checks.yml
name: AI Compliance Checks

on:
  pull_request:
    branches: [main, develop]

jobs:
  compliance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Check for secrets in code
      - name: Secret Scanning
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      # Check for PII patterns in code
      - name: PII Pattern Check
        run: |
          # Check for common PII patterns
          ISSUES=0

          # Social Security Numbers
          if grep -rn '[0-9]\{3\}-[0-9]\{2\}-[0-9]\{4\}' --include="*.py" --include="*.js" --include="*.ts" --include="*.java" src/ tests/; then
            echo "::error::Potential SSN pattern found in code"
            ISSUES=$((ISSUES + 1))
          fi

          # Email addresses in non-test code (basic check)
          if grep -rn '[a-zA-Z0-9._%+-]\+@[a-zA-Z0-9.-]\+\.[a-zA-Z]\{2,\}' --include="*.py" --include="*.js" --include="*.ts" src/ | grep -v 'example\.com' | grep -v 'test\.'; then
            echo "::warning::Potential real email addresses found in source code"
          fi

          if [ $ISSUES -gt 0 ]; then
            exit 1
          fi

      # License compliance check
      - name: License Check
        run: |
          npx license-checker --failOn "GPL-2.0;GPL-3.0;AGPL-3.0" || true

      # Verify .copilotignore exists
      - name: Verify Copilotignore
        run: |
          if [ ! -f .copilotignore ]; then
            echo "::error::.copilotignore file is missing"
            exit 1
          fi
          echo "✅ .copilotignore file present"

      # Check for AI disclosure in PR
      - name: Check AI Disclosure
        if: github.event_name == 'pull_request'
        run: |
          PR_BODY="${{ github.event.pull_request.body }}"
          # Informational only — not a gate
          if echo "$PR_BODY" | grep -qi "ai\|copilot\|claude\|generated"; then
            echo "✅ PR includes AI-related disclosure"
          else
            echo "ℹ️ No AI disclosure found in PR — this is OK if no significant AI assistance was used"
          fi
```

### 4.2 SAST Scanning

Static Application Security Testing should be configured to catch AI-generated vulnerabilities:

```yaml
# Additional SAST checks for AI-generated code patterns
name: SAST for AI Code

on:
  pull_request:
    branches: [main]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # CodeQL analysis — catches common vulnerability patterns
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript, python

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3

      # Additional checks for AI-specific patterns
      - name: AI Code Pattern Checks
        run: |
          echo "Checking for common AI-generated anti-patterns..."

          # Check for hardcoded URLs that might be from training data
          if grep -rn 'https\?://[a-z]*\.example\.' --include="*.py" --include="*.js" --include="*.ts" src/; then
            echo "::warning::Found example.* URLs in source code — verify these are intentional"
          fi

          # Check for TODO/FIXME that AI might have generated
          TODO_COUNT=$(grep -rn 'TODO\|FIXME\|HACK\|XXX' --include="*.py" --include="*.js" --include="*.ts" src/ | wc -l)
          echo "Found $TODO_COUNT TODO/FIXME/HACK/XXX comments — review for AI-generated placeholders"
```

### 4.3 Dependency Auditing

AI-generated code may introduce unexpected dependencies:

```yaml
# Dependency audit workflow
name: Dependency Audit

on:
  pull_request:
    paths:
      - 'package.json'
      - 'package-lock.json'
      - 'requirements.txt'
      - 'Pipfile.lock'
      - 'go.sum'

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Check for new dependencies added by AI suggestions
      - name: Dependency Diff
        run: |
          echo "## New Dependencies Check" >> $GITHUB_STEP_SUMMARY

          # Compare dependency changes
          git diff origin/main -- package.json | grep '^\+.*"dependencies"' -A 50 | head -60
          git diff origin/main -- requirements.txt | grep '^\+' | head -30

          echo "Review any new dependencies — AI tools may suggest unnecessary packages"

      # Vulnerability scanning
      - name: npm audit
        if: hashFiles('package-lock.json') != ''
        run: npm audit --audit-level=high

      - name: pip audit
        if: hashFiles('requirements.txt') != ''
        run: |
          pip install pip-audit
          pip-audit -r requirements.txt
```

---

## 5. Compliance Mapping Matrix

Use this matrix to map AI development activities to compliance requirements:

### Activity-to-Framework Mapping

| AI Development Activity | SOC 2 | HIPAA | GDPR | PCI-DSS | ISO 27001 |
|------------------------|-------|-------|------|---------|-----------|
| Using AI code completion | CC6.1, CC7.1 | §164.312(a) | Art. 32 | Req. 6.3 | A.14.2 |
| Sending code to AI chat | CC6.6, CC6.7 | §164.312(e) | Art. 28 | Req. 6.5 | A.13.2 |
| AI-generated code in production | CC8.1 | §164.312(c) | Art. 25 | Req. 6.4 | A.14.2 |
| AI tool access management | CC6.1, CC6.2 | §164.312(a) | Art. 32 | Req. 7.1 | A.9.2 |
| AI tool vendor management | CC9.2 | §164.308(b) | Art. 28 | Req. 12.8 | A.15.1 |
| AI usage logging | CC7.2 | §164.312(b) | Art. 30 | Req. 10.2 | A.12.4 |
| AI data incident response | CC7.3, CC7.4 | §164.308(a)(6) | Art. 33 | Req. 12.10 | A.16.1 |
| AI policy training | CC1.4 | §164.308(a)(5) | Art. 39 | Req. 12.6 | A.7.2 |

### Control-to-Tool Mapping

| Control | GitHub Copilot Business | Claude Code (API) | Generic AI Chat |
|---------|------------------------|-------------------|-----------------|
| Data encryption in transit | ✅ TLS | ✅ TLS | ✅ TLS |
| Training data opt-out | ✅ Default | ✅ API default | ⚠️ Varies |
| Enterprise agreement | ✅ Available | ✅ Available | ❌ Often not |
| Audit logs | ✅ Admin console | ✅ API logs | ❌ Limited |
| Access controls | ✅ Org-level | ✅ API key mgmt | ⚠️ Account-level |
| Data retention control | ✅ No retention | ✅ API policy | ❌ May retain |
| SOC 2 certified | ✅ Yes | ✅ Yes | ⚠️ Varies |
| BAA available (HIPAA) | ⚠️ Limited | ❌ Not standard | ❌ No |
| IP indemnification | ✅ Enterprise | ❌ Not standard | ❌ No |
| Data residency options | ✅ Enterprise | ⚠️ Limited | ❌ Usually not |

---

## 6. Incident Response for AI-Related Events

### 6.1 AI-Specific Incident Types

| Incident Type | Severity | Response |
|---------------|----------|----------|
| Secrets exposed via AI prompt | 🔴 High | Rotate credentials immediately; assess exposure |
| PII sent to AI tool | 🔴 High | Notify privacy team; assess GDPR/HIPAA implications |
| AI tool data breach (provider) | 🔴 High | Follow provider notification; assess impact |
| AI-generated code vulnerability in production | 🟡 Medium | Standard vulnerability response; patch and review |
| Unapproved AI tool usage discovered | 🟡 Medium | Assess data exposure; revoke access; retrain |
| AI tool misconfiguration (training opt-in) | 🟡 Medium | Correct configuration; assess data sent during window |

### 6.2 Response Playbook Template

```markdown
## AI Data Exposure Incident Response

### Identification
- [ ] What AI tool was involved?
- [ ] What data was exposed?
- [ ] Data classification level?
- [ ] When did the exposure occur?
- [ ] Who was involved?
- [ ] Which repositories/projects were affected?

### Containment
- [ ] Revoke or rotate any exposed credentials immediately
- [ ] Disable the AI tool access if necessary
- [ ] Preserve audit logs for investigation
- [ ] Isolate affected systems if warranted

### Assessment
- [ ] What is the AI provider's data retention for this interaction?
- [ ] Was the data used for model training? (Check provider policy)
- [ ] Does this trigger regulatory notification requirements?
  - GDPR: 72-hour notification window
  - HIPAA: 60-day notification requirement
  - State breach notification laws
- [ ] What is the scope of potential impact?

### Remediation
- [ ] Update .copilotignore to prevent recurrence
- [ ] Review and update data classification
- [ ] Implement additional technical controls
- [ ] Conduct targeted training for affected team

### Post-Incident
- [ ] Document the incident and response
- [ ] Update incident response plan with lessons learned
- [ ] Report to compliance/legal as required
- [ ] Schedule follow-up review in 30 days
```

---

## 7. Compliance Reporting Templates

### 7.1 Quarterly Compliance Report

```markdown
# AI Development Compliance Report — Q[X] [Year]

## Executive Summary
[Brief summary of compliance status]

## Tool Inventory
| Tool | Users | Plan Type | Compliance Status |
|------|-------|-----------|-------------------|
| GitHub Copilot | [N] | Enterprise | ✅ Compliant |
| Claude Code | [N] | API (Enterprise) | ✅ Compliant |

## Metrics
- **Policy training completion:** [X]% of developers
- **Repositories with .copilotignore:** [X]% of active repos
- **Secret scanning alerts resolved:** [X] of [Y]
- **AI-related incidents:** [N] (Severity breakdown: ...)
- **Exception requests:** [N] (Approved: [N], Denied: [N])

## Audit Findings
[List any findings from internal or external audits]

## Action Items
[List open action items with owners and due dates]

## Next Steps
[Planned improvements for next quarter]
```

### 7.2 Annual Compliance Review

```markdown
# Annual AI Development Compliance Review — [Year]

## Policy Review
- [ ] AI Usage Policy reviewed and updated
- [ ] Data handling procedures reviewed
- [ ] Approved tools list current
- [ ] Exception process functioning

## Vendor Review
- [ ] AI provider agreements renewed
- [ ] Vendor security assessments current
- [ ] Data processing agreements valid
- [ ] Provider SOC 2 reports reviewed

## Training Review
- [ ] All developers trained (new and existing)
- [ ] Training content updated for policy changes
- [ ] Training effectiveness assessed

## Technical Controls Review
- [ ] .copilotignore files audited
- [ ] Secret scanning effective
- [ ] CI/CD compliance gates functioning
- [ ] Monitoring and alerting operational

## Incident Review
- [ ] AI-related incidents reviewed
- [ ] Lessons learned incorporated
- [ ] Response procedures updated
- [ ] Tabletop exercise conducted

## Regulatory Update
- [ ] New regulations assessed for impact
- [ ] Policy updated for regulatory changes
- [ ] Legal counsel consulted on emerging issues
```

---

## 8. Getting Started with Compliance

### Minimum Viable Compliance

For teams just starting their compliance journey with AI tools:

**Week 1 — Foundation:**
- [ ] Document which AI tools are in use
- [ ] Ensure all tools use organizational (not personal) accounts
- [ ] Create `.copilotignore` for all repositories
- [ ] Enable secret scanning

**Week 2 — Policy:**
- [ ] Adopt or customize the AI usage policy
- [ ] Brief all developers on the policy
- [ ] Configure AI tool privacy settings (training opt-out, etc.)

**Week 3 — Monitoring:**
- [ ] Enable audit logging for AI tools
- [ ] Set up basic alerting for anomalous usage
- [ ] Review and document AI provider data handling

**Week 4 — Integration:**
- [ ] Add compliance checks to CI/CD pipeline
- [ ] Update incident response plan for AI scenarios
- [ ] Schedule quarterly compliance review

**Ongoing:**
- [ ] Monthly: Review audit logs and alerts
- [ ] Quarterly: Full compliance checklist review
- [ ] Annually: Policy update, vendor review, training refresh
