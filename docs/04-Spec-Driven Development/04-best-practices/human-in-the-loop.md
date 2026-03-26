# Human-in-the-Loop Review

> Where humans must validate before AI agents proceed.

---

## The Human Role in SDD

SDD doesn't replace human judgment — it **structures** when and where humans need to intervene. AI agents handle the generation; humans handle the validation.

```
AI Generates ──► Human Validates ──► AI Proceeds
     │                  │                  │
  spec draft       review & approve    plan draft
  plan draft       review & approve    task draft
  task list        review & approve    code
  code             review & merge      done
```

---

## Mandatory Human Checkpoints

### Checkpoint 1: Spec Approval
**Before Plan phase begins.**

| Human Checks | Why It Matters |
|-------------|---------------|
| Are requirements actually needed? | AI can't judge business value |
| Are acceptance criteria correct? | AI may misinterpret business rules |
| Is scope appropriate? | AI tends to include too much |
| Are there regulatory implications? | AI doesn't know your compliance needs |

### Checkpoint 2: Plan Review
**Before Tasks phase begins.**

| Human Checks | Why It Matters |
|-------------|---------------|
| Is the architecture appropriate? | AI may over-engineer |
| Does it integrate with existing code? | AI may not know all constraints |
| Are technology choices justified? | AI may default to popular, not appropriate |
| Are there security implications? | AI may miss threat vectors |

### Checkpoint 3: Task Validation
**Before Implementation begins.**

| Human Checks | Why It Matters |
|-------------|---------------|
| Is task granularity right? | AI may create too many tiny tasks |
| Are dependencies correct? | AI may miss implicit dependencies |
| Is the order optimal? | AI may not know team constraints |

### Checkpoint 4: Code Review
**Before merge.**

| Human Checks | Why It Matters |
|-------------|---------------|
| Does code match spec intent? | AI may implement letter, not spirit |
| Are there security vulnerabilities? | AI-generated code needs security review |
| Is the code maintainable? | AI may produce technically correct but unmaintainable code |
| Do tests actually test the right things? | AI may generate tests that pass but don't validate behavior |

---

## Review Strategies

### Time-Boxed Review
Set a maximum review time based on artifact size:
```
Spec review:  15-30 minutes
Plan review:  15-30 minutes
Task review:  10-15 minutes
Code review:  Scales with code size
```

### Diff Review for Iterations
After the first approved version, review only the changes:
```bash
git diff main..spec/notifications -- .specify/specs/notifications/spec.md
```

### AI-Assisted Review
Use a second AI to review the first AI's work:
```
"Review this spec for completeness and clarity.
Does it have:
- Clear acceptance criteria?
- Documented edge cases?
- Specific (not vague) requirements?
- Defined scope boundaries?
Flag any issues."
```

Then a **human reviews the review** — meta-validation.

---

## When to Override AI Decisions

### Signs the AI Got It Wrong

| Signal | Action |
|--------|--------|
| Plan uses technology not in your stack | Override with correct tech |
| Tasks are in wrong order for your codebase | Reorder based on actual dependencies |
| Code implements spec literally but misses intent | Clarify spec, regenerate |
| Generated tests only cover happy path | Add edge case requirements |
| Architecture is over-engineered for scale | Simplify to match actual needs |

### How to Override

```markdown
## Human Override Notes

### Plan §3.2 — Data Storage
AI suggested: MongoDB for flexible schema
Override: Use PostgreSQL (matches existing stack, team expertise)
Reason: Consistency > flexibility for this feature size

### Task 7 — Caching Layer
AI suggested: Redis caching for all API responses
Override: Skip caching in v1, add when performance data supports it
Reason: Premature optimization
```

---

## Building Review Culture

### Make Review Fast
- Spec reviews shouldn't feel like document reviews
- Use structured templates so reviewers know where to look
- Keep specs concise (right-sized for complexity)

### Make Review Valuable
- Reviewers should focus on **correctness and completeness**, not style
- Use checklists to make reviews systematic
- Track defects found per phase to demonstrate value

### Make Review Collaborative
- Reviews are conversations, not gatekeeping
- "What if...?" questions improve specs
- Disagreements are resolved by clarifying the spec, not by hierarchy

---

*Next: [Keeping Specs Alive →](keeping-specs-alive.md)*
