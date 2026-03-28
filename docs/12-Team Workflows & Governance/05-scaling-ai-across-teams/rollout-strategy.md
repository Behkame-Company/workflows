# AI Tool Rollout Strategy

> Phased adoption across an organization — from pilot to full scale. Rushing adoption leads to inconsistency and resistance. A measured rollout with feedback loops leads to sustainable success.

---

## Four-Phase Rollout

```
Phase 1          Phase 2           Phase 3           Phase 4
PILOT            EXPAND            SCALE             OPTIMIZE
1 team           3-5 teams         All teams         Org-wide
8-12 weeks       8-12 weeks        12-16 weeks       Ongoing
────────────────────────────────────────────────────────────────▶
     Learn            Adapt             Standardize       Improve
```

---

## Phase 1: Pilot (8-12 Weeks)

**Goal:** Validate AI tools with one team on a low-risk project, measure results, learn lessons.

### Team Selection
Choose a pilot team that:
- ✅ Has a willing team lead (champion, not skeptic)
- ✅ Works on a non-critical project (room for mistakes)
- ✅ Has a mix of experience levels (diverse feedback)
- ✅ Represents a common tech stack (learnings transfer)
- ❌ Not your most critical production system
- ❌ Not a team that's already overloaded

### Activities

| Week | Activity |
|---|---|
| 1-2 | Tool setup, instruction file creation, team training |
| 3-4 | Guided usage with buddy system, daily check-ins |
| 5-6 | Independent usage, first PR reviews with AI checklist |
| 7-8 | Prompt library started, process refinements |
| 9-10 | Metrics collection, retrospective, document learnings |
| 11-12 | Pilot report creation, recommendations for Phase 2 |

### Metrics to Collect
- Developer satisfaction (survey, 1-5 scale)
- Time spent on common tasks (before vs. after)
- Code quality indicators (bug rate, review cycles)
- Instruction file effectiveness (rules followed %)
- Security incidents related to AI tools (hopefully zero)
- Adoption rate (% of team actively using tools)

### Pilot Report Template

```markdown
## AI Tool Pilot Report

### Team and Duration
- Team: [name]
- Duration: [dates]
- Project: [name and description]

### Tools Evaluated
- [Tool 1]: [license tier, configuration]
- [Tool 2]: [license tier, configuration]

### Quantitative Results
- Developer satisfaction: [X/5 average]
- Estimated time savings: [X% on targeted tasks]
- AI-related incidents: [count]
- Adoption rate: [X% of team using daily]

### Qualitative Findings
- What worked well: [list]
- What didn't work: [list]
- Surprises: [list]

### Recommendations for Phase 2
- Tools to proceed with: [list]
- Standards to establish: [list]
- Training needs: [list]
- Risks to mitigate: [list]

### Artifacts Created
- Instruction file: [link]
- Prompt library: [link]
- Review checklist: [link]
- Onboarding guide: [link]
```

---

## Phase 2: Expand (8-12 Weeks)

**Goal:** Share pilot learnings with 3-5 teams, adapt standards, build a champion network.

### Team Selection
- Choose teams that expressed interest during the pilot
- Include at least one team with a different tech stack
- Include one team with higher security requirements
- Assign a pilot team member as advisor to each new team

### Activities

| Week | Activity |
|---|---|
| 1-2 | Pilot team presents learnings, new teams do tool setup |
| 3-4 | Adapted onboarding for each team, champion identified per team |
| 5-6 | Cross-team standup (weekly, 15 min), shared prompt contributions |
| 7-8 | Standards refinement based on multi-team feedback |
| 9-10 | Champion network established, regular meetings |
| 11-12 | Phase 2 retrospective, preparation for org-wide scale |

### Adapting Standards
The pilot team's standards won't work perfectly for every team. Expect to:

- **Keep:** Core security rules, review process, quality gates
- **Adapt:** Naming conventions, file organization, tool-specific configuration
- **Add:** Tech-stack-specific rules for different teams
- **Remove:** Rules that proved unnecessary or counterproductive

### Building the Champion Network
- Identify one champion per expanding team
- Champions meet bi-weekly to share learnings
- Champions have dedicated time (10-20%) for AI workflow activities
- See [Building a Champion Network](champion-network.md) for full details

---

## Phase 3: Scale (12-16 Weeks)

**Goal:** All teams adopt AI tools with organization-wide standards, training program, and governance.

### Prerequisites
Before scaling to all teams:
- [ ] Champion network is active and effective
- [ ] Base standards are documented and validated across 3+ teams
- [ ] Training program exists (not just documentation)
- [ ] Security team has approved tool configuration
- [ ] Budget is approved for licenses
- [ ] IT has provisioned tool access and SSO

### Activities

| Week | Activity |
|---|---|
| 1-4 | Organization-wide training rollout (cohort-based) |
| 5-8 | Team-by-team onboarding with champion support |
| 9-12 | Standardization of review processes across all teams |
| 13-16 | Governance framework finalized, metrics baseline established |

### Rollout Order
Not all teams adopt simultaneously. Prioritize:

1. **High-readiness teams** — willing, have a champion, lower risk
2. **High-impact teams** — where AI tools will have the most benefit
3. **Skeptical teams** — need extra support, champion mentoring
4. **Compliance-sensitive teams** — need additional governance review

### Training Program Structure

```
Track 1: Developer Fundamentals (all developers)
- Day 1: Tool setup and first use
- Day 2-3: Instruction files, prompt patterns, team standards
- Day 4-5: Review process, testing with AI, independent practice

Track 2: Champion Training (one per team)
- Session 1: Facilitating adoption on your team
- Session 2: Creating and maintaining instruction files
- Session 3: Measuring and improving AI outcomes
- Session 4: Mentoring and troubleshooting

Track 3: Leadership Briefing (managers, directors)
- Session 1: AI tools landscape and business case
- Session 2: Governance, compliance, and risk
- Session 3: Measuring ROI and setting expectations
```

---

## Phase 4: Optimize (Ongoing)

**Goal:** Metrics-driven improvement, custom tooling, advanced patterns.

### Activities
- Monthly metrics review and reporting
- Quarterly standards review and update
- Continuous prompt library improvement
- Custom skill and agent development
- Cross-team knowledge sharing events
- Tool evaluation for new capabilities

### Optimization Areas

| Area | Actions |
|---|---|
| Quality | Track and reduce AI-related bug rate |
| Velocity | Measure and improve time savings |
| Satisfaction | Regular developer surveys |
| Security | Continuous security posture monitoring |
| Cost | License optimization, usage patterns |
| Innovation | Encourage experimentation with new capabilities |

---

## Key Success Factors

1. **Executive sponsorship** — someone with authority and budget supports this
2. **Budget for training** — not just licenses, but time for developers to learn
3. **Clear success metrics** — defined before the pilot starts
4. **Feedback channels** — easy way for developers to share issues and wins
5. **Patience** — benefits compound over months, not days
6. **Flexibility** — be willing to adapt the plan based on feedback

---

## Risks and Mitigations

| Phase | Risk | Mitigation |
|---|---|---|
| Pilot | Pilot team is too enthusiastic (biased results) | Include skeptics in pilot, measure objectively |
| Pilot | Tool doesn't work for the team's tech stack | Evaluate multiple tools, have a backup plan |
| Expand | Standards don't transfer to other teams | Involve expanding teams in standards revision |
| Expand | Champions burn out | Limit commitment to 10-20%, rotate if needed |
| Scale | Resistance from teams not involved in pilot | Include resistant voices early, address concerns |
| Scale | Security/compliance blockers discovered late | Involve security team from Phase 1 |
| Optimize | Metric gaming (optimizing for numbers, not outcomes) | Use qualitative feedback alongside metrics |
| All | Tool costs escalate faster than value | Track per-developer cost vs. productivity gain |

---

## Rollout Timeline Template

```
Month 1-3:   PILOT
             └── 1 team, low-risk project
             └── Create base standards
             └── Collect metrics

Month 4-6:   EXPAND  
             └── 3-5 teams
             └── Adapt standards
             └── Build champion network

Month 7-10:  SCALE
             └── All teams
             └── Training program
             └── Governance framework

Month 11+:   OPTIMIZE
             └── Metrics-driven improvement
             └── Custom tooling
             └── Continuous evolution
```

## Next Steps

- [Build a champion network →](champion-network.md)
- [Maintain cross-team consistency →](cross-team-consistency.md)
- [Address enterprise governance →](enterprise-considerations.md)
- [Assess your team's maturity →](../01-fundamentals/maturity-model.md)
