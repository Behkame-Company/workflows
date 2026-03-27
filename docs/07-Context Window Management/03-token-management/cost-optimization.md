# Cost Optimization

> Practical strategies to reduce token costs without sacrificing quality.

---

## Understanding Token Costs

| Token Type | Relative Cost | Example |
|-----------|--------------|---------|
| **Cached input** | Cheapest (~90% off) | System prompts reused across turns |
| **Standard input** | Base rate | User messages, tool results |
| **Output** | 3-5x input cost | Model responses, tool calls |
| **Thinking** | Same as output | Extended thinking (Claude) |

**Key insight**: Output tokens cost 3-5x more than input tokens. Keeping responses concise saves the most money.

---

## Cost Reduction Strategies

### 1. Prompt Caching
Cache static content that doesn't change between turns:
```
System prompt:     Cache ✅ (same every turn)
Tool definitions:  Cache ✅ (same every turn)
Conversation:      Can't cache (changes each turn)
```

Caching the system prompt alone can reduce input costs by 10-30%.

### 2. Concise Instructions
```
❌ Verbose (200 tokens):
"When you encounter a situation where the user requests..."

✅ Concise (50 tokens):
"For user requests: validate input, check permissions, respond."
```

### 3. Targeted Tool Results
```
❌ Read entire file: 1,500 tokens
✅ Read specific lines: 200 tokens
   Savings: 87%

❌ Full search results: 5,000 tokens
✅ File paths + snippets: 800 tokens
   Savings: 84%
```

### 4. Compress Conversation History
Clear or summarize old turns that are no longer relevant:
```
Turn 1-5: Discussed authentication approach → Summarize to 100 tokens
Turn 6-10: Implemented JWT module → Keep (recent, relevant)
```

### 5. Use Smaller Models for Simple Tasks
| Task | Model | Cost |
|------|-------|------|
| Code formatting | Haiku | $ |
| Code generation | Sonnet | $$ |
| Architecture review | Opus | $$$$ |

Route tasks to the cheapest model that can handle them.

---

## Cost Monitoring

Track these metrics:
- **Tokens per task** — How many tokens does a typical feature take?
- **Cost per PR** — What does AI review cost per pull request?
- **Waste ratio** — What percentage of tokens are low-signal?
- **Cache hit rate** — Are you caching effectively?

---

## Key Takeaways

1. **Output is 3-5x more expensive** — Keep responses concise
2. **Cache static content** — System prompts, tool definitions
3. **Read less, not more** — Targeted retrieval saves tokens and money
4. **Route by complexity** — Cheap models for simple tasks
5. **Measure waste** — Track and reduce low-signal tokens

---

## Next Steps

- 🔗 [Extended Thinking Tokens](extended-thinking-tokens.md) — Thinking budget
- 🔗 [Context Awareness](context-awareness.md) — Self-managing models
