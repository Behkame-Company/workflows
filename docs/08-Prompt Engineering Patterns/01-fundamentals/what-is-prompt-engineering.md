# What Is Prompt Engineering?

> The discipline of crafting inputs that reliably produce useful outputs from AI models.

---

## Definition

**Prompt engineering** is the practice of designing, testing, and refining the text inputs (prompts) given to large language models to achieve specific, reliable outcomes.

It is **not** just writing good questions. It's a systematic discipline that includes:
- Understanding how models process text
- Structuring inputs for maximum signal
- Testing and iterating on prompt effectiveness
- Matching techniques to task complexity

---

## Evolution

| Era | Approach | Focus |
|-----|----------|-------|
| **2020-2022** | Prompt crafting | Finding magic words that work |
| **2022-2023** | Prompt engineering | Systematic techniques (CoT, few-shot) |
| **2024-2025** | Context engineering | Curating all tokens holistically |
| **2025+** | Agentic prompting | Teaching AI how to use tools and reason |

> **Key shift**: We moved from "what words make the AI do what I want" to "what information does the AI need to succeed at this task."

---

## Why It Matters for Developers

### Without prompt engineering:
```
Prompt: "Fix this code"
Result: Model guesses what's wrong, changes random things, may break more
```

### With prompt engineering:
```
Prompt: "This TypeScript function throws 'Cannot read property 
of undefined' when `user.profile` is null. Fix the null check 
following our project pattern of early returns with explicit 
error messages. Keep the existing return type."

Result: Precise fix matching your codebase conventions
```

The difference: **specificity**, **context**, and **constraints**.

---

## Core Principles

### 1. Clarity Over Cleverness
Write prompts that a new team member could follow. If a human would be confused, the model will be too.

### 2. Show, Don't Just Tell
Examples are worth more than descriptions. One good example beats a paragraph of instructions.

### 3. Constrain the Output Space
Tell the model what format you want, what to include, what to exclude. Open-ended prompts get open-ended (and often wrong) results.

### 4. Match Technique to Complexity
- Simple task → Zero-shot (direct instruction)
- Moderate task → Few-shot (with examples)
- Complex reasoning → Chain-of-thought
- Multi-step → Prompt chaining or agentic patterns

### 5. Test and Iterate
Prompts are code. They should be tested, versioned, and refined based on empirical results.

---

## The Prompt Engineering Mindset

Think of yourself as a **technical writer** for an extremely fast, extremely literal reader who:
- Has read the entire internet but has no memory of your project
- Follows instructions exactly as written (not as intended)
- Gets confused by ambiguity but excels with clear structure
- Learns patterns instantly from examples
