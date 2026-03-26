# Context Budget Management

> Understanding token limits and optimizing what Copilot sees.

---

## The Context Window Problem

Every AI model has a **context window** — a maximum amount of text it can process at once. Your instructions compete with:

- The system prompt (Copilot's built-in behavior)
- Open file contents
- Conversation history
- Code being analyzed
- Tool output and results
- File references in prompts

**Your instructions are a small fraction of the total context.** Making them concise means more room for actual code.

---

## Token Budget Guidelines

### Practical Limits

| Copilot Feature | Effective Instruction Limit |
|-----------------|----------------------------|
| Copilot Chat | ~1,000-2,000 words of instructions |
| Coding Agent | ~2,000-3,000 words (larger context) |
| Code Review | ~4,000 characters (~700 words) |
| Inline Completions | Instructions have minimal impact |

### Rule of Thumb

```
copilot-instructions.md:  500-1,000 words max
Each .instructions.md:    200-500 words max
Each .prompt.md:          300-800 words max
Total loaded at once:     1,500-2,500 words ideal
```

---

## What Gets Loaded When

### Scenario: Working on a React component

```
Loaded:
+ Organization instructions:           ~100 words
+ copilot-instructions.md:             ~500 words
+ frontend.instructions.md (applyTo):  ~300 words
+ Open file contents:                  ~2,000 words
+ Chat history:                        ~1,000 words
= Total instruction load:              ~900 words ← Good

Remaining for code context:            ~7,000+ words
```

### Scenario: Working in agent mode on a complex task

```
Loaded:
+ Organization instructions:           ~100 words
+ copilot-instructions.md:             ~800 words
+ frontend.instructions.md:            ~400 words
+ testing.instructions.md:             ~400 words
+ Task description:                    ~500 words
+ Code from multiple files:            ~10,000 words
= Total instruction load:              ~1,700 words ← OK but heavy

Remaining for reasoning:               varies by model
```

---

## Optimization Strategies

### Strategy 1: Prioritize by Impact

Not all instructions are equally valuable. Rank them:

| Priority | Type | Impact |
|----------|------|--------|
| **P0** | Build/test/lint commands | Critical — without these, Coding Agent can't verify work |
| **P0** | Stack declaration | Critical — prevents wrong framework suggestions |
| **P1** | Hard boundaries ("never do X") | High — prevents common mistakes |
| **P1** | Core conventions (3-5 rules) | High — shapes code quality |
| **P2** | Structure overview | Medium — helps navigation |
| **P2** | Additional conventions | Medium — refinement |
| **P3** | Style preferences | Low — nice to have |
| **P3** | Explanatory context | Low — wastes tokens |

**When you're over budget**: Cut from the bottom up.

### Strategy 2: Use Path-Specific Files to Reduce Load

Instead of one 1,500-word copilot-instructions.md, split:

```
Before (always loaded):
  copilot-instructions.md: 1,500 words

After (contextual loading):
  copilot-instructions.md:     500 words (always loaded)
  frontend.instructions.md:    400 words (only for frontend files)
  backend.instructions.md:     400 words (only for backend files)
  database.instructions.md:    300 words (only for DB files)
```

Now Copilot only loads ~900 words at a time instead of 1,500.

### Strategy 3: Remove Low-Value Content

Delete instructions that:
- State the obvious ("Write syntactically correct code")
- Copilot already does by default ("Use proper indentation")
- Are too vague to follow ("Write clean code")
- Duplicate what's in the linter config ("Use 2-space indentation" — ESLint handles this)

### Strategy 4: Compress Wordy Instructions

```markdown
# Before: 45 words
When you are writing TypeScript code, please make sure to always enable
strict mode in the TypeScript configuration. This means setting "strict":
true in the tsconfig.json file. Additionally, you should avoid using the
any type wherever possible.

# After: 8 words
Use TypeScript strict mode; never use `any`.
```

Same information, 82% fewer tokens.

### Strategy 5: Move Examples to Prompt Files

Instructions should state **rules**. If you need elaborate examples, put them in a prompt file that's loaded on-demand:

```markdown
# In copilot-instructions.md (always loaded)
## Error Handling
API routes return `{ success: false, error: string }` on failure.

# In .github/prompts/error-handling-guide.prompt.md (on-demand)
# Error Handling Guide
[Detailed examples, patterns, edge cases — loaded only when needed]
```

---

## Token-per-Instruction Analysis

| Instruction | Tokens | Value | Keep? |
|-------------|--------|-------|-------|
| `npm run test` — Run all tests | ~10 | Critical | ✅ |
| Use TypeScript strict mode | ~6 | High | ✅ |
| Never commit secrets | ~4 | High | ✅ |
| Use 2-space indentation | ~5 | Low (linter handles) | ❌ |
| Write clean, maintainable code | ~6 | Zero (too vague) | ❌ |
| Always remember to handle errors properly in all cases | ~12 | Low (too vague) | ❌ → rewrite |
| Catch errors in API routes, return `{ error, code }` | ~12 | High | ✅ |

---

## Measuring Your Budget

### Quick Estimate

```
1 word ≈ 1.3 tokens
100 words ≈ 130 tokens
500 words ≈ 650 tokens
1,000 words ≈ 1,300 tokens
```

### Practical Test

1. Count words in your instruction files
2. Add up all files that load simultaneously
3. Compare against guidelines:
   - Under 1,000 words total: ✅ Efficient
   - 1,000-2,000 words: ⚠️ Acceptable
   - Over 2,000 words: ⛔ Trim or restructure

---

## Budget Optimization Checklist

- [ ] copilot-instructions.md is under 1,000 words
- [ ] Each path-specific file is under 500 words
- [ ] No duplicate rules across files
- [ ] Every rule scores 3+ on the effectiveness rubric
- [ ] No hedging language ("try to", "when possible")
- [ ] No obvious/default behaviors stated
- [ ] Complex examples moved to prompt files
- [ ] Total simultaneous load under 2,000 words

---

*Next: [Maintaining Instructions](maintaining-instructions.md) →*
