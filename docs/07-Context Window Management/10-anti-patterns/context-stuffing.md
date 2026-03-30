# Context Stuffing

> The #1 anti-pattern — loading everything into context and hoping for the best.

---

## What Is Context Stuffing?

Context stuffing is the practice of dumping as much information as possible into the context window under the assumption that "more context = better results."

```
Developer thought process:
"The model needs to understand my project..."
  → Pastes entire README (2,000 tokens)
  → Pastes all config files (3,000 tokens)
  → Pastes 10 source files (15,000 tokens)
  → Pastes the full error log (5,000 tokens)
  → Pastes the package.json (500 tokens)
  → Asks: "Why is auth broken?"

Total: 25,500 tokens of context
Actually needed: Error message + the auth function (~500 tokens)
```

---

## Why It Fails

### 1. Context Rot
Context rot degrades output quality well before hitting token limits. See [Context Rot](../01-fundamentals/context-rot.md) for details.

### 2. Distraction
Irrelevant context creates false connections:
```
Context includes: payment code + auth code + config files
Model: "The auth issue might be related to the Stripe 
        webhook configuration..." (red herring)
```

### 3. Cost
Input tokens are billed. 25K tokens per request × 100 requests = significant cost for zero benefit.

### 4. Speed
More input tokens = longer inference time. The model processes every token, relevant or not.

---

## Real-World Context Stuffing Examples

| Anti-Pattern | What They Stuff | What They Need |
|-------------|----------------|---------------|
| "Fix this bug" | Entire codebase | Error + buggy function |
| "Add a feature" | All existing features | Pattern to follow + requirements |
| "Review this PR" | All project history | Just the diff + coding standards |
| "Write tests" | All existing tests | Function to test + test patterns |
| "Explain this error" | Full stack trace + all files | Error message + relevant function |

---

## The Fix: Targeted Context

```
❌ Context Stuffing:
   "Here's my entire project: [25,000 tokens]
    Why doesn't auth work?"

✅ Targeted Context:
   "Auth returns 401 for valid credentials.
    Error: 'Invalid signature' in jwt.verify()
    Here's the validateToken function: [200 tokens]
    We recently changed from HS256 to RS256.
    What's wrong?"
```

Same problem. Same fix. 50x less context. Better results.

---

## Key Takeaways

1. **More context ≠ better results** — It often makes results worse
2. **Signal-to-noise ratio matters** — 500 focused tokens > 25K unfocused tokens
3. **Cost scales linearly** — You're paying for every irrelevant token
4. **Distraction is real** — Irrelevant context creates false connections
5. **Start minimal, add as needed** — Don't dump, curate
