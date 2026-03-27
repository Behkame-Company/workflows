# Over-Reliance on AI

> The automation bias trap — when humans stop thinking because AI is "checking."

---

## What Is Automation Bias?

Automation bias is the tendency to favor suggestions from automated systems over human judgment, even when the AI is wrong.

**In code review, this manifests as:**
- "AI approved it, so it's fine" (skipping human review)
- "AI didn't flag it, so there's no issue" (false negative blindness)
- Reduced vigilance during human review because "AI already checked"
- Accepting AI suggestions without understanding them

---

## Real-World Symptoms

| Symptom | What's Happening |
|---------|-----------------|
| Human reviewers approve in <2 minutes | They're rubber-stamping, not reviewing |
| "AI didn't catch it" becomes an excuse | Team trusts AI more than it deserves |
| Fewer human comments per review | Humans deferring to AI |
| Bug escape rate increases despite AI review | AI gives false sense of security |
| Nobody can explain what the AI suggested | Blindly accepting without understanding |

---

## What AI Actually Misses

AI code review is bad at:
- **Business logic correctness** — AI doesn't know your domain
- **Architecture decisions** — "Should this be a microservice?" is a human question
- **Race conditions** — Temporal logic is hard for AI
- **Edge cases specific to your users** — AI lacks domain context
- **Performance at scale** — "Works in tests" ≠ "works at 10K RPS"
- **Organizational context** — "We're migrating away from this library"

---

## Prevention Strategies

### 1. Maintain Required Human Reviewer
Never remove the human approval requirement. AI is additive, not replacement:
```yaml
# Branch protection
required_reviews: 1          # Human reviewer required
required_status_checks:
  - ai-review                # AI review also required
```

### 2. Rotate Reviewers
Don't let the same person always rubber-stamp. Fresh eyes catch more:
- Assign reviewers automatically via round-robin
- Require different reviewer for critical paths

### 3. "What Did AI Miss?" Exercise
Periodically review PRs and ask:
- What did the AI reviewer miss?
- What did it flag that wasn't important?
- Document patterns to improve both AI and human reviews

### 4. Red Team Reviews
Monthly exercise where a developer intentionally introduces subtle bugs:
- Does AI catch them?
- Do human reviewers catch them?
- What type of bugs slip through?

---

## Key Takeaways

1. **AI review supplements human review** — Never replaces it
2. **Monitor human review quality** — If reviews get faster, they might be getting worse
3. **Test AI coverage** — Red team exercises reveal gaps
4. **Rotate reviewers** — Prevent rubber-stamping
5. **"AI didn't flag it" is not an excuse** — Humans own the final decision

---

## Next Steps

- 🔗 [Review Theater](review-theater.md) — Reviews that look good but catch nothing
- 🔗 [Trust Framework](../07-human-ai-collaboration/trust-framework.md) — Calibrating trust
