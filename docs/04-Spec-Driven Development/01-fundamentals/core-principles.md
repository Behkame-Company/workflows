# Core Principles of Spec-Driven Development

> The 7 principles that make SDD work — violate any one and the process breaks down.

---

## Principle 1: Specification as Source of Truth

The spec is **the** authoritative description of what the software should do. Not the code, not the tests, not the Jira tickets — the spec.

```
❌  "The code IS the documentation"
✅  "The spec defines the intent; the code implements it"
```

### What This Means in Practice
- Specs are **version-controlled** alongside code
- When code and spec disagree, the spec is investigated first
- Specs are **reviewed** with the same rigor as pull requests
- Specs are **referenced** in commit messages and PRs

---

## Principle 2: Intent Over Implementation

Specs describe **what** and **why** — never **how**.

```markdown
❌  Bad Spec:
"Use a Redis sorted set with ZADD to store session timestamps,
 and run ZRANGEBYSCORE to evict expired sessions every 5 minutes"

✅  Good Spec:
"Sessions expire after 30 minutes of inactivity.
 Expired sessions are cleaned up automatically.
 The system must support 10,000 concurrent sessions."
```

**Why**: When you specify *how*, you constrain the AI's solution space. The AI (or team) may find a better implementation than what you'd prescribe.

**Exception**: When a specific technology choice is a hard requirement (compliance, existing infrastructure), state it as a **constraint**, not an implementation detail.

---

## Principle 3: Multi-Step Refinement

Good specs don't appear in one draft. SDD mandates iterative refinement:

```
Draft 1:  "User authentication system"
Draft 2:  "User auth with email/password, OAuth2, 2FA support"
Draft 3:  Add acceptance criteria, edge cases, error states
Draft 4:  Technical constraints, performance requirements
Draft 5:  Team review, final sign-off
```

### The Clarification Step
Between Specify and Plan, the **Clarify** step is critical:
- What happens when X fails?
- Is there a rate limit?
- How does this interact with existing feature Y?
- What are the data constraints?

---

## Principle 4: Living Specifications

Specs are **not** write-once documents. They evolve:

```
Feature lifecycle:
  v1.0 spec → v1.0 implementation → learnings
  v1.1 spec (updated with learnings) → v1.1 implementation → ...
```

### Keeping Specs Alive
- Update spec when requirements change (before changing code)
- Add discovered edge cases to the spec retroactively
- Review spec accuracy during retrospectives
- Automate drift detection where possible

---

## Principle 5: Appropriate Granularity

Specs should be **right-sized** — detailed enough to be unambiguous, brief enough to be useful.

| Spec Level | When to Use | Typical Length |
|------------|-------------|---------------|
| **Feature spec** | New capability | 50–200 lines |
| **Enhancement spec** | Change to existing feature | 20–50 lines |
| **Bug fix spec** | Not needed | Skip — use issue description |
| **System spec** | Cross-cutting concern | 100–300 lines |
| **API contract spec** | New API | 30–100 lines |

### The "Goldilocks" Rule
```
❌  Too little:  "Build auth"
❌  Too much:   300-line spec for a color change
✅  Just right:  Spec covers requirements, acceptance criteria, and edge cases
                 for the actual complexity of the work
```

---

## Principle 6: Human-in-the-Loop Validation

AI agents can draft specs, generate plans, and implement code — but **humans must validate at every phase transition**:

```
Specify  ──[human review]──►  Plan
Plan     ──[human review]──►  Tasks
Tasks    ──[human review]──►  Implement
Implement ──[human review]──► Merge
```

### What Humans Validate
| Phase | Human Checks |
|-------|-------------|
| Specify → Plan | Does the plan address all requirements? |
| Plan → Tasks | Are tasks complete and properly scoped? |
| Tasks → Implement | Does implementation match spec intent? |
| Implement → Merge | Does code pass acceptance criteria? |

---

## Principle 7: Specs Are Testable

Every requirement in a spec must be **independently verifiable**:

```markdown
❌  Untestable:  "The system should be fast"
✅  Testable:    "API responses return within 200ms at p95 under 1000 QPS"

❌  Untestable:  "The UI should look nice"
✅  Testable:    "The login page matches the Figma design within 2px tolerance"

❌  Untestable:  "Handle errors gracefully"
✅  Testable:    "On network failure, show retry button with 'Connection lost' message"
```

### The Acceptance Criteria Test

For each requirement, ask: **"How would I write a test for this?"**

If you can't answer that question, the requirement is too vague.

---

## Principles Summary

| # | Principle | One-Liner |
|---|-----------|-----------|
| 1 | Source of Truth | Spec is authoritative, not code |
| 2 | Intent Over Implementation | Describe *what*, not *how* |
| 3 | Multi-Step Refinement | Iterate specs through drafts |
| 4 | Living Specifications | Update specs as you learn |
| 5 | Appropriate Granularity | Right-size for complexity |
| 6 | Human-in-the-Loop | Humans validate phase transitions |
| 7 | Testable Requirements | Every requirement must be verifiable |

---

*Next: [The SDD Mindset →](the-sdd-mindset.md)*
