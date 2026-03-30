# Escalation Protocols

> When AI must defer to humans — clear rules for escalation and override.

---

## Escalation Triggers

### Automatic Escalation (AI → Human)

| Trigger | Action | Rationale |
|---------|--------|-----------|
| **Critical security finding** | Notify security team immediately | Potential active vulnerability |
| **AI confidence below threshold** | Flag for human review | AI unsure = human decides |
| **Regulated code path** | Require compliance reviewer | SOX/GDPR/PCI requirements |
| **Cross-service impact** | Require architect review | Blast radius too large |
| **AI disagreement** | Flag both opinions | Multiple AI tools disagree |
| **Novel pattern** | Flag for human | No training data precedent |

### Manual Escalation (Developer → Human)

| Trigger | Action |
|---------|--------|
| Developer disagrees with AI | Dismiss with documented rationale |
| AI suggestion seems harmful | Escalate to senior developer |
| AI can't understand context | Request human review with context |
| Multiple review rounds with no resolution | Escalate to team lead |

---

## Escalation Workflow

```
AI Finding
    │
    ├── Critical/High severity?
    │   YES → Block merge + Notify reviewer
    │   NO  ↓
    │
    ├── AI confidence < 80%?
    │   YES → Flag for human review (don't block)
    │   NO  ↓
    │
    ├── Regulated code path? (CODEOWNERS)
    │   YES → Require specialist approval
    │   NO  ↓
    │
    └── Standard finding → Developer addresses
```

---

## Override Protocol

When a developer overrides an AI finding:

1. **Document the rationale** — Comment explaining why the AI is wrong
2. **Human reviewer must see it** — Override is visible in the review
3. **Log for audit** — Track override patterns to tune AI
4. **Batch review overrides** — Periodically review all overrides to find patterns

---

## Key Takeaways

1. **Clear triggers** — Define exactly when AI must defer to humans
2. **Severity drives escalation** — Critical blocks, low informs
3. **Override requires rationale** — Don't dismiss AI silently
4. **Audit overrides** — Patterns reveal where AI needs tuning or humans need training
5. **Regulated code always escalates** — No exceptions for compliance-sensitive code
