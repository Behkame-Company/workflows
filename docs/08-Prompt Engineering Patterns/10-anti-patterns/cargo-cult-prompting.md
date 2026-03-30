# Cargo-Cult Prompting

> Copying prompts without understanding why they work — and why they break.

---

## What Is Cargo-Cult Prompting?

Cargo-cult prompting is blindly copying prompt patterns from blog posts, tutorials, or other teams without understanding:
1. **Why** the pattern works
2. **When** it applies
3. **What** makes it fail

Named after cargo cults — imitating the form without understanding the function.

---

## Common Cargo-Cult Patterns

### 1. "Let's Think Step by Step" Everywhere
```
❌ "Rename the variable from `x` to `userName`. Let's think step by step."
```
Chain-of-thought helps for complex reasoning, not simple transformations. Adding it to simple tasks wastes tokens.

### 2. Aggressive Emphasis from 2023
```
❌ "CRITICAL: You MUST ALWAYS use grep before making any changes. 
    FAILURE TO DO THIS IS UNACCEPTABLE."
```
Modern models (Claude 4.6, GPT-4o) follow normal language well. CAPS and emphasis cause overtriggering.

### 3. Copying .cursorrules Between Projects
```
❌ Copying a Next.js project's .cursorrules to a Python FastAPI project
```
Project instructions must match the actual project. Rules for React hooks don't help in a Python backend.

### 4. "You are an expert" Without Specifics
```
❌ "You are a world-class expert software engineer with 30 years of experience 
    in all programming languages and frameworks."
```
Hyperbole adds no useful information. Specific expertise (domain, priorities) does.

### 5. Magic Phrases
```
❌ "Take a deep breath and focus"
❌ "I'll tip you $200 for a good answer"
❌ "This is very important for my career"
```
These were anecdotal tricks that showed marginal effects in specific benchmarks. They don't reliably improve code generation.

---

## Why Cargo-Cult Prompts Fail

| Issue | Why |
|-------|-----|
| Context mismatch | Pattern designed for different model/tool/version |
| Outdated | Models improve; techniques that helped GPT-3 may hinder GPT-4 |
| Overfitting to examples | Pattern only works for the specific demo case |
| Missing the core principle | Copying the format but not the insight |

---

## How to Avoid It

### 1. Understand Before Copying
Ask: **Why does this prompt work?** If you can't explain the mechanism, don't copy it.

### 2. Test in Your Context
Run the prompt with **your** code, **your** model, **your** use case. Blog post results don't transfer automatically.

### 3. Adapt, Don't Copy
Take the **principle** and apply it to your context:

```
Blog post: "You are a senior Python developer who..."
Your version: "You are a senior TypeScript developer who prioritizes 
type safety and works in our Next.js monorepo."
```

### 4. Track What Works
Keep a record of which prompts perform well in your specific environment. Your best practices may differ from the internet's.

### 5. Update Regularly
Prompt patterns that worked 6 months ago may be obsolete. Model updates change optimal prompting strategies.

---

## The Test

Before using a prompt pattern from the internet:
1. ✅ Can you explain **why** it works?
2. ✅ Have you tested it with **your** model and code?
3. ✅ Is it still relevant for the **current** model version?
4. ✅ Have you adapted it to **your** project's context?

If any answer is "no," you're cargo-culting.
