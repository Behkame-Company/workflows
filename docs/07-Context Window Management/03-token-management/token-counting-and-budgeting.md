# Token Counting & Budgeting

> How to measure, plan, and allocate your token budget for AI-assisted development.

---

## Why Count Tokens?

Without counting, you're driving blind:
- You don't know how close you are to the limit
- You can't predict when compaction is needed
- You can't optimize what you can't measure
- Exceeding the window causes errors or truncation

---

## Token Counting Methods

### 1. API-Based Counting (Precise)
```python
# Anthropic token counting API
response = client.messages.count_tokens(
    model="claude-sonnet-4-6",
    messages=[{"role": "user", "content": prompt}],
    system=system_prompt,
    tools=tool_definitions
)
print(f"Input tokens: {response.input_tokens}")
```

### 2. Estimation Rules of Thumb
| Content Type | Estimation |
|-------------|-----------|
| English text | 1 token per 4 characters |
| Code | 1 token per 3.5 characters |
| JSON | 1 token per 3 characters |
| Markdown | 1 token per 3.8 characters |

### 3. Tiktoken Library (OpenAI Models)
```python
import tiktoken
encoder = tiktoken.encoding_for_model("gpt-4")
tokens = encoder.encode("your text here")
print(f"Token count: {len(tokens)}")
```

---

## Token Budget Planning

### Budget Allocation Template

```
Total Context Window:        200,000 tokens
─────────────────────────────────────────
System Prompt:                 2,000 tokens (1%)
Tool Definitions:              3,000 tokens (1.5%)
Injected Context:             15,000 tokens (7.5%)
Conversation History:        150,000 tokens (75%)
Response Budget:              16,000 tokens (8%)
Safety Buffer:                14,000 tokens (7%)
─────────────────────────────────────────
Compaction Trigger:    at    150,000 tokens (75%)
```

### Budget Rules
1. **Reserve 8-10% for output** — Don't fill the window completely
2. **Set compaction trigger at 75%** — Leave headroom for the last turn
3. **Track accumulation rate** — Tokens per turn helps predict when you'll hit limits
4. **Tool results need the most budget** — Allocate 60-70% for conversation + tools

---

## Monitoring Token Usage

### Per-Turn Tracking
```
Turn 1: +500 input, +200 output = 700 total
Turn 2: +800 input (file read), +300 output = 1,800 cumulative
Turn 3: +2,000 input (search results), +500 output = 4,300 cumulative
...
Turn 20: Approaching 80,000 tokens → consider compaction
```

### Warning Thresholds
| Threshold | Action |
|-----------|--------|
| 50% used | Normal operation, no action |
| 75% used | ⚠️ Consider compaction or clearing old tool results |
| 85% used | 🔶 Compact now or start wrapping up |
| 95% used | ❌ Critical — risk of truncation or error |

---

## Key Takeaways

1. **Measure tokens, don't guess** — Use APIs or libraries for precision
2. **Budget before you start** — Allocate space for each component
3. **Reserve output space** — Don't fill the window completely
4. **Compact at 75%** — Before you run out, not after
5. **Track per-turn growth** — Predict when you'll need action

---

## Next Steps

- 🔗 [Cost Optimization](cost-optimization.md) — Reducing token spend
- 🔗 [Context Awareness](context-awareness.md) — Models that self-manage
