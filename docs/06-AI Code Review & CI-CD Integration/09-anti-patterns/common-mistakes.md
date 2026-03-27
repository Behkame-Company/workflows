# Common Mistakes

> The top 10 mistakes teams make when adopting AI code review and CI/CD.

---

## Mistake #1: All-or-Nothing Adoption

**What happens**: Team enables AI review with full enforcement on day one.  
**Why it fails**: High false positive rate → developers revolt → AI review disabled.  
**Fix**: Shadow mode first, gate mode after tuning. See [Incremental Adoption](../08-best-practices/incremental-adoption.md).

---

## Mistake #2: No Custom Instructions

**What happens**: Team uses AI reviewer with default settings.  
**Why it fails**: AI doesn't know your conventions, tech stack, or review priorities.  
**Fix**: Write project-specific `copilot-instructions.md` on day one.

---

## Mistake #3: Duplicating Linter Checks

**What happens**: AI comments on formatting, import order, unused variables.  
**Why it fails**: Linter already catches these → AI adds noise, not value.  
**Fix**: Tell AI to ignore linter-covered rules. See [Signal-to-Noise](../08-best-practices/signal-to-noise.md).

---

## Mistake #4: Ignoring AI Findings Without Rationale

**What happens**: Developers dismiss AI comments with no explanation.  
**Why it fails**: No learning loop → AI never improves → team never justifies overrides.  
**Fix**: Require rationale when dismissing findings.

---

## Mistake #5: No Metrics or Baseline

**What happens**: Team adopts AI review but doesn't measure impact.  
**Why it fails**: Can't prove value → management questions ROI → tool removed.  
**Fix**: Measure baseline before adoption. See [Metrics & KPIs](../08-best-practices/metrics-and-kpis.md).

---

## Mistake #6: Auto-Merging AI-Fixed PRs

**What happens**: Self-healing CI auto-fixes and auto-merges without human review.  
**Why it fails**: AI fix may be syntactically correct but semantically wrong → bugs ship.  
**Fix**: AI can fix and push, but never auto-merge. Always require human approval.

---

## Mistake #7: Treating AI Review as Complete Review

**What happens**: "AI approved it, so it's good to merge."  
**Why it fails**: AI misses architecture issues, business logic errors, and context.  
**Fix**: AI review is first pass, not final approval. See [Over-Reliance](over-reliance-on-ai.md).

---

## Mistake #8: Too Many AI Review Tools

**What happens**: Copilot + CodeRabbit + PR-Agent + custom bot all reviewing.  
**Why it fails**: Conflicting feedback, comment spam, developer confusion.  
**Fix**: One primary AI reviewer, one static analysis tool, one human reviewer.

---

## Mistake #9: Not Reviewing Generated Tests

**What happens**: AI generates tests, they pass, team assumes they're good.  
**Why it fails**: Tests may have weak assertions or test implementation, not behavior.  
**Fix**: Mutation testing + human review of generated tests.

---

## Mistake #10: Skipping CI on "Small" Changes

**What happens**: Developer thinks "it's just a typo fix" and merges without CI.  
**Why it fails**: Even small changes can break things. AI-generated code especially.  
**Fix**: All PRs go through CI. No exceptions.

---

## Key Takeaways

- Start gradual, measure everything, tune continuously
- AI is a tool, not a replacement for human judgment
- One AI reviewer is enough — don't stack them
- Never auto-merge AI fixes
- Always review AI-generated tests

---

## Next Steps

- 🔗 [Over-Reliance on AI](over-reliance-on-ai.md) — The automation bias trap
- 🔗 [Review Theater](review-theater.md) — Reviews that catch nothing
