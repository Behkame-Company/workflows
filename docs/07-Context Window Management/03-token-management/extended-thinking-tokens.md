# Extended Thinking Tokens

> How thinking tokens work, how they affect the context window, and how to manage them.

---

## What Are Thinking Tokens?

When extended thinking is enabled, Claude generates internal reasoning tokens before producing the visible response. These tokens:

- **Count as output tokens** (billed at output rates)
- **Count toward the context window** during the current turn
- **Are automatically stripped** from future turns (not carried forward)
- **Can be substantial** — from hundreds to tens of thousands of tokens

---

## How Thinking Affects the Context Window

```
Turn 1:
  Input:     5,000 tokens (prompt + query)
  Thinking:  3,000 tokens (reasoning)         ← Generated, billed
  Output:    1,000 tokens (visible response)
  Window:    9,000 tokens total

Turn 2:
  Input:     6,000 tokens (Turn 1 without thinking + new query)
  Thinking:  2,000 tokens (new reasoning)
  Output:      800 tokens (response)
  Window:    8,800 tokens total

Note: Turn 1's 3,000 thinking tokens are NOT included in Turn 2's input.
```

**Key formula:**
```
Effective window = (input_tokens - previous_thinking_tokens) + current_turn_tokens
```

---

## Adaptive Thinking (Claude 4.6)

Claude 4.6 uses **adaptive thinking** — the model dynamically decides how much to think based on:
- **Query complexity** — Simple questions get little or no thinking
- **Effort parameter** — Higher effort = more thinking
- **Task type** — Coding and math trigger more thinking

### Controlling Thinking

| Control | How | Effect |
|---------|-----|--------|
| **Effort parameter** | `effort: "low"/"medium"/"high"` | Adjust thinking depth |
| **Adaptive thinking** | `thinking: { type: "adaptive" }` | Model self-calibrates |
| **No thinking** | Omit `thinking` parameter | No thinking tokens |

---

## Special Case: Thinking + Tool Use

When thinking is combined with tool calls, thinking tokens must be preserved during the tool use cycle:

```
Turn 1:  User query
         → Thinking (must keep) + Tool call
Turn 2:  Tool result + Thinking block (preserved)
         → Response (thinking can now be dropped)
Turn 3:  Thinking from Turn 1 automatically stripped
```

**Critical**: During tool use, thinking blocks must be passed back with tool results. The API handles this automatically, but don't manually strip thinking blocks mid-tool-cycle.

---

## Cost Implications

| Scenario | Thinking Tokens | Cost Impact |
|----------|----------------|-------------|
| Simple Q&A | 0-100 | Negligible |
| Code generation | 500-3,000 | Moderate |
| Complex debugging | 2,000-10,000 | Significant |
| Architecture analysis | 5,000-20,000 | High |

**Optimization**: Use lower effort for straightforward tasks. Reserve high effort for complex reasoning.

---

## Key Takeaways

1. **Thinking tokens cost output rates** — Most expensive token type
2. **Automatically stripped between turns** — Don't accumulate in context
3. **Must preserve during tool cycles** — Don't manually strip mid-tool-use
4. **Adaptive thinking self-calibrates** — Use `effort` to adjust
5. **Complex tasks benefit, simple don't** — Match thinking to task complexity

---

## Next Steps

- 🔗 [Context Awareness](context-awareness.md) — Models that track their budget
- 🔗 [Cost Optimization](cost-optimization.md) — Reducing spend
