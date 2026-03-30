# Enterprise AI Tool Governance

> Large-scale organizational concerns for AI tool adoption — from license management and compliance to network restrictions and cost control. This guide addresses what leadership, security, and IT teams need to know.

---

## Governance Framework Overview

```
┌─────────────────────────────────────────────────────────┐
│                   Enterprise AI Governance               │
├──────────────┬──────────────┬──────────────┬────────────┤
│   Identity   │     Data     │  Compliance  │    Cost    │
│   & Access   │   & Privacy  │  & Audit     │  & Budget  │
├──────────────┼──────────────┼──────────────┼────────────┤
│ SSO/SAML     │ Data         │ SOC2         │ License    │
│ SCIM         │ residency    │ HIPAA        │ management │
│ RBAC         │ Content      │ GDPR         │ Usage      │
│ MFA          │ exclusions   │ Audit trails │ tracking   │
│              │ Retention    │              │ Chargeback │
└──────────────┴──────────────┴──────────────┴────────────┘
```

---

## 1. License Management

### GitHub Copilot Plans

| Feature | Individual | Business | Enterprise |
|---|---|---|---|
| Code completion | ✅ | ✅ | ✅ |
| Chat | ✅ | ✅ | ✅ |
| CLI | ✅ | ✅ | ✅ |
| Organization policies | ❌ | ✅ | ✅ |
| Content exclusions | ❌ | ✅ | ✅ |
| Audit logs | ❌ | ✅ | ✅ |
| SSO/SAML | ❌ | Via GitHub | ✅ |
| IP indemnity | ❌ | ❌ | ✅ |
| Self-hosted models | ❌ | ❌ | ✅ |

**Recommendation:** Business or Enterprise for teams; Enterprise for regulated industries.

### Claude Plans

| Feature | Free | Pro | Team | Enterprise |
|---|---|---|---|---|
| Claude Code | Limited | ✅ | ✅ | ✅ |
| API access | ❌ | Limited | ✅ | ✅ |
| SSO | ❌ | ❌ | ❌ | ✅ |
| Data retention controls | ❌ | ❌ | ✅ | ✅ |
| Admin console | ❌ | ❌ | ✅ | ✅ |
| No training on data | ❌ | ❌ | ✅ | ✅ |

**Recommendation:** Team for most organizations; Enterprise for regulated industries.

### License Allocation Strategy

```
Step 1: Identify all developers who will use AI tools
Step 2: Assign licenses based on rollout phase
  - Phase 1 (Pilot): 5-10 licenses
  - Phase 2 (Expand): 20-50 licenses
  - Phase 3 (Scale): All developers
Step 3: Track usage to identify unused licenses
Step 4: Reallocate unused licenses quarterly
```

---

## 2. SSO and Identity

### SAML Configuration

Ensure AI tools authenticate through your identity provider:

```yaml
# GitHub Copilot Enterprise - SAML SSO
identity_provider:
  type: SAML 2.0
  providers: [Okta, Azure AD, OneLogin, PingFederate]
  
configuration:
  - Enable SAML SSO at organization level
  - Require SAML authentication for all members
  - Configure SCIM for automatic provisioning/deprovisioning
  - Enforce MFA through IdP
```

### SCIM Provisioning

Automate user lifecycle management:

```
Employee joins → IdP creates account → SCIM provisions AI tool access
Employee leaves → IdP disables account → SCIM removes AI tool access
Role changes → IdP updates groups → SCIM adjusts permissions
```

### Access Control

| Role | Copilot Access | Model Access | Admin Access |
|---|---|---|---|
| Developer | Code completion, chat | Standard models | None |
| Senior Dev | All features | Standard + advanced | None |
| Team Lead | All features | All approved models | Team policies |
| Security | Review access | All models for testing | Security policies |
| Admin | N/A | N/A | Full admin |

---

## 3. Data Residency

### Where Does Code Go?

Understand the data flow for each AI tool:

```
Developer's Code → AI Tool API → Model Processing → Response
      │                │               │
      │                │               └── Where is the model?
      │                └── Where is the API endpoint?
      └── Where is the developer?
      
Key questions:
- Does code leave the developer's machine?
- Is code stored on the AI provider's servers?
- Is code used for model training?
- In which geographic region is data processed?
```

### Data Residency Requirements

| Requirement | GitHub Copilot Enterprise | Claude Enterprise |
|---|---|---|
| Data processing region | Configurable | Configurable |
| Code storage | Transient (not stored) | Configurable retention |
| Training exclusion | ✅ (Business/Enterprise) | ✅ (Team/Enterprise) |
| On-premises option | Via GitHub Enterprise Server | Not currently |
| VPN/private link | Via GitHub private networking | Via API configuration |

### Data Classification

```
PUBLIC code:        Open source, public APIs → standard AI tool usage
INTERNAL code:      Proprietary business logic → AI tools with business tier
CONFIDENTIAL code:  Trade secrets, algorithms → AI tools with enterprise tier + content exclusions
RESTRICTED code:    Regulated data (HIPAA/PCI) → AI tools with self-hosted models or excluded entirely
```

---

## 4. Compliance

### SOC2

For organizations with SOC2 compliance:

- [ ] AI tool usage documented in system description
- [ ] Access controls include AI tool permissions
- [ ] Change management process covers AI-generated code
- [ ] Monitoring includes AI tool audit logs
- [ ] Vendor risk assessment completed for AI tool providers
- [ ] Data handling addendum covers AI tool data flows

### HIPAA

For organizations handling protected health information (PHI):

- [ ] AI tools configured with BAA (Business Associate Agreement) if processing PHI
- [ ] Content exclusions prevent PHI from being sent to AI tools
- [ ] `.copilotignore` covers all files that may contain PHI
- [ ] Audit logs track AI tool usage in PHI-adjacent systems
- [ ] Developer training includes PHI handling with AI tools

### GDPR

For organizations processing EU personal data:

- [ ] Data Processing Agreement (DPA) in place with AI tool providers
- [ ] Data residency configured for EU processing where required
- [ ] Retention policies configured (no unnecessary data storage)
- [ ] Personal data excluded from AI tool context via content exclusions
- [ ] Developer training includes GDPR considerations with AI tools

---

## 5. Audit Trails

### What to Log

```yaml
audit_events:
  - who: Developer identity (SSO username)
  - what: AI tool action (completion, chat, code generation)
  - when: Timestamp
  - where: Repository, file path (if applicable)
  - tool: Which AI tool was used
  - model: Which model was invoked
  - scope: autocomplete | chat | CLI | agent
```

### GitHub Copilot Audit Log

GitHub Copilot Enterprise provides audit logs via the GitHub audit log:

```
GitHub Organization Settings → Audit Log → Filter: copilot
```

Events captured:
- Copilot seat assignment/removal
- Policy changes
- Content exclusion changes

### Custom Audit Trail

For more granular tracking, implement custom logging:

```yaml
# Example: Custom audit webhook
audit:
  destination: https://your-siem.example.com/api/ingest
  events:
    - ai_code_generation
    - ai_pr_review
    - ai_chat_session
  fields:
    - developer_id
    - repository
    - tool_used
    - model_used
    - timestamp
```

---

## 6. Model Selection Policies

### Approved Models

Define which AI models are approved for organizational use:

| Model | Status | Use Cases | Restrictions |
|---|---|---|---|
| GPT-4o | ✅ Approved | General coding, chat | Standard data handling |
| Claude Sonnet | ✅ Approved | Complex reasoning, review | Enterprise tier only |
| Claude Haiku | ✅ Approved | Quick tasks, autocomplete | Standard data handling |
| Open source (local) | ✅ Approved | Sensitive code | Must run on approved infra |
| Unapproved models | ❌ Blocked | N/A | Not evaluated for security |

### Model Evaluation Criteria

Before approving a new model:

- [ ] Vendor security assessment completed
- [ ] Data handling practices reviewed
- [ ] Terms of service reviewed (training, retention)
- [ ] Performance evaluated for relevant use cases
- [ ] Cost analysis completed
- [ ] Compliance implications assessed

---

## 7. MCP Server Policies

### Approved MCP Servers

```yaml
mcp_policy:
  allowed:
    - type: official
      description: "GitHub-published MCP servers"
    - type: internal
      description: "Organization-developed MCP servers"
      registry: "https://internal-registry.example.com"
  
  blocked:
    - type: unreviewed
      description: "Any MCP server not in the approved list"
  
  review_required:
    - type: third-party
      description: "Community MCP servers require security review"
```

### MCP Server Review Checklist

Before approving an MCP server:
- [ ] Source code reviewed (or trusted publisher verified)
- [ ] Data access scope reviewed (what data can it access?)
- [ ] Network access reviewed (what external calls does it make?)
- [ ] No credential harvesting risk
- [ ] Runs within approved network boundaries
- [ ] Logging and monitoring configured

---

## 8. Network Restrictions

### Proxy Configuration

```yaml
# Corporate proxy configuration for AI tools
proxy:
  http_proxy: "http://proxy.example.com:8080"
  https_proxy: "http://proxy.example.com:8080"
  no_proxy: "localhost,127.0.0.1,.internal.example.com"

# Tool-specific proxy settings
copilot:
  env:
    HTTP_PROXY: "http://proxy.example.com:8080"
    HTTPS_PROXY: "http://proxy.example.com:8080"
```

### Firewall Rules

Required outbound access for AI tools:

| Tool | Destination | Port | Protocol |
|---|---|---|---|
| GitHub Copilot | `*.githubcopilot.com` | 443 | HTTPS |
| GitHub Copilot | `*.github.com` | 443 | HTTPS |
| Claude Code | `api.anthropic.com` | 443 | HTTPS |
| Claude Code | `claude.ai` | 443 | HTTPS |

### Air-Gapped Environments

For environments without internet access:
- Self-hosted models via GitHub Enterprise Server + Copilot
- Local LLM deployment (open-source models)
- Increased focus on instruction files and manual processes

---

## 9. Budget and Cost Management

### Cost Tracking

```
Monthly AI tool cost = (Copilot licenses × price) + (Claude licenses × price) + (API usage)
Per-developer cost = Monthly AI tool cost / active developers
ROI indicator = Estimated time savings × average developer cost - AI tool cost
```

### Budget Template

| Category | Monthly Cost | Annual Cost | Notes |
|---|---|---|---|
| Copilot Business (50 seats) | $950 | $11,400 | $19/seat/month |
| Claude Team (50 seats) | $1,500 | $18,000 | $30/seat/month |
| MCP server hosting | $200 | $2,400 | Internal servers |
| Training time | $5,000 | $60,000 | Estimated dev time |
| Champion time | $2,000 | $24,000 | 10-20% of champions |
| **Total** | **$9,650** | **$115,800** | |

### Cost Optimization

- Monitor license utilization (reassign unused seats)
- Choose appropriate model tiers (don't use premium for simple tasks)
- Consolidate tools where possible (reduce vendor count)
- Track API usage to prevent runaway costs

---

## GitHub Copilot Organization Policies

For GitHub Copilot Business/Enterprise, configure at the organization level:

### Content Exclusions

```yaml
# GitHub Organization Settings → Copilot → Content exclusions
content_exclusions:
  - repository: "org/secrets-repo"
  - path: "**/config/production/**"
  - path: "**/*.env"
  - path: "**/credentials/**"
  - repository: "org/hipaa-data"
```

### Policy Settings

```yaml
copilot_policies:
  suggestions_matching_public_code: block  # Prevent public code suggestions
  copilot_chat_in_ide: allow
  copilot_cli: allow
  copilot_in_github: allow  # PR summaries, etc.
```

---

## Enterprise Governance Framework

### Summary Checklist

```markdown
## Enterprise AI Governance Checklist

### Identity & Access
- [ ] SSO configured for all AI tools
- [ ] SCIM provisioning active
- [ ] Role-based access defined
- [ ] MFA enforced

### Data & Privacy
- [ ] Data residency requirements met
- [ ] Content exclusions configured
- [ ] Training opt-out confirmed
- [ ] Data retention policies set

### Compliance
- [ ] Vendor risk assessments complete
- [ ] DPAs/BAAs in place
- [ ] Audit logging enabled
- [ ] Compliance documentation updated

### Security
- [ ] Approved models defined
- [ ] MCP server policy established
- [ ] Network restrictions configured
- [ ] Secret scanning active

### Operations
- [ ] License management process
- [ ] Budget tracking established
- [ ] Cost optimization reviews scheduled
- [ ] Incident response plan includes AI tools

### Governance
- [ ] Policy owner assigned
- [ ] Review cadence established (quarterly)
- [ ] Exception process defined
- [ ] Communication plan for policy changes
```
