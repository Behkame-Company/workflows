# Metrics & KPIs

> Comprehensive tracking for AI code review and CI/CD effectiveness.

---

## KPI Categories

### Velocity KPIs
| KPI | Definition | Target | How to Measure |
|-----|-----------|--------|---------------|
| **Mean time to first review** | Time from PR open to first review comment | <2 hours | GitHub API |
| **Mean time to merge** | Time from PR open to merge | <24 hours | GitHub API |
| **PR throughput** | PRs merged per developer per week | Baseline + 20% | GitHub API |
| **Review rounds** | Average iterations before approval | ≤2 | GitHub API |

### Quality KPIs
| KPI | Definition | Target | How to Measure |
|-----|-----------|--------|---------------|
| **Defect escape rate** | Bugs reaching production per PR | <5% | Issue tracking |
| **Rollback rate** | Deployments rolled back | <2% | Deploy logs |
| **Test coverage (new)** | Coverage of new code | ≥80% | Coverage tool |
| **Security findings** | Critical/high vulns per sprint | 0 | CodeQL/Semgrep |

### AI Effectiveness KPIs
| KPI | Definition | Target | How to Measure |
|-----|-----------|--------|---------------|
| **AI suggestion acceptance** | % of suggestions applied | 30-60% | Review bot data |
| **AI false positive rate** | % of findings dismissed | <20% | Review bot data |
| **AI finding density** | Findings per PR | 3-8 | Review bot data |
| **Human review time saved** | Reduction in human review time | 30-50% | Time tracking |

---

## Dashboard Template

Track weekly and display trends:

```
Week   | TTM (hrs) | Acceptance | FP Rate | Defect Escape | Coverage
-------|-----------|------------|---------|---------------|--------
W1     | 32        | 22%        | 45%     | 8%            | 62%
W4     | 24        | 38%        | 28%     | 5%            | 71%
W8     | 18        | 52%        | 15%     | 3%            | 82%
W12    | 16        | 58%        | 12%     | 2%            | 85%
```

---

## Key Takeaways

1. **Measure before and after** — Baseline is essential for proving value
2. **Track AI-specific metrics** — Acceptance rate and false positives
3. **Quality AND velocity** — Both should improve, not trade off
4. **Weekly reviews for first 3 months** — Tune based on trends
5. **Share dashboard with team** — Transparency drives improvement

---

## Next Steps

- 🔗 [Common Mistakes](../09-anti-patterns/common-mistakes.md) — What to avoid
- 🔗 [Enterprise at Scale](../10-advanced/enterprise-review-at-scale.md) — Scaling metrics
