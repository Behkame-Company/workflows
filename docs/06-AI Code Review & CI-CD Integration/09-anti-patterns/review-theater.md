# Review Theater

> When code reviews look thorough but catch nothing meaningful.

---

## What Is Review Theater?

Review theater is the practice of going through the motions of code review without actually improving code quality. Adding AI review can make this worse if not done correctly.

---

## Common Forms

### 1. The Rubber Stamp
```
Reviewer: "LGTM 👍" (after 30 seconds)
AI: 3 findings posted
Developer: Dismissed all without reading
Result: Nothing was actually reviewed
```

### 2. The Nitpick Review
```
Reviewer: "Can you rename this variable?"
AI: 15 style comments
Developer: Renames variables, fixes formatting
Result: No actual bugs or logic issues examined
```

### 3. The Metric-Chasing Review
```
Team tracks "AI findings addressed" metric
Developer: Creates PR → AI posts 5 comments → addresses all 5
Reality: None of the 5 were meaningful
Result: Metrics look great, quality unchanged
```

### 4. The Confidence Theater
```
Manager: "We have AI + human review on every PR"
Reality: AI catches formatting, human rubber-stamps
Result: Organization feels safe but isn't
```

---

## Signs You Have Review Theater

| Signal | Implication |
|--------|------------|
| Average review time <5 min for large PRs | Reviewers aren't reading code |
| All AI findings are "applied" (100%) | Including wrong ones |
| Zero comments from human reviewers | Humans deferred entirely to AI |
| Defect escape rate hasn't improved | Reviews aren't catching bugs |
| PRs get bigger because "AI will catch issues" | Reduced personal responsibility |

---

## How to Fix It

### 1. Focus AI on Real Issues
Configure AI to catch bugs, security, and logic errors — not style:
```markdown
# copilot-instructions.md
Focus on:
- Logic errors and potential bugs
- Security vulnerabilities
- Missing error handling
- Race conditions

Ignore:
- Style and formatting
- Naming preferences
- Comment quality
```

### 2. Track Meaningful Metrics
Instead of "findings addressed", track:
- Bugs caught in review (before production)
- Security issues prevented
- Human reviewer comments per PR (should be >0)

### 3. Require Substantive Human Comments
Set a cultural norm: human reviewer must add at least one substantive comment or explicitly state what they checked.

### 4. Periodic Audit
Monthly review of random sample of PRs:
- Was the AI review meaningful?
- Did the human reviewer add value?
- Were any issues dismissed inappropriately?

---

## Key Takeaways

1. **AI review can enable theater** — Feels thorough, catches nothing
2. **Track outcomes, not activity** — Bugs caught, not comments made
3. **Human must add value** — Not just click "approve"
4. **Audit periodically** — Sample PRs monthly to check quality
5. **Focus AI on substance** — Bugs over style
