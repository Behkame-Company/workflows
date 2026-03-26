# Enterprise SDD

> Scaling spec-driven development across teams, departments, and organizations.

---

## Enterprise Challenges

Scaling SDD from a single team to an organization introduces:

| Challenge | Description |
|-----------|-------------|
| Cross-team specs | Features that span multiple teams |
| Spec governance | Who approves what, and when |
| Template standardization | Consistent spec quality across 50+ engineers |
| Spec discovery | Finding relevant specs in a large codebase |
| Compliance mapping | Tracing specs to regulatory requirements |
| Onboarding | Teaching new hires the SDD workflow |

---

## Organization-Level Constitution

### Hierarchy

```
Organization Constitution (company-wide)
├── Division Standards (e.g., "Platform" vs "Consumer")
│   ├── Team Constitution (project-specific)
│   │   └── Feature Specs
```

### Example: Org Constitution
```markdown
# Acme Corp Engineering Constitution

## Universal Standards
- All production features require a specification
- Specs must include acceptance criteria and edge cases
- Security review required for features handling PII
- Accessibility compliance: WCAG 2.1 AA minimum

## Spec Governance
- Specs reviewed by at least 2 engineers
- Architecture-impacting specs require tech lead approval
- Cross-team specs require representatives from each team
```

---

## Spec Ownership at Scale

### The RACI Model for Specs

| Role | Responsible | Accountable | Consulted | Informed |
|------|------------|-------------|-----------|----------|
| Spec drafting | Feature engineer | Tech lead | Product manager | Team |
| Spec review | Peer engineer | Tech lead | Security, QA | Team |
| Plan creation | Senior engineer | Tech lead | Architects | Team |
| Implementation | Engineers / AI | Tech lead | Spec author | Stakeholders |

### Cross-Team Specs
When a feature spans teams:

```markdown
# Cross-Team Spec: Unified Search

## Owning Team: Search Infrastructure
## Contributing Teams: Product Catalog, User Profiles, Content

## Team Responsibilities
| Team | Owns | Provides to Search |
|------|------|-------------------|
| Search Infra | Search API, indexing pipeline | Query API |
| Product Catalog | Product data model | Product index adapter |
| User Profiles | User data model | User index adapter |
| Content | Content data model | Content index adapter |

## Integration Contract
Each team provides an index adapter implementing:
[interface specification here]
```

---

## Compliance and Audit Trails

### Regulatory Mapping

```markdown
## Spec: User Data Export (GDPR Article 20)

### Regulatory Reference
| Regulation | Article | Requirement |
|-----------|---------|-------------|
| GDPR | Art. 20 | Data portability — structured, machine-readable format |
| GDPR | Art. 15 | Right of access — complete data within 30 days |
| SOC 2 | CC6.1 | Logical access controls |

### Acceptance Criteria (mapped to regulation)
- AC-1 [GDPR Art.20]: Export includes all user data in JSON format
- AC-2 [GDPR Art.15]: Export delivered within 72 hours of request
- AC-3 [SOC2 CC6.1]: Export requires authenticated request with 2FA
```

### Audit Trail
```
Spec created:     2026-01-15 by @jane (PR #142)
Spec reviewed:    2026-01-17 by @bob, @alice (PR #142)
Spec approved:    2026-01-18 by @techleadmike (PR #142)
Plan created:     2026-01-19 by @bob (PR #145)
Implementation:   2026-01-22 to 2026-02-01 (PRs #148-155)
Verification:     2026-02-02 by @qa-team (verified all ACs)
```

---

## Spec Templates at Scale

### Template Library
```
.specify/templates/
├── feature-spec.md          # Standard feature
├── api-spec.md             # API endpoint/service
├── migration-spec.md       # Data/schema migration
├── integration-spec.md     # Cross-team integration
├── compliance-spec.md      # Regulatory feature
├── performance-spec.md     # Performance improvement
└── deprecation-spec.md     # Feature deprecation plan
```

### Template Governance
- Templates owned by engineering standards team
- Changes require RFC process
- New templates created when a pattern repeats 3+ times

---

## Metrics and Reporting

### SDD Health Dashboard

```
Organization SDD Metrics — Q1 2026
────────────────────────────────────
Spec Coverage:       82% (target: 80%) ✅
Avg Review Time:     1.2 days (target: <2 days) ✅
Spec Accuracy:       91% at completion ✅
Rework Rate:         15% (target: <20%) ✅
Specs Per Team/Month: 4.3 avg
Active Specs:        127 across 15 teams
```

### ROI Measurement
Track before/after SDD adoption:
- Rework rate (should decrease)
- Time to production (should stabilize)
- Bug escape rate (should decrease)
- Onboarding time (should decrease)
- AI agent effectiveness (should increase)

---

*Next: [Multi-Agent Pipelines →](multi-agent-pipelines.md)*
