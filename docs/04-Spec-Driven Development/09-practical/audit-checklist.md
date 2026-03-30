# SDD Audit Checklist

> Quarterly review of your Spec-Driven Development practice.

---

## How to Use This Checklist

Run this audit quarterly. Score each section honestly. Use the results to identify improvement areas and track progress over time.

---

## Section 1: Spec Coverage (20 points)

| Check | Points | Score |
|-------|--------|-------|
| >80% of production features have specs | 5 | /5 |
| All new features start with a spec (not retrofit) | 5 | /5 |
| Specs exist for cross-cutting concerns (auth, logging, etc.) | 5 | /5 |
| Project has a constitution document | 5 | /5 |
| **Section Total** | **20** | **/20** |

---

## Section 2: Spec Quality (25 points)

| Check | Points | Score |
|-------|--------|-------|
| All specs have testable acceptance criteria | 5 | /5 |
| All specs have documented edge cases (≥3 per spec) | 5 | /5 |
| All specs have explicit scope boundaries (in/out) | 5 | /5 |
| Specs use consistent structure (following templates) | 5 | /5 |
| Non-functional requirements include specific metrics | 5 | /5 |
| **Section Total** | **25** | **/25** |

---

## Section 3: Spec Freshness (20 points)

| Check | Points | Score |
|-------|--------|-------|
| >90% of specs match current code behavior | 5 | /5 |
| Specs updated in same PR as behavioral code changes | 5 | /5 |
| No spec older than 6 months without review | 5 | /5 |
| Spec changes have git history with clear commit messages | 5 | /5 |
| **Section Total** | **20** | **/20** |

---

## Section 4: Workflow Adherence (20 points)

| Check | Points | Score |
|-------|--------|-------|
| Quality gates exist between all 4 phases | 5 | /5 |
| Spec reviews completed within 1 business day | 5 | /5 |
| Plans trace back to spec requirements | 5 | /5 |
| Tasks have clear acceptance criteria | 5 | /5 |
| **Section Total** | **20** | **/20** |

---

## Section 5: Team Practices (15 points)

| Check | Points | Score |
|-------|--------|-------|
| All team members have written at least 1 spec this quarter | 3 | /3 |
| Spec review comments are constructive (not rubber-stamp) | 3 | /3 |
| Templates are available and used consistently | 3 | /3 |
| New hires learn SDD process within first 2 weeks | 3 | /3 |
| Retrospectives include spec process improvements | 3 | /3 |
| **Section Total** | **15** | **/15** |

---

## Scoring Guide

| Score | Rating | Action |
|-------|--------|--------|
| 90–100 | ★★★★★ Excellent | Maintain and refine; share practices with other teams |
| 75–89 | ★★★★☆ Good | Address weak areas; consider automation |
| 60–74 | ★★★☆☆ Developing | Focus on consistency and adoption |
| 40–59 | ★★☆☆☆ Emerging | Re-train team; simplify process |
| <40 | ★☆☆☆☆ Not Practicing | Restart with lightweight SDD adoption |

---

## Quarterly Improvement Plan

After scoring, pick **2-3 focus areas** for the next quarter:

```markdown
## Q2 2026 SDD Improvement Plan

### Focus Area 1: Spec Freshness (Score: 12/20)
- Action: Add CI check for spec updates when behavior changes
- Owner: @devops-lead
- Target: 18/20 by Q3

### Focus Area 2: Team Practices (Score: 9/15)
- Action: Monthly "spec workshop" for practice and template refinement
- Owner: @tech-lead
- Target: 12/15 by Q3

### Focus Area 3: Spec Quality (Score: 18/25)
- Action: Add edge case requirement to spec review checklist
- Owner: @qa-lead
- Target: 22/25 by Q3
```

---

## Drift Detection Quick Audit

Run this 15-minute spot-check monthly:

```
Pick 3 random specs from .specify/specs/

For each spec:
  1. Read acceptance criteria
  2. Find corresponding code
  3. Verify code matches spec behavior
  4. Check if tests exist for each AC

Results:
  Spec 1: [feature]  — Status: ✅ Current / ⚠️ Minor drift / ❌ Major drift
  Spec 2: [feature]  — Status: ✅ / ⚠️ / ❌
  Spec 3: [feature]  — Status: ✅ / ⚠️ / ❌
```

---

## Historical Tracking

Track audit scores over time:

| Quarter | Coverage | Quality | Freshness | Workflow | Team | Total |
|---------|----------|---------|-----------|----------|------|-------|
| Q1 2026 | /20 | /25 | /20 | /20 | /15 | /100 |
| Q2 2026 | /20 | /25 | /20 | /20 | /15 | /100 |
| Q3 2026 | /20 | /25 | /20 | /20 | /15 | /100 |
| Q4 2026 | /20 | /25 | /20 | /20 | /15 | /100 |

**Target**: Steady improvement quarter over quarter. Don't expect perfection — aim for consistent progress.
