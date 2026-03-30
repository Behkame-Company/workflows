# Reflection & Critique

> Teaching the model to evaluate and improve its own output.

---

## The Pattern

**Generate → Critique → Refine**: The most practical advanced pattern for development tasks.

```
Step 1: Generate initial solution
Step 2: Critique the solution against criteria
Step 3: Refine based on the critique
```

---

## Why It Works

Models catch their own errors when explicitly asked to review. The critique step activates different knowledge than the generation step — similar to how a developer writes code and then reviews it with fresh eyes.

---

## Implementation Patterns

### Single-Prompt Reflection

```
Write a TypeScript function that validates email addresses.

After writing the function:
1. Review it for edge cases you may have missed
2. Check if it handles: empty string, very long input, 
   unicode characters, multiple @ signs
3. If any issues found, provide the corrected version
```

### Two-Step Chained Reflection

**Step 1 — Generate**:
```
Write an Express middleware that rate-limits API requests 
to 100 per minute per user, using an in-memory store.
```

**Step 2 — Critique (feed step 1's output back)**:
```
Review this rate-limiting middleware for:
1. Race conditions in concurrent requests
2. Memory leaks (does the store grow unbounded?)
3. Edge cases (what if user ID is missing?)
4. Accuracy (does the window actually slide?)

For each issue found, provide the corrected code.
```

### Self-Check Prompt

```
Before you give your final answer, verify it against these criteria:
- Does it handle all the edge cases mentioned?
- Is it type-safe (no implicit any)?
- Would it pass a code review?
- Does it follow the single responsibility principle?

If any check fails, fix the issue before responding.
```

Anthropic specifically recommends this approach: *"Ask Claude to self-check. Append something like 'Before you finish, verify your answer against [test criteria].' This catches errors reliably, especially for coding and math."*

---

## Structured Critique Framework

```xml
<critique_criteria>
  <criterion name="correctness">Does it produce the right output for all inputs?</criterion>
  <criterion name="safety">Does it handle errors, null values, and edge cases?</criterion>
  <criterion name="performance">Is it efficient for expected data sizes?</criterion>
  <criterion name="readability">Is it clear and maintainable?</criterion>
  <criterion name="conventions">Does it follow our project patterns?</criterion>
</critique_criteria>

Generate the function, then evaluate against each criterion.
For any criterion that scores below 8/10, revise the implementation.
```

---

## When to Use Reflection

| Task | Reflection Value |
|------|:----------------:|
| Complex algorithm implementation | ★★★ |
| Security-sensitive code | ★★★ |
| API design | ★★★ |
| Simple utility function | ★ |
| Formatting/renaming | ✗ |
| One-line change | ✗ |

**Rule of thumb**: Use reflection when the cost of a bug exceeds the cost of the extra tokens.

---

## Extended Thinking as Implicit Reflection

Modern models with extended thinking (Claude 4.x adaptive thinking) perform reflection internally. When thinking is enabled:

```python
response = client.messages.create(
    model="claude-sonnet-4-6-20250725",
    thinking={"type": "adaptive"},
    messages=[...]
)
```

The model may internally critique and revise before producing output. This is often more effective than manual reflection prompts.
