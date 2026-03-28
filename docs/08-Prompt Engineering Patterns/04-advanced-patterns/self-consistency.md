# Self-Consistency

> Multiple reasoning paths with majority voting — improving reliability for complex tasks.

---

## What Is Self-Consistency?

Self-consistency generates multiple independent answers to the same question, then selects the most common answer. If 3 out of 5 reasoning paths reach the same conclusion, that conclusion is likely correct.

```
Path 1: O(n²) ← 
Path 2: O(n²) ←   Majority: O(n²) ✓
Path 3: O(n²) ←
Path 4: O(n log n)
Path 5: O(n²) ←
```

---

## How It Works

1. **Ask the same question multiple times** (with temperature > 0)
2. **Each response reasons independently** (different CoT paths)
3. **Extract the final answer** from each response
4. **Vote**: Most common answer wins

```python
answers = []
for _ in range(5):
    response = call_model(prompt, temperature=0.7)
    answer = extract_answer(response)
    answers.append(answer)

final_answer = most_common(answers)
confidence = answers.count(final_answer) / len(answers)
```

---

## When to Use Self-Consistency

| Use Case | Why It Helps |
|----------|-------------|
| Algorithm complexity analysis | Different reasoning paths may catch errors |
| Security vulnerability assessment | Multiple perspectives find more issues |
| Code correctness verification | Reduces single-path reasoning errors |
| Architecture trade-off decisions | Reveals which option is more robust |

### When NOT to Use
- Simple, deterministic tasks (waste of tokens)
- Tasks with no clear "correct" answer
- When you need exactly one response (formatting, generation)
- Budget-constrained situations (5x the cost)

---

## Practical Implementation

### For Code Review

```python
def robust_review(code: str, num_paths: int = 3) -> dict:
    issues_found = []
    
    for i in range(num_paths):
        response = call_model(
            f"Review this code for bugs. Path {i+1}:\n{code}",
            temperature=0.5
        )
        issues = parse_issues(response)
        issues_found.append(issues)
    
    # Issues found by majority of paths
    confirmed = [
        issue for issue in all_unique_issues(issues_found)
        if count_paths_finding(issue, issues_found) > num_paths / 2
    ]
    
    return {"confirmed_issues": confirmed, "paths_run": num_paths}
```

### For Yes/No Decisions

```
Run 3 independent analyses:
1. Is this change backward-compatible? Think through it carefully.
2. [Same question, fresh analysis]
3. [Same question, fresh analysis]

If 2+ say "no", flag as breaking change.
```

---

## Self-Consistency vs Other Techniques

| Technique | Approach | Cost | Reliability |
|-----------|----------|------|-------------|
| Single CoT | One reasoning path | 1x | Good |
| Self-Consistency | Multiple paths + voting | 3-5x | Better |
| Tree of Thought | Branching with evaluation | 5-10x | Best for search |
| Reflection | Generate + critique + refine | 2-3x | Good for quality |

---

## Cost-Effective Alternatives

If 5x cost is too high:
1. **Use 3 paths** instead of 5 (still significant improvement)
2. **Only apply to critical decisions** (security, breaking changes)
3. **Use lower-cost models** for the voting paths
4. **Use at temperature 0 twice** — if answers differ, investigate

---

## Next Steps

- 🔗 [Tree of Thought](tree-of-thought.md) — Branching exploration
- 🔗 [Reflection & Critique](reflection-and-critique.md) — Self-evaluation pattern
