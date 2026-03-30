# Chain-of-Thought Prompting

> Teaching the model to show its work — the breakthrough technique for complex reasoning.

---

## What Is Chain-of-Thought?

Chain-of-Thought (CoT) prompting asks the model to explain its reasoning step by step before giving a final answer. This dramatically improves accuracy on tasks requiring logic, math, or multi-step reasoning.

**Without CoT**:
```
Q: A function has O(n) in a loop that runs O(log n) times. What's the total complexity?
A: O(n)  ← Wrong
```

**With CoT**:
```
Q: A function has O(n) in a loop that runs O(log n) times. 
   What's the total complexity? Think step by step.

A: Let me work through this:
1. The inner operation is O(n)
2. The outer loop runs O(log n) times
3. Total = inner × outer = O(n) × O(log n) = O(n log n)

The total complexity is O(n log n).  ← Correct
```

---

## Why It Works

1. **Forces structured reasoning**: The model can't skip steps
2. **Makes errors visible**: You can see where reasoning goes wrong
3. **Activates relevant knowledge**: Step-by-step triggers deeper pattern matching
4. **Reduces hallucination**: Each step can be verified

---

## Three Flavors of CoT

### 1. Zero-Shot CoT
Simply add "Let's think step by step" or "Think through this carefully."

```
Determine if this regex matches the string "2024-01-15":
/^\d{4}-\d{2}-\d{2}$/

Let's think step by step.
```

### 2. Few-Shot CoT
Provide examples that demonstrate the reasoning process.

```xml
<example>
Q: Does this function handle the empty array case?
function sum(arr) { return arr.reduce((a, b) => a + b) }

Reasoning:
1. reduce() is called on arr
2. No initial value is provided
3. If arr is empty, reduce() without initial value throws TypeError
4. Therefore: No, it does not handle empty arrays.

Answer: No — reduce() with no initial value throws on empty arrays.
</example>

Now analyze: [your code]
```

### 3. Structured CoT with Tags
Use XML tags to separate reasoning from the answer.

```
Analyze this code for race conditions.

<thinking>
[Your step-by-step reasoning here]
</thinking>

<answer>
[Your final conclusion here]
</answer>
```

---

## CoT for Coding Tasks

### Code Review
```
Review this function for bugs. For each potential issue:
1. Identify the suspicious code
2. Explain what could go wrong
3. Describe the conditions that trigger the bug
4. Suggest a fix

Think through each code path carefully before concluding.
```

### Debugging
```
This test is failing with "Expected 5 but received 4".

Walk through the function execution step by step with the test input:
1. What are the initial values?
2. What happens in each iteration?
3. Where does the actual result diverge from expected?
```

### Architecture Decisions
```
We need to choose between REST and GraphQL for our API.

Think through these aspects:
1. Our data access patterns (nested vs flat)
2. Number of client types (web, mobile, third-party)
3. Caching requirements
4. Team experience
5. Performance requirements

Then recommend one with reasoning.
```

---

## When to Use CoT

| Task | CoT Needed? | Why |
|------|:-----------:|-----|
| Simple code generation | ❌ | Model knows patterns |
| Complex algorithm | ✅ | Multi-step logic |
| Bug diagnosis | ✅ | Need to trace execution |
| Code formatting | ❌ | Pattern matching task |
| Architecture review | ✅ | Trade-off analysis |
| Variable renaming | ❌ | Simple transformation |
| Performance analysis | ✅ | Requires counting operations |

**Rule of thumb**: If a human would need scratch paper, use CoT.

---

## Extended Thinking (Modern Approach)

Modern models (Claude 4.x) have built-in "thinking" capability that's more powerful than manual CoT:

```python
# Anthropic API — adaptive thinking
response = client.messages.create(
    model="claude-sonnet-4-6-20250725",
    thinking={"type": "adaptive"},
    messages=[{"role": "user", "content": "Analyze this complex algorithm..."}]
)
```

**Adaptive thinking** lets the model decide how much to think based on complexity. For simple questions, it responds directly. For complex ones, it reasons extensively.

### When to Use Manual CoT vs Extended Thinking

| Scenario | Approach |
|----------|----------|
| API with thinking support | Use extended thinking |
| Copilot Chat / CLI | Manual CoT ("think step by step") |
| Need to see reasoning | Manual CoT with `<thinking>` tags |
| Quick simple tasks | Neither — just ask directly |
