# Waterfall in Disguise

> When SDD becomes big-design-up-front — and how to avoid it.

---

## The Criticism

The #1 criticism of Spec-Driven Development:

> "SDD is just waterfall with Markdown files."

This criticism is **valid when SDD is practiced badly** and **invalid when practiced correctly**.

---

## The Waterfall Trap

### How Teams Fall Into It

```
Waterfall-SDD (Bad):
  Month 1:  Write exhaustive specs for everything
  Month 2:  Architecture review board approves
  Month 3:  Task decomposition meetings
  Month 4:  Finally start coding
  Month 5:  Requirements changed — specs are obsolete
  Month 6:  Start over or ignore specs entirely
```

### The Warning Signs

| Signal | You're Doing Waterfall |
|--------|----------------------|
| Specs take weeks to approve | ❌ |
| No code written until all specs are "done" | ❌ |
| Spec changes require a formal change request | ❌ |
| Specs are treated as binding contracts | ❌ |
| Implementation reveals the spec was wrong, but it's "too late" | ❌ |
| Spec reviews have 10+ attendees | ❌ |
| Your spec process has a Gantt chart | ❌ |

---

## How SDD Differs From Waterfall

| Dimension | Waterfall | Proper SDD |
|-----------|----------|------------|
| **Spec timing** | Everything upfront | Just-in-time per feature |
| **Spec size** | Complete system | Single feature |
| **Spec mutability** | Change-controlled | Living document |
| **Feedback loops** | At the end | At every phase boundary |
| **Iteration** | None until "complete" | Continuous within phases |
| **Duration per cycle** | Months | Hours to days |
| **Who reviews** | Review board | 1-2 peers |

---

## The "Just-In-Time Spec" Principle

The key differentiator: **spec only what you're about to build**.

```
Waterfall:
  Spec Feature A + B + C + D + E → Build A → Build B → ...
  ❌ Specs B-E are stale by the time you get to them

SDD:
  Spec Feature A → Build A → Spec Feature B → Build B → ...
  ✅ Each spec is fresh and informed by what you learned
```

### The One-Sprint Rule
A spec should describe work completable in one sprint (1-2 weeks). If it's bigger, decompose it.

---

## Avoiding the Trap

### Practice 1: Time-Box Specification
```
Small feature:   30 minutes max for spec
Medium feature:  2 hours max for spec
Large feature:   1 day max, then decompose
```

If you've been writing a spec for a week, you're doing waterfall.

### Practice 2: Approve with "Good Enough"
Specs don't need to be perfect. They need to be **good enough to start**:
```
Good enough = All acceptance criteria are testable
              + Edge cases for likely failures
              + Clear scope boundaries
```

### Practice 3: Parallel Phases
Don't wait for all specs before starting work:
```
Week 1:  Spec Feature A → Start Plan A
Week 1:  Plan A done → Start Tasks A + Spec Feature B (parallel)
Week 2:  Implement A + Plan B (parallel)
Week 2:  Feature A done → Spec Feature C
```

### Practice 4: Embrace Spec Changes
```
"The spec changed" is NORMAL, not a failure.

Process:
  1. Document why the spec changed
  2. Update the spec
  3. Assess impact on plan/tasks
  4. Continue with updated context
```

### Practice 5: Short Feedback Loops
```
✅  Spec → Plan → Tasks → Implement (days)
❌  Spec (2 weeks) → Plan (1 week) → Tasks (3 days) → Implement (2 weeks)
```

---

## The Spectrum

SDD exists on a spectrum between chaos and waterfall:

```
Vibe Coding                    Proper SDD                     Waterfall
     │                              │                              │
     ▼                              ▼                              ▼
No specs                    Right-sized specs              Exhaustive specs
No planning                 Iterative planning             Upfront planning
No review                   Peer review                    Board review
Fast & messy                Fast & structured              Slow & rigid
```

**Sweet spot**: Just enough specification to reduce ambiguity without slowing delivery.

---

## Red Flags in Your SDD Practice

Ask these questions monthly:

| Question | If Yes → |
|----------|----------|
| Are specs gathering dust because requirements changed? | Specs are too detailed or written too early |
| Does specifying take longer than implementing? | Over-specifying — simplify |
| Do developers bypass specs and code directly? | Specs aren't providing value — fix the process |
| Are spec reviews a bottleneck? | Reduce ceremony, trust the team |
| Has no spec changed after approval? | Specs may not be "living" enough |
