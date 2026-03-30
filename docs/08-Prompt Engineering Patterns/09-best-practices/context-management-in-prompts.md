# Context Management in Prompts

> Providing the right amount of context — not too little, not too much.

---

## The Goldilocks Problem

```
Too little context:     Model guesses, hallucinates, uses wrong patterns
Right context:          Model produces exactly what you need
Too much context:       Attention diluted, model misses key instructions, slower
```

---

## What to Include

### Always Include
| Context | Why |
|---------|-----|
| Error messages (exact) | Pinpoints the issue |
| Function signature | Defines the interface |
| Project conventions that apply | Prevents wrong patterns |
| Success criteria | Defines "done" |

### Include When Relevant
| Context | When |
|---------|------|
| Related types/interfaces | When generating or modifying typed code |
| Test expectations | When the code needs to pass specific tests |
| Database schema | When writing queries or migrations |
| API contract | When building endpoints or clients |

### Don't Include
| Context | Why Not |
|---------|---------|
| Entire files when only a function matters | Wastes attention budget |
| Unrelated project files | Noise drowns signal |
| Historical context from past sessions | Likely outdated |
| Your thought process | Focus on the task, not the journey |

---

## Context Targeting Strategies

### 1. Reference Instead of Paste
```
❌ [pasting 500 lines of code]
   "Fix the bug in this code"

✅ "Fix the null reference bug in getUserProfile() at line 42 
    of src/services/user.ts. The user.profile field can be null 
    for unverified users."
```

In Copilot CLI and Claude Code, the tool can read the file itself. Give it the location.

### 2. Provide Minimal Reproduction
```
Instead of the whole module, provide:
- The function that fails
- The input that triggers the failure
- The error message
- The expected behavior
```

### 3. Use Targeted File Reads
```
"Read lines 30-60 of src/auth/service.ts (the login function)
 and lines 1-20 of src/auth/types.ts (the interfaces it uses)"
```

---

## Context Budget by Task Type

| Task | Context Size | What to Include |
|------|:------------:|----------------|
| Simple bug fix | Small | Error, affected function, expected behavior |
| New function | Medium | Interface/types, project patterns, constraints |
| Refactoring | Medium | The code to refactor + the target pattern |
| Architecture review | Large (but curated) | Key files, not everything |
| Full feature | Split into subtasks | Different context per subtask |

---

## The Signal-to-Noise Rule

> Every token in context should increase the probability of a correct response. If it doesn't, it's noise.

Before including context, ask: **"Does the model need this to succeed?"**
