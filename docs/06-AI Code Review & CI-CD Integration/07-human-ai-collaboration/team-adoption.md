# Team Adoption

> How to roll out AI code review to your team with minimal friction and maximum buy-in.

---

## The 4-Phase Rollout

### Phase 1: Shadow Mode (Week 1-2)
AI reviews run but **don't block merges**:
- Enable AI review as non-required status check
- Team sees AI feedback but isn't blocked
- Collect data on false positive rate and signal quality
- No workflow changes required

### Phase 2: Advisory Mode (Week 3-4)
AI reviews are visible and expected, but **still don't block**:
- Encourage team to address AI findings
- Track how many suggestions are applied
- Gather team feedback on noise vs. signal
- Tune custom instructions based on feedback

### Phase 3: Gate Mode (Month 2)
AI review becomes a **required status check for non-critical findings**:
- Block merge on critical/high AI findings
- Warn on medium findings
- Team must address or dismiss with rationale
- Continue tuning instructions

### Phase 4: Full Integration (Month 3+)
AI review is fully integrated into the workflow:
- Required status check for all PRs
- Custom instructions refined from team feedback
- Metrics dashboard active
- Periodic review of effectiveness

---

## Change Management

### Common Objections and Responses

| Objection | Response |
|-----------|---------|
| "AI review is too noisy" | Start in "chill" mode; tune custom instructions |
| "It slows down merging" | AI review adds seconds, not hours; it speeds up human review |
| "I don't trust AI to review my code" | It's a first pass, not a final decision; you still approve |
| "It'll replace human reviewers" | It handles mechanical checks so humans focus on architecture |
| "Too many false positives" | Track and tune; target <20% false positive rate |

### Building Champions
1. **Identify early adopters** — Developers who are AI-curious
2. **Share wins** — "AI caught this bug before it shipped"
3. **Celebrate savings** — "Review time dropped 35% this month"
4. **Iterate on feedback** — Adjust instructions based on team input

---

## Training Checklist

- [ ] Team understands what AI review checks
- [ ] Team knows how to apply AI suggestions (one-click)
- [ ] Team knows how to dismiss findings with rationale
- [ ] Custom instructions reflect team standards
- [ ] Escalation protocol is documented
- [ ] Metrics dashboard is accessible

---

## Key Takeaways

1. **4 phases**: Shadow → Advisory → Gate → Full integration
2. **Start non-blocking** — Build trust before enforcing
3. **Tune continuously** — Custom instructions improve over time
4. **Address objections proactively** — Have responses ready
5. **Measure and share wins** — Data drives adoption
