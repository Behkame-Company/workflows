# Enterprise & Organization Strategies

> Scaling AGENTS.md across teams, organizations, and enterprise environments.

---

## The Enterprise Challenge

Individual repos are easy: one AGENTS.md, one team, one stack.

At enterprise scale, you face:
- **100+ repositories** with different stacks
- **Multiple teams** with different conventions
- **Compliance requirements** (SOC2, HIPAA, PCI-DSS)
- **Governance** (who controls what agents can do?)
- **Consistency** (how do you enforce standards across repos?)

---

## Layer 1: Organization-Wide Standards

### The Org Template

Create a shared AGENTS.md template that all repos start from:

```markdown
# AGENTS.md — [Org Name] Standard Template
# Last updated: 2026-03-01

## Global Policies
- NEVER commit credentials, tokens, or secrets
- NEVER disable security middleware or CORS
- NEVER create administrative backdoors
- NEVER access production databases from development
- NEVER use eval(), Function(), or dynamic code execution
- All agent-generated PRs require human review before merge

## Security
- Follow OWASP Top 10 guidelines
- Sanitize all user input
- Use parameterized queries only
- Log security-relevant events to audit trail

## Git Standards
- Conventional commits: feat:, fix:, docs:, test:, chore:, ci:
- Branch naming: [type]/[ticket-id]/[description]
- PRs require at least 2 approvals
- Squash merge to main

## Below this line, add project-specific instructions
---
```

### Distribution

1. **GitHub Template Repository**: Include AGENTS.md in your org's template repo
2. **Repository creation script**: Auto-generate AGENTS.md with org template
3. **CI check**: Validate that AGENTS.md includes required org sections

---

## Layer 2: Team-Level Conventions

Teams add their stack-specific rules on top of org standards:

```markdown
# AGENTS.md — [Team Name] / [Project Name]
# Inherits org policies from above.

## Stack
TypeScript 5.4, React 18, Next.js 15, Prisma, PostgreSQL.

## Setup
- Install: `pnpm install`
- Test: `pnpm test`
- Build: `pnpm build`

## Team Conventions
- TypeScript strict mode
- Named exports only
- Server Components by default
- Zod for validation
```

---

## Layer 3: Repository-Specific Rules

Individual repos add their unique requirements:

```markdown
## Project-Specific
- Feature flags: LaunchDarkly (check `isFeatureEnabled()` before new features)
- A/B testing: use `getVariant()` from src/lib/experiments
- Monitoring: all API routes wrapped in `withTracing()` middleware
```

---

## Governance Model

### Who Controls What

| Layer | Owner | Review Required |
|-------|-------|----------------|
| Org policies (NEVER rules, security) | Security team | Security team + CTO |
| Team conventions (stack, patterns) | Tech lead | Senior engineers |
| Project specifics (commands, structure) | Any engineer | Team review |

### CODEOWNERS Configuration

```
# Org-wide AGENTS.md governance
AGENTS.md @org/security-team @org/platform-team

# For monorepos, per-package ownership
apps/web/AGENTS.md @org/frontend-team
apps/api/AGENTS.md @org/backend-team
services/*/AGENTS.md @org/platform-team
```

---

## Compliance Integration

### SOC2 Requirements

```markdown
## Compliance (SOC2)
- All data access must go through authorized service layer
- User actions logged to audit trail (src/lib/audit)
- PII must be encrypted at rest
- Access control: role-based (never hard-coded permissions)
- Change management: all changes via PR, never direct to main
```

### HIPAA Requirements

```markdown
## Compliance (HIPAA)
- NEVER log patient data (names, DOB, SSN, medical records)
- NEVER return PII in API error messages
- All patient data encrypted in transit (TLS) and at rest (AES-256)
- Access: minimum necessary principle
- Audit trail required for all data access
```

### PCI-DSS Requirements

```markdown
## Compliance (PCI-DSS)
- NEVER store credit card numbers in application database
- NEVER log payment card data
- Payment processing: Stripe API only (never direct card handling)
- All payment-related code in src/server/payments/ (isolated module)
- Quarterly security review of payment code required
```

---

## Scaling Patterns

### Pattern: The AGENTS.md Registry

Maintain a central registry of approved AGENTS.md templates:

```
org-standards/
├── templates/
│   ├── typescript-react.md     ← Template for React projects
│   ├── python-fastapi.md       ← Template for FastAPI projects
│   ├── go-service.md           ← Template for Go microservices
│   └── base.md                 ← Minimal template (org policies only)
├── sections/
│   ├── security.md             ← Reusable security section
│   ├── git-workflow.md         ← Reusable git workflow section
│   └── compliance-soc2.md     ← Reusable compliance section
└── validators/
    └── validate-agents-md.sh   ← CI validation script
```

### Pattern: CI Validation

```yaml
# .github/workflows/validate-agents-md.yml
name: Validate AGENTS.md
on: [push, pull_request]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Validate AGENTS.md
        run: |
          test -f AGENTS.md || (echo "ERROR: AGENTS.md required" && exit 1)
          for section in "Boundaries" "Security" "Setup"; do
            grep -q "## $section" AGENTS.md || (echo "Missing: $section" && exit 1)
          done
```

See [Templates Library](../../08-practical/templates-library.md) for full validation scripts including NEVER-rule checks and file size warnings.

### Pattern: Automated Sync

Use a GitHub Action triggered on changes to `org-standards/` to push updates to all repos. The action iterates over a repo matrix, updating the org-policy sections of each repo's AGENTS.md while preserving project-specific sections.

See [Templates Library](../../08-practical/templates-library.md) for a complete sync workflow.

---

## Metrics & Reporting

### What to Track

| Metric | How to Measure | Target |
|--------|---------------|--------|
| AGENTS.md adoption | Repos with AGENTS.md / total repos | > 90% |
| Freshness | Average days since last update | < 60 days |
| File size | Average lines per AGENTS.md | < 150 lines |
| CI compliance | Repos passing AGENTS.md validation | > 95% |
| Agent PR success | Agent PRs merged without rework | > 60% |
| Security compliance | AGENTS.md includes required NEVER rules | 100% |

### Dashboard Example

```
AGENTS.md Org Health — March 2026
──────────────────────────────────
Adoption:    142/150 repos (95%)  ✅
Freshness:   avg 28 days         ✅
Avg size:    87 lines             ✅
CI passing:  139/142 (98%)        ✅
Agent PRs:   67% merge rate       ⚠️ (target: 70%)
Security:    142/142 (100%)       ✅
```

---

## Getting Started at Enterprise Scale

1. **Week 1**: Create org template with security policies
2. **Week 2**: Add CI validation to template repos
3. **Week 3**: Roll out to 5 pilot repos; gather feedback
4. **Week 4**: Iterate on template based on feedback
5. **Month 2**: Roll out to all active repos
6. **Quarterly**: Audit compliance and update org policies
