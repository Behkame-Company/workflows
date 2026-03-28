# Clarity & Specificity

> The most impactful prompt engineering principle: be precise about what you want.

---

## The Clarity Spectrum

```
Vague                                              Precise
─────────────────────────────────────────────────────────►
"Fix this"    "Fix the bug"   "Fix the null    "Fix the null
                               reference on     reference on
                               line 42"         line 42 by adding
                                                optional chaining
                                                to user.profile"
```

**Every step toward precision dramatically improves output quality.**

---

## Five Dimensions of Clarity

### 1. WHAT — The Action
```
❌ "Work on the auth module"
✅ "Add refresh token rotation to the auth module"
```

### 2. WHERE — The Scope
```
❌ "Update the tests"
✅ "Add tests for the createUser function in src/services/__tests__/user.test.ts"
```

### 3. HOW — The Approach
```
❌ "Make it faster"
✅ "Add a Redis cache for the getUserById query with 60-second TTL"
```

### 4. WHY — The Motivation
```
❌ "Add error handling"
✅ "Add error handling because the function currently fails silently 
    when the database is unreachable, causing downstream 500 errors"
```

### 5. WHEN DONE — Success Criteria
```
❌ "Implement pagination"
✅ "Implement offset-based pagination. Done when:
    - GET /users?page=2&limit=10 returns correct page
    - Response includes { data: User[], total: number, page: number }
    - All existing tests still pass"
```

---

## Specificity Patterns

### Reference Existing Code
```
"Follow the pattern in src/api/users.controller.ts for error handling"
```
More specific than describing the pattern. The model reads and mirrors it.

### Use Exact Values
```
❌ "Use a reasonable timeout"
✅ "Use a 30-second timeout"
```

### Name the Output Format
```
❌ "Give me the results"
✅ "Return a JSON array of objects with { file: string, line: number, issue: string }"
```

### Quantify When Possible
```
❌ "Keep functions short"
✅ "Keep functions under 25 lines"

❌ "Limit the response"
✅ "Respond in 3-5 bullet points"
```

---

## The Precision Checklist

Before sending a prompt, verify:

- [ ] **Action** is a specific verb (generate, fix, refactor, add, remove)
- [ ] **Target** is named (function name, file path, component)
- [ ] **Constraints** are explicit (what to keep, what to change, what to avoid)
- [ ] **Format** is defined (if the output structure matters)
- [ ] **Ambiguity** is resolved (no "probably", "maybe", "something like")

---

## When Vague Is OK

Not every prompt needs maximum precision:
- **Brainstorming**: "What approaches could we take for caching?"
- **Exploration**: "Explain how this module works"
- **Quick answers**: "What does this regex do?"

Use precision for tasks with specific expected outputs.

---

## Next Steps

- 🔗 [Context Management in Prompts](context-management-in-prompts.md) — Right amount of context
- 🔗 [Instruction Following](../02-core-techniques/instruction-following.md) — Clear instruction patterns
