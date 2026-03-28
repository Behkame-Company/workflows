# Refactoring Prompts

> Prompts for improving existing code while preserving behavior.

---

## The Refactoring Prompt Formula

```
[What to change] + [Why] + [What to preserve] + [Pattern to follow]
```

The key difference from code generation: you must specify **what NOT to change**.

---

## Common Refactoring Patterns

### Extract Function
```
Extract the validation logic from lines 15-30 of handleSubmit() 
into a separate function called validateFormData().

Requirements:
- The extracted function should be pure (no side effects)
- Keep the same validation rules
- Return { valid: boolean; errors: string[] }
- handleSubmit() should call the new function
- Do NOT change any other logic in handleSubmit()
```

### Convert Callbacks to Async/Await
```
Refactor this function from callback-style to async/await.

Preserve:
- All error handling behavior
- Return types
- Function signature (except removing callback parameter)

Replace:
- callback(null, result) → return result
- callback(error) → throw error
- nested callbacks → sequential awaits
```

### Simplify Conditionals
```
Simplify the conditional logic in processOrder().
Currently it has 5 levels of nested if/else.

Strategy:
- Use early returns for guard clauses
- Extract complex conditions into named booleans
- Maximum 2 levels of nesting

Do NOT change:
- The business logic outcomes
- The error messages
- The logging
```

---

## Scope Control

The most important aspect of refactoring prompts is **limiting scope**:

```
✅ "Refactor ONLY the database query function to use Prisma"
✅ "Change ONLY the error handling pattern, keep all business logic"
✅ "Rename userId to userIdentifier in THIS FILE ONLY"

❌ "Refactor this module" (what aspect? everything?)
❌ "Clean up this code" (clean up how?)
❌ "Make this better" (better in what dimension?)
```

---

## Safe Refactoring Prompt Template

```markdown
Refactor [target function/file] to [specific change].

## What to change
- [Specific transformation]
- [Specific transformation]

## What to preserve
- All existing behavior and return values
- All error handling
- All existing tests must still pass
- Public API / function signatures

## Pattern to follow
See [reference file] for the target pattern.

## Verification
After refactoring, show me:
1. What changed (diff summary)
2. Any behavioral differences (should be none)
3. Any new dependencies added
```

---

## Type of Refactoring → Prompt Strategy

| Refactoring | Key Instruction |
|-------------|----------------|
| Rename | "Rename X to Y in all usages within this file" |
| Extract | "Extract [lines/logic] into [named function]" |
| Inline | "Inline this wrapper function, removing the indirection" |
| Move | "Move this function to [target file], update all imports" |
| Simplify | "Reduce nesting to max 2 levels using early returns" |
| Modernize | "Convert to [modern pattern] while keeping same behavior" |

---

## Next Steps

- 🔗 [Debugging Prompts](debugging-prompts.md) — Finding and fixing bugs
- 🔗 [Code Generation Prompts](code-generation-prompts.md) — Creating new code
