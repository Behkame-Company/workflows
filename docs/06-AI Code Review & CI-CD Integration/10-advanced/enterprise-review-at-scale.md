# Enterprise Review at Scale

> Strategies for managing AI code review across multiple teams and hundreds of repositories.

---

## Challenges at Scale

| Challenge | At Small Scale | At Enterprise Scale |
|-----------|---------------|-------------------|
| **Instructions** | One copilot-instructions.md | Org-level + per-repo override |
| **Consistency** | Team agrees informally | Need enforced standards |
| **Metrics** | Glance at dashboard | Aggregated across 100+ repos |
| **Cost** | Negligible | Budget management needed |
| **Governance** | Trust-based | Policy-enforced |

---

## Organizational Architecture

```
┌─────────────────────────────────────────┐
│           Organization Level            │
│  • Global copilot-instructions.md       │
│  • Org-wide rulesets                    │
│  • Central metrics dashboard            │
│  • Budget allocation                    │
└──────────────┬──────────────────────────┘
               │
    ┌──────────┼──────────┐
    ▼          ▼          ▼
┌──────┐  ┌──────┐  ┌──────┐
│Team A│  │Team B│  │Team C│
│      │  │      │  │      │
│Custom│  │Custom│  │Custom│
│rules │  │rules │  │rules │
└──────┘  └──────┘  └──────┘
```

### Layered Instruction Model
1. **Org-level**: Security rules, compliance requirements, universal standards
2. **Team-level**: Tech-stack-specific rules, team conventions
3. **Repo-level**: Project-specific rules, overrides

---

## Governance Model

### Centralized Policies
| Policy | Enforcement |
|--------|-------------|
| Security scanning required | Org-level ruleset |
| AI review enabled | Org-level ruleset |
| Test coverage >70% | Org-level status check |
| No critical vulnerabilities | Branch protection |

### Team Autonomy
| Allowed to Customize | Not Allowed to Override |
|----------------------|----------------------|
| Review instructions | Security scanning |
| Quality thresholds | Required human review |
| AI reviewer choice | Vulnerability blocking |
| Test coverage target (above minimum) | Audit logging |

---

## Metrics Aggregation

Dashboard views:
1. **Executive view** — Defect escape rate, security posture, velocity trends
2. **Team lead view** — Per-team metrics, review quality, adoption rates
3. **Developer view** — Personal metrics, suggestion history, learning

---

## Key Takeaways

1. **Layer instructions** — Org → Team → Repo hierarchy
2. **Centralize governance** — Security and compliance non-negotiable
3. **Allow team autonomy** — For conventions and quality thresholds
4. **Aggregate metrics** — Different views for different audiences
5. **Budget management** — Monitor AI API costs across teams

---

## Next Steps

- 🔗 [Multi-Repo Review](multi-repo-review.md) — Cross-repository analysis
- 🔗 [Custom Review Agents](custom-review-agents.md) — Building your own
