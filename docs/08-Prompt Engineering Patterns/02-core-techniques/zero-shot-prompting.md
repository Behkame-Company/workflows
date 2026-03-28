# Zero-Shot Prompting

> Direct instructions without examples — the simplest prompting technique.

---

## What Is Zero-Shot?

Zero-shot prompting means giving the model an instruction without any examples. You rely entirely on the model's training to understand what you want.

```
Prompt: "Classify this text as positive, negative, or neutral: 
'The new API is much faster but the documentation is lacking'"

Response: "Mixed/Neutral — positive about performance, negative about documentation"
```

No examples were provided. The model inferred the task from the instruction alone.

---

## When Zero-Shot Works Well

| Task Type | Example | Why It Works |
|-----------|---------|-------------|
| Classification | "Is this a bug report or feature request?" | Well-known task pattern |
| Simple generation | "Write a JSDoc comment for this function" | Common format |
| Translation | "Convert this SQL query to Prisma" | Direct transformation |
| Summarization | "Summarize this PR in 3 bullet points" | Standard task |
| Formatting | "Convert this object to a TypeScript interface" | Pattern-matching |

---

## When Zero-Shot Fails

- **Complex reasoning**: "Calculate the time complexity of this algorithm" → Use Chain-of-Thought
- **Unusual formats**: "Output as a Mermaid diagram with custom styling" → Use examples (few-shot)
- **Domain-specific conventions**: "Follow our API response format" → Needs examples or context
- **Multi-step tasks**: "Refactor this module, update tests, and write migration" → Use decomposition

---

## Writing Effective Zero-Shot Prompts

### ❌ Weak
```
Fix this code
```

### ✅ Strong
```
Fix the null reference error in this TypeScript function. 
Use optional chaining (?.) and nullish coalescing (??) operators. 
Keep the existing return type and function signature.
```

### The Formula
```
[Action verb] + [Specific task] + [Constraints] + [Format]
```

Examples:
- "**Review** this function **for SQL injection vulnerabilities**. **Only flag real risks, not theoretical ones.** **List each as: line, risk, fix.**"
- "**Convert** this JavaScript function **to TypeScript** with **strict types, no `any`**. **Return only the code.**"

---

## Zero-Shot Enhancement Techniques

### 1. Task Decomposition in Instructions
```
Analyze this React component:
1. First, identify all props and their types
2. Then, check for missing error boundaries  
3. Finally, suggest performance optimizations
```

### 2. Output Anchoring
```
Respond with EXACTLY this format:
Severity: [low/medium/high/critical]
Issue: [one sentence]
Fix: [code snippet]
```

### 3. Negative Constraints
```
Review this code for bugs.
- Do NOT comment on code style or formatting
- Do NOT suggest architectural changes
- ONLY flag actual runtime errors or logic bugs
```

### 4. Adding "Let's think step by step"
```
Analyze the time complexity of this sorting function.
Let's think step by step.
```

This simple addition (zero-shot Chain-of-Thought) can dramatically improve reasoning quality.

---

## Zero-Shot vs Few-Shot Decision Tree

```
Is the task well-known? (classification, summarization, translation)
├── Yes → Try zero-shot first
│   ├── Output correct? → Done ✓
│   └── Output wrong? → Add examples (few-shot)
└── No → Use few-shot from the start
```

---

## Next Steps

- 🔗 [Few-Shot Prompting](few-shot-prompting.md) — When zero-shot isn't enough
- 🔗 [Chain-of-Thought](chain-of-thought.md) — For complex reasoning tasks
