# Metrics & Benchmarks

> Measuring the quality of AI-generated code and the effectiveness of your review process.

---

## Code Quality Metrics

| Metric | What It Measures | Target |
|--------|-----------------|--------|
| **Defect escape rate** | Bugs that reach production | <5% of PRs |
| **Test coverage (new code)** | % of new lines tested | ≥80% |
| **Cyclomatic complexity** | Code complexity per function | ≤10 |
| **Code duplication** | % of duplicated code | <3% for new code |
| **Security findings** | Critical/high vulnerabilities | 0 |
| **Build success rate** | % of builds that pass | >95% |

---

## Review Process Metrics

| Metric | What It Measures | Target |
|--------|-----------------|--------|
| **Mean time to review** | Hours from PR open to first review | <4 hours |
| **Mean time to merge** | Hours from PR open to merge | <24 hours |
| **Review rounds** | Number of review iterations per PR | ≤2 |
| **AI suggestion acceptance rate** | % of AI suggestions applied | 30-60% |
| **False positive rate** | % of AI findings that are noise | <20% |
| **Review coverage** | % of PRs that receive review | 100% |

---

## AI-Specific Metrics

| Metric | What It Measures | Why It Matters |
|--------|-----------------|---------------|
| **AI code acceptance rate** | % of AI-generated code that makes it to production unchanged | Measures AI quality |
| **AI defect density** | Bugs per 1000 lines of AI-generated code | Compare to human-written code |
| **Hallucination rate** | % of AI code with non-existent imports/APIs | Critical safety metric |
| **Rework rate** | % of AI code that needs significant changes after review | Measures AI usefulness |

---

## Tracking Dashboard

Build a dashboard with these key indicators:

```
┌─────────────────────────────────────────────────┐
│  AI CODE QUALITY DASHBOARD                      │
│                                                 │
│  Defect Escape Rate:  3.2%  ✅ (target: <5%)   │
│  AI Acceptance Rate:  67%   ✅ (target: >50%)   │
│  False Positive Rate: 15%   ✅ (target: <20%)   │
│  Mean Time to Merge:  18h   ✅ (target: <24h)   │
│  Coverage (new code): 84%   ✅ (target: ≥80%)   │
│  Security Findings:   0     ✅ (target: 0)      │
│                                                 │
│  Trend: ↑ Improving over last 4 weeks           │
└─────────────────────────────────────────────────┘
```

---

## Key Takeaways

1. **Track AI-specific metrics** — Hallucination rate, rework rate, acceptance rate
2. **Compare AI vs human code** — Defect density should be similar or better
3. **Optimize signal-to-noise** — False positive rate under 20%
4. **Time to merge is key** — If AI review slows merging, it's a net negative
5. **Dashboard visibility** — Make metrics visible to the whole team
