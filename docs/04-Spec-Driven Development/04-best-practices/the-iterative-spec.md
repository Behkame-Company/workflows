# The Iterative Spec

> Evolving specifications through feedback loops — not writing them once.

---

## The Iteration Mindset

Specs are not carved in stone. They're **living drafts** that improve through iteration:

```
Draft 1:  Rough idea → "what do we want?"
Draft 2:  Clarification → "what did we miss?"
Draft 3:  Review → "is this feasible?"
Draft 4:  Refinement → "let's tighten the language"
Draft 5:  Approval → "this is what we're building"
```

---

## The Spec Iteration Lifecycle

| Phase | Time | Owner | Deliverables |
|-------|------|-------|-------------|
| 1. Discovery Draft | 15 min | Author | Summary, 3–5 user stories, initial ACs, known edge cases |
| 2. Clarification | 30 min | Author + Stakeholders | Gap analysis, resolved ambiguities, boundary definitions |
| 3. Technical Review | 30 min | Author + Engineers | Feasibility validation, effort estimate, dependency risks |
| 4. Refinement | 15 min | Author | Tightened language, specific values, finalized scope |
| 5. Approval | 10 min | Stakeholders | All reviewers approve, status → "Approved", ready for Plan |

---

## Spec Versioning Strategy

### When to Create a New Version

```
v1.0 — Initial approved spec
v1.1 — Minor additions (new edge case discovered during implementation)
v1.2 — Clarifications (ambiguity found and resolved)
v2.0 — Major changes (scope expansion, new requirements)
```

### Version History in the Spec

```markdown
## Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-15 | @jane | Initial spec |
| 1.1 | 2026-01-22 | @bob | Added rate limiting edge case |
| 1.2 | 2026-02-01 | @jane | Clarified session expiry behavior |
| 2.0 | 2026-03-10 | @alice | Added push notifications (v2 scope) |
```

---

## Feedback Loops

### Loop 1: Implementation Feedback
```
Implement ─── finds gap ───► Update spec ───► Continue implementation
```

During implementation, spec gaps are discovered. Feed them back:
```markdown
## Implementation Notes (added during development)
- Discovered: WebSocket reconnection needs spec'd behavior
- Resolution: Added AC-3.4 for reconnection with exponential backoff
```

### Loop 2: Testing Feedback
```
Test ─── finds untested scenario ───► Add to spec ───► Add test
```

### Loop 3: User Feedback
```
Deploy ─── user reports unexpected behavior ───► Update spec ───► Fix
```

---

## The "Just Enough" Principle

Iterate specs to **just enough** detail — no more, no less:

```
Iteration 1:  "Users can search products"
              → Not enough for implementation

Iteration 2:  "Users can search products by name, category, and price range.
               Results paginated at 20/page, sorted by relevance."
              → Enough for most features

Iteration 3:  Full spec with edge cases, error handling, performance criteria
              → Needed for complex/critical features
```

### When to Stop Iterating
- [ ] Every acceptance criterion is independently testable
- [ ] Edge cases cover the most likely failure modes
- [ ] A developer who's never seen the feature could implement it
- [ ] An AI agent would know exactly what to build
- [ ] Scope boundaries are explicit

---

## Common Iteration Mistakes

### 1. Analysis Paralysis
```
❌  Spending 3 days perfecting a spec for a 2-hour feature
✅  Matching spec effort to feature complexity (see Spec Sizing guide)
```

### 2. Iteration Without New Information
```
❌  Rewriting the same section 5 times for "better wording"
✅  Each iteration adds new information or resolves specific ambiguity
```

### 3. Iterating Alone
```
❌  Author writes and rewrites in isolation
✅  Each iteration involves different perspectives (product, engineering, QA)
```
