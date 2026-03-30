# AI vs Human Code Review

> Understanding what each excels at, and why the hybrid model wins.

---

## Comparison Matrix

| Dimension | AI Review | Human Review |
|-----------|-----------|-------------|
| **Speed** | Seconds (instant on PR open) | Hours to days |
| **Consistency** | Same standards every time | Varies by reviewer, mood, workload |
| **Availability** | 24/7, unlimited capacity | Limited by team size |
| **Pattern detection** | Excellent — trained on millions of repos | Good but fatigues after ~200 lines |
| **Security scanning** | Systematic, covers known vulnerability patterns | Depends on reviewer expertise |
| **Business logic** | Poor — lacks domain context | Excellent — understands the "why" |
| **Architecture** | Limited — sees diff, not full system | Strong — understands system evolution |
| **Novel situations** | Weak — relies on training patterns | Strong — can reason about new problems |
| **Cost per review** | Low (pennies per PR) | High (engineer time × salary) |
| **False positives** | Medium-High (needs tuning) | Low (experienced reviewers) |

---

## The Hybrid Model

Neither AI nor human review alone is sufficient. The hybrid model uses each where it's strongest:

```
┌─────────────────────────────────────────┐
│  AI HANDLES (Automated First Pass)      │
│                                         │
│  ✅ Style and formatting violations     │
│  ✅ Known bug patterns                  │
│  ✅ Security vulnerability detection    │
│  ✅ Test coverage gaps                  │
│  ✅ Documentation completeness          │
│  ✅ Performance anti-patterns           │
│  ✅ Dependency issues                   │
└───────────────────┬─────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│  HUMAN HANDLES (Focused Second Pass)    │
│                                         │
│  ✅ Business logic correctness          │
│  ✅ Architectural decisions             │
│  ✅ API design and contracts            │
│  ✅ Edge cases requiring domain context │
│  ✅ Performance for specific workloads  │
│  ✅ Team knowledge sharing              │
│  ✅ Final merge/reject decision         │
└─────────────────────────────────────────┘
```

---

## Risk-Based Routing

Not all code changes need the same level of review:

| Change Type | AI Review | Human Review | Rationale |
|------------|-----------|-------------|-----------|
| Formatting/typos | ✅ Auto-approve | ❌ Skip | Zero risk |
| Test additions | ✅ Required | ⚠️ Optional | Low risk, high value |
| Bug fixes (small) | ✅ Required | ✅ Required | Medium risk |
| New features | ✅ Required | ✅ Required | High risk |
| Architecture changes | ✅ Required | ✅✅ Senior required | Critical risk |
| Security-sensitive | ✅ Required | ✅✅ Security team | Critical risk |
| Database migrations | ✅ Required | ✅✅ DBA required | Irreversible |

---

## Key Takeaways

1. **AI excels at breadth** — It catches everything consistently across every PR
2. **Humans excel at depth** — They understand context, intent, and architecture
3. **The hybrid model wins** — AI filters the noise so humans focus on what matters
4. **Route by risk** — Not every change needs the same review intensity
5. **AI saves 30-50% of review time** — But never 100%
