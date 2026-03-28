# Mental Models for LLMs

> How large language models actually process your prompts — understanding the machine helps you write better prompts.

---

## How LLMs "Think"

LLMs don't understand meaning. They predict the most likely next token based on patterns in their training data. Understanding this changes how you write prompts.

```
Your prompt → Tokenization → Attention across all tokens → Next token prediction → Response
```

---

## Key Mental Models

### 1. The Brilliant New Employee

The most practical model: think of the AI as a highly skilled developer who just joined your team. They:
- Have read millions of codebases but know nothing about yours
- Follow instructions literally (not "as intended")
- Need explicit context about your conventions
- Excel when given clear requirements and examples

**Implication**: If a new hire would be confused by your prompt, so will the model.

### 2. The Autocomplete Engine

At its core, the model predicts "what text would most likely come next." This means:
- Prompts that look like patterns from training data get pattern-matching responses
- Code that follows common conventions gets better completions
- Unusual naming or structure may confuse the model

**Implication**: Frame prompts in familiar patterns. "Review this code for bugs" works better than "Find problems with this code thing."

### 3. The Attention Budget

Models have limited attention. The attention mechanism weighs each token against every other token (n² complexity). As context grows:
- Tokens compete for "attention weight"
- Critical instructions can get drowned by large code pastes
- Position matters: start and end get more attention than the middle

**Implication**: Put the most important information first and last. Keep context lean.

---

## Tokenization Effects

### What You Type vs What the Model Sees

```
Human sees:    "getUserById"
Model sees:    ["get", "User", "By", "Id"]  (4 tokens)

Human sees:    "function"
Model sees:    ["function"]  (1 token)

Human sees:    "    "  (4 spaces)
Model sees:    ["    "]  (1 token in most tokenizers)
```

### Why This Matters

- **Naming conventions affect token count**: `getUserById` (4 tokens) vs `get_user_by_id` (7 tokens)
- **Comments are tokens**: Every `// TODO` uses budget
- **Whitespace matters**: Indentation consumes tokens
- **Language differences**: Python is token-efficient; Java/HTML is token-expensive

---

## Position Effects

### The U-Shaped Recall Curve

```
Recall Accuracy
     ┌─────────────────────────────┐
100% │ ■                         ■ │
 80% │  ■                       ■  │
 60% │   ■                     ■   │
 40% │    ■■                 ■■    │
 20% │      ■■■■■■■■■■■■■■■       │
  0% │                             │
     └─────────────────────────────┘
      Start    Middle       End
           Position in Context
```

**The model recalls information best from the beginning and end of the context.**

**Implication**:
- System prompt (beginning): Core rules and identity
- Recent conversation (end): Current task and instructions
- Middle: Supporting context (may be partially forgotten)

---

## Temperature and Randomness

| Temperature | Behavior | Use For |
|-------------|----------|---------|
| 0.0 | Deterministic, same output each time | Code generation, structured output |
| 0.3-0.5 | Slightly varied, mostly consistent | General development tasks |
| 0.7-1.0 | Creative, varied responses | Brainstorming, documentation |
| >1.0 | Highly random, may be incoherent | Rarely useful for coding |

**Implication**: For coding tasks, use low temperature (0.0-0.3). For creative writing, use higher.

---

## What Models Are Good and Bad At

### Good At
- Pattern matching and completion
- Following explicit, structured instructions
- Translating between formats (JSON ↔ TypeScript ↔ SQL)
- Applying known patterns to new contexts
- Parallel reasoning about multiple concerns

### Bad At
- Counting things accurately (characters, lines, tokens)
- Complex arithmetic without step-by-step reasoning
- Maintaining state across very long conversations
- Knowing what they don't know (hallucination risk)
- Following ambiguous or contradictory instructions

**Implication**: Play to strengths. Use step-by-step for math. Use examples for format. Don't trust counts without verification.

---

## Next Steps

- 🔗 [Prompt vs Context Engineering](prompt-vs-context-engineering.md) — The bigger picture
- 🔗 [Chain-of-Thought](../02-core-techniques/chain-of-thought.md) — Working around reasoning limits
