# Keeping Specs Alive

> Preventing specification rot and code-spec divergence.

---

## The Spec Decay Problem

Specs decay the moment implementation begins. Without active maintenance:

```
Week 1:   Spec and code are 100% aligned ✅
Week 4:   "Minor" deviations during implementation (spec: 95% accurate)
Month 3:  Several undocumented changes (spec: 70% accurate)
Month 6:  Spec is misleading fiction (spec: 40% accurate)
Year 1:   Nobody reads the spec — it's "that old document" (spec: useless)
```

---

## Strategies to Keep Specs Current

### Strategy 1: Update Spec First, Then Code

When requirements change, update the spec **before** changing code:

```
❌  Flow that causes drift:
    Slack request → Code change → "We should update the spec" → Never happens

✅  Flow that prevents drift:
    Slack request → Update spec → Review spec change → Change code
```

### Strategy 2: Spec Diffs in PRs

Require spec updates in PRs that change behavior:

```markdown
# PR: Change session timeout from 30 to 60 days

## Changed Files
- src/config/auth.ts (session timeout constant)
- .specify/specs/auth/spec.md (updated AC-2.3 from "30 days" to "60 days")

⚠️ PRs that change behavior without updating the spec should be flagged in review.
```

### Strategy 3: Automated Drift Detection

```yaml
# .github/workflows/spec-drift-check.yml
on:
  pull_request:
jobs:
  check-spec-drift:
    steps:
      - name: Check if behavioral changes update specs
        run: |
          # If src/ files changed but .specify/ files didn't,
          # add a review reminder
          changed_src=$(git diff --name-only origin/main -- 'src/')
          changed_specs=$(git diff --name-only origin/main -- '.specify/')
          if [ -n "$changed_src" ] && [ -z "$changed_specs" ]; then
            echo "::warning::Source files changed without spec updates. Verify no behavioral changes."
          fi
```

### Strategy 4: Quarterly Spec Audits

Schedule regular reviews:

```markdown
## Quarterly Spec Audit — Q1 2026

| Feature | Spec Status | Action Needed |
|---------|-------------|---------------|
| User Auth | ✅ Current | None |
| Notifications | ⚠️ Stale | Update: push notifications added but not in spec |
| Cart | ❌ Outdated | Major rewrite: discount system completely changed |
| Search | ✅ Current | None |
```

### Strategy 5: Spec-Code Cross-References

Make specs easy to find from code and vice versa:

```typescript
/**
 * @spec .specify/specs/notifications/spec.md §3.2
 * @acceptance AC-3.2: Notifications are rate-limited to 100/hour/user
 */
export class NotificationRateLimiter {
```

```markdown
# spec.md §3.2 Rate Limiting

**Implementation**: src/services/NotificationRateLimiter.ts
**Tests**: src/services/__tests__/NotificationRateLimiter.test.ts
```

---

## Measuring Spec Health

### Spec Freshness Score

```
Score = (Spec last updated) - (Code last changed)

Green:   Spec updated within same PR or same week    → Healthy
Yellow:  Spec 1-4 weeks behind code changes           → Needs attention
Red:     Spec 1+ months behind code changes            → Spec rot
```

### Coverage Metrics

Track what percentage of features have active specs:
```
Total features:           25
Features with specs:      20  (80%)
Specs updated this quarter: 15  (75% of specs)
Spec accuracy (audited):  18/20 current (90%)
```

---

## When Specs Should NOT Be Updated

Not every code change requires a spec update:

| Change Type | Spec Update? |
|-------------|-------------|
| Behavior change | ✅ Always |
| New edge case handling | ✅ Always |
| Performance optimization | ⚠️ Only if spec has performance criteria |
| Refactoring (same behavior) | ❌ No |
| Dependency update | ❌ No |
| Bug fix (original spec was correct) | ❌ No |
| Bug fix (spec was wrong) | ✅ Fix the spec |

---

*Next: [Team Collaboration →](team-collaboration.md)*
