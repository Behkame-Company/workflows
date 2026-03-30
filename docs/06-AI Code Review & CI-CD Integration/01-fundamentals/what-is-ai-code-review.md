# What Is AI Code Review?

> How large language models are transforming code review from a manual bottleneck into an automated first-pass safety net.

---

## The Problem AI Code Review Solves

Traditional code review has three persistent problems:

1. **Bottleneck**: Senior developers spend 20-30% of their time reviewing others' code
2. **Inconsistency**: Different reviewers catch different issues; standards drift over time
3. **Fatigue**: After reviewing 200-400 lines, human attention drops significantly

AI code review addresses all three by providing an automated, consistent, tireless first pass.

---

## How It Works

```
Developer opens PR
        │
        ▼
┌─────────────────────────────┐
│  AI REVIEWER (First Pass)   │
│                             │
│  • Analyzes diff + context  │
│  • Checks against rules     │
│  • Flags issues by severity │
│  • Suggests fixes           │
│  • Posts PR comments        │
└────────────┬────────────────┘
             │
             ▼
┌─────────────────────────────┐
│  HUMAN REVIEWER (Second)    │
│                             │
│  • Reviews AI findings      │
│  • Focuses on architecture  │
│  • Checks business logic    │
│  • Approves or requests     │
│    changes                  │
└─────────────────────────────┘
```

### What AI Reviews Well
- **Syntax and style** — Formatting, naming conventions, code smells
- **Security vulnerabilities** — SQL injection, XSS, hardcoded secrets
- **Bug patterns** — Null references, off-by-one errors, race conditions
- **Performance** — N+1 queries, unnecessary allocations, missing indexes
- **Test coverage** — Missing tests for new code paths
- **Documentation** — Missing or outdated comments

### What AI Reviews Poorly
- **Business logic correctness** — Does this feature meet the product requirement?
- **Architectural fit** — Does this belong in this module?
- **User experience impact** — Will this confuse end users?
- **Organizational context** — Is this the right time for this change?

---

## The Layered Review Stack

Modern code quality uses multiple layers, each catching different issues:

```
Layer 1: FORMATTERS (Prettier, Black)     → Style consistency
Layer 2: LINTERS (ESLint, Ruff)           → Code smells, simple bugs
Layer 3: STATIC ANALYSIS (CodeQL, Semgrep) → Security, data flow
Layer 4: AI REVIEWERS (Copilot, CodeRabbit) → Context-aware issues
Layer 5: HUMAN REVIEWERS                   → Architecture, business logic
```

**Each layer is complementary, not competing.** AI reviewers don't replace linters — they handle the issues linters can't catch, while linters handle deterministic checks AI shouldn't waste tokens on.

---

## Key Takeaways

1. **AI review is a first pass, not a final pass** — Humans still make the merge decision
2. **Consistent and tireless** — AI catches the same issues every time, at any hour
3. **Layered approach** — Combine formatters + linters + static analysis + AI + human review
4. **Context-aware** — Unlike linters, AI understands intent and can reason about code
5. **Time savings of 30-50%** — Teams report significant reduction in review cycles
