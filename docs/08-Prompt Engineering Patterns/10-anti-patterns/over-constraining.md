# Over-Constraining

> Too many rules that conflict, confuse, and reduce output quality.

---

## The Problem

More rules ≠ better output. After a threshold, additional constraints hurt because:
1. Rules contradict each other
2. Model attention splits across too many concerns
3. Rules in the middle of a long list get forgotten
4. Model becomes overly cautious, producing bland output

---

## Symptoms of Over-Constraining

| Symptom | Cause |
|---------|-------|
| Model produces very generic code | Afraid of violating a rule |
| Model asks clarifying questions instead of acting | Paralyzed by conflicting rules |
| Output ignores some rules while following others | Can't satisfy all simultaneously |
| Responses are extremely verbose | Trying to address every constraint |
| Model refuses to complete the task | Perceives conflicts as unsolvable |

---

## Examples

### ❌ Over-Constrained
```
Write a function that:
- Is pure (no side effects)
- Logs every step for debugging
- Is under 10 lines
- Has comprehensive error handling
- Is highly performant
- Has full JSDoc documentation
- Follows functional programming patterns
- Uses early returns
- Has no more than 2 parameters
- Returns a discriminated union
- Is compatible with our testing framework
- Supports both sync and async usage
- ...
```

"Pure but logs every step" is contradictory. "Under 10 lines" conflicts with "comprehensive error handling" and "full JSDoc."

### ✅ Right-Sized
```
Write a pure function that validates an email address.

Key requirements:
- Returns { valid: boolean; error?: string }
- No side effects (pure)
- Handle: empty input, invalid format, too long (>254 chars)

Follow the pattern in src/validators/phone.ts.
```

---

## The Rule of 10-15

For system prompts and instruction files:

```
5 rules:   Model follows all of them consistently
10 rules:  Model follows most of them
15 rules:  Model follows the ones at the start and end
30 rules:  Model picks and chooses randomly
50+ rules: Model follows a handful and ignores the rest
```

**Keep persistent instructions to 10-15 critical rules.**

---

## How to Reduce Constraints

### 1. Prioritize Ruthlessly
From your 30 rules, which 10 would you keep if forced to choose? Keep those.

### 2. Merge Related Rules
```
❌ Separate rules:
- Use const for immutable bindings
- Use let only when reassignment is needed  
- Never use var
- Prefer destructuring
- Use template literals instead of concatenation

✅ Merged:
- Modern JavaScript: const by default, let when needed, 
  destructuring, template literals
```

### 3. Reference Instead of Specify
```
❌ "Follow these 20 code style rules..."
✅ "Follow the code style in .eslintrc and our existing code patterns"
```

Let your linter enforce style. Use the prompt for things linters can't check.

### 4. Use Path-Specific Instructions
Instead of one massive instruction file, use scoped files:
```
.github/copilot-instructions.md        → 10 global rules
src/api/.instructions.md                → 5 API-specific rules
src/components/.instructions.md         → 5 component-specific rules
```
