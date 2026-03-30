# Tokens and Tokenization

> Understanding how text becomes tokens — the fundamental unit of context window measurement.

---

## What Are Tokens?

Tokens are the atomic units that language models process. They're not characters, not words — they're sub-word pieces that the model's tokenizer creates.

**Rough rules of thumb:**
- 1 token ≈ 4 characters in English
- 1 token ≈ 0.75 words
- 100 tokens ≈ 75 words
- 1,000 tokens ≈ 750 words ≈ ~1.5 pages of text

---

## How Tokenization Works

```
Input:  "function calculateTotal(items) {"
Tokens: ["function", " calculate", "Total", "(", "items", ")", " {"]
Count:  7 tokens

Input:  "const API_KEY = process.env.API_KEY;"
Tokens: ["const", " API", "_KEY", " =", " process", ".env", ".API", "_KEY", ";"]
Count:  9 tokens
```

### What's Expensive (More Tokens)
- Long variable names
- Verbose comments and documentation
- Repeated boilerplate code
- Full file contents when only a snippet is needed
- Large JSON/XML tool outputs
- Base64-encoded content (images, files)

### What's Cheap (Fewer Tokens)
- Concise, well-named code
- Summary instead of full content
- Structured data (tables) instead of prose
- References (file paths) instead of full files

---

## Token Counts for Common Development Artifacts

| Artifact | Approximate Tokens |
|----------|-------------------|
| Simple function (10 lines) | 50-100 |
| Full source file (200 lines) | 800-1,500 |
| README.md (typical) | 500-2,000 |
| package.json | 200-500 |
| Full API response (JSON) | 500-5,000 |
| Error stack trace | 200-500 |
| Git diff (medium PR) | 1,000-5,000 |
| Entire small project (50 files) | 50,000-100,000 |

---

## Tokenization Across Languages

Different programming languages tokenize differently:

| Language | Tokens per Line (avg) | Notes |
|----------|----------------------|-------|
| Python | 8-12 | Concise syntax, fewer tokens |
| JavaScript | 10-15 | More punctuation and braces |
| Java | 12-18 | Verbose type declarations |
| HTML | 15-25 | Very tag-heavy |
| JSON | 10-20 | Keys repeat, lots of punctuation |
| YAML | 6-10 | Most token-efficient config format |

---

## Practical Implications

### For Context Window Management
- **Count tokens, not lines** — 100 lines of Java ≠ 100 lines of Python
- **Token counting APIs exist** — Use them before sending large inputs
- **Images are expensive** — A single image can cost 1,000-6,000 tokens
- **Tool results dominate** — A file read can inject 1,000+ tokens instantly

### For Cost Management
- Input tokens are billed per request (cheaper than output)
- Output tokens (responses) are more expensive
- Thinking tokens (Claude) count as output but aren't carried forward
- Cached tokens are discounted

---

## Key Takeaways

1. **1 token ≈ 4 characters** — Use this for quick estimates
2. **Code is token-dense** — A 200-line file is ~1,000 tokens
3. **Tool outputs accumulate fast** — File reads, search results add up
4. **Concise > verbose** — Shorter code means more room in context
5. **Count before you send** — Use token counting APIs for precision
