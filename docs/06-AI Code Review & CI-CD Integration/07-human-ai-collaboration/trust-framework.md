# Trust Framework

> A risk-based model for when to trust AI code review suggestions and when to require human override.

---

## The Risk-Based Trust Model

Not all AI suggestions deserve the same level of trust:

```
                    HIGH TRUST              LOW TRUST
                    (Auto-accept)           (Human required)
                    
  LOW RISK ─────── ✅ Formatting fixes      ⚠️ Unusual formatting
                    ✅ Typo corrections      
                    ✅ Import sorting        
                                           
  MEDIUM RISK ──── ⚠️ Simple refactoring   🔶 Logic changes
                    ⚠️ Error msg updates    🔶 API modifications
                                           
  HIGH RISK ────── 🔶 Small bug fixes      ❌ Architecture changes
                    🔶 Test additions        ❌ Security-sensitive code
                                            ❌ Database migrations
```

---

## Trust Decision Matrix

| Factor | Trust AI | Require Human |
|--------|---------|---------------|
| **Blast radius** | Single file, local change | Multiple services, cross-cutting |
| **Reversibility** | Easily reverted | Database migration, data loss |
| **AI confidence** | High (clear pattern match) | Low (novel situation) |
| **Domain** | Standard code patterns | Business-specific logic |
| **Compliance** | Non-regulated code | SOX, GDPR, PCI-affected code |
| **Complexity** | Simple, well-understood | Complex, multi-step logic |

---

## Trust Levels

### Level 1: Auto-Accept
AI suggestion is applied automatically, logged for audit:
- Formatting fixes
- Import optimization
- Whitespace changes
- Comment typos

### Level 2: Accept with Audit
AI suggestion is applied; human reviews post-hoc in batch:
- Simple refactoring (rename variable, extract function)
- Test additions for existing code
- Documentation updates

### Level 3: Require Approval
AI suggests; human must explicitly approve:
- Bug fixes
- New functionality
- API changes
- Configuration changes

### Level 4: Human-Only
AI may advise but cannot propose changes:
- Security-critical code
- Database schema changes
- Authentication/authorization logic
- Payment/financial code
- Deployment configurations

---

## Building Trust Over Time

```
Week 1-2:   Level 3-4 for everything (validate AI accuracy)
Week 3-4:   Level 2 for formatting, Level 3 for code changes
Month 2:    Level 1 for formatting, Level 2 for refactoring
Month 3+:   Steady state based on measured accuracy
```

Track AI suggestion acceptance rate and false positive rate. If acceptance rate is consistently >80%, increase trust. If it drops below 50%, tighten controls.

---

## Key Takeaways

1. **Trust is earned, not assumed** — Start conservative, loosen over time
2. **Risk-based routing** — Different trust levels for different change types
3. **Measure and adjust** — Track acceptance and false positive rates
4. **Human-only for irreversible changes** — Never auto-accept database migrations
5. **Audit everything** — Even auto-accepted changes should be logged

---

## Next Steps

- 🔗 [Review Workflow Design](review-workflow-design.md) — Designing the handoff
- 🔗 [Escalation Protocols](escalation-protocols.md) — When to escalate
