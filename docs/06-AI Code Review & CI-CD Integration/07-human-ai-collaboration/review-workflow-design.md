# Review Workflow Design

> Designing the human-AI handoff process for effective collaborative code review.

---

## The Handoff Model

```
┌─────────────────────────────────────────────────────┐
│  PHASE 1: AI Pre-Screen (Automated)                 │
│                                                     │
│  1. AI reviews diff against rules                   │
│  2. AI categorizes findings by severity             │
│  3. AI generates PR summary                         │
│  4. AI posts inline comments with suggestions       │
│  5. AI sets status: ✅ pass / ⚠️ warn / ❌ block   │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  PHASE 2: Developer Response                        │
│                                                     │
│  1. Developer reviews AI findings                   │
│  2. Applies valid suggestions (one-click)           │
│  3. Dismisses false positives with rationale        │
│  4. Addresses blocking issues                       │
│  5. Pushes updates → AI re-reviews incrementally    │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  PHASE 3: Human Review (Focused)                    │
│                                                     │
│  Human reviewer sees:                               │
│  • AI summary (saves reading entire diff)           │
│  • AI findings (already addressed or dismissed)     │
│  • Remaining issues to focus on                     │
│                                                     │
│  Human focuses on:                                  │
│  • Architecture and design decisions                │
│  • Business logic correctness                       │
│  • AI findings that were dismissed (verify)         │
│  • Overall code quality and maintainability         │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
                  ✅ Merge or 🔄 Request Changes
```

---

## Role Assignments

| Role | Responsibility | Who |
|------|---------------|-----|
| **AI Reviewer** | First-pass analysis, pattern detection | Copilot / CodeRabbit |
| **Developer** | Address AI findings, push fixes | PR author |
| **Human Reviewer** | Architecture, logic, final approval | Team member |
| **Required Reviewer** | Security, compliance sign-off | Senior / specialist |

---

## Workflow Rules

| Rule | Implementation |
|------|---------------|
| Every PR gets AI review | Copilot auto-review via rulesets |
| AI review before human review | Human reviewer sees AI-filtered PR |
| Block on critical AI findings | Required status check |
| 1 human approval required | Branch protection rule |
| 2 approvals for sensitive code | CODEOWNERS for critical paths |

---

## Key Takeaways

1. **AI reviews first** — Filters noise so humans focus on what matters
2. **Developer addresses AI findings** — Before human review starts
3. **Human reviewer sees pre-filtered PR** — Higher quality review in less time
4. **Clear role separation** — AI handles breadth, humans handle depth
5. **Incrementally reviews** — Each push re-triggers AI analysis

---

## Next Steps

- 🔗 [Escalation Protocols](escalation-protocols.md) — When things need human attention
- 🔗 [Team Adoption](team-adoption.md) — Rolling out to your team
