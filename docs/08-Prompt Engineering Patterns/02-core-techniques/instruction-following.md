# Instruction Following

> Writing clear, unambiguous instructions that models execute reliably.

---

## The Golden Rule

> Show your prompt to a colleague with minimal context on the task. If they'd be confused, the AI will be too. — Anthropic

---

## Principles of Clear Instructions

### 1. Start with a Verb
```
❌ "The function needs error handling"
✅ "Add try-catch error handling to this function"
```

### 2. Be Specific About Scope
```
❌ "Fix the bugs"
✅ "Fix the null reference error on line 42 where user.profile is accessed without a null check"
```

### 3. Use Sequential Steps
```
Refactor this authentication module:
1. Extract the token validation into a separate function
2. Replace the callback pattern with async/await
3. Add input validation using Zod schemas
4. Update the error responses to use our standard format
```

### 4. Tell What TO Do, Not What NOT To Do
```
❌ "Don't use any types"
✅ "Use explicit TypeScript types for all parameters and return values"

❌ "Don't write long functions"
✅ "Keep each function under 25 lines. Extract helper functions as needed."
```

### 5. Provide Motivation
```
❌ "Always use early returns"
✅ "Use early returns for validation checks — this reduces nesting 
    and makes the happy path easier to read"
```

Claude and other models generalize better when they understand **why**.

---

## Instruction Formats

### Numbered Steps (for ordered tasks)
```
1. Read the existing test file
2. Identify uncovered code paths
3. Add test cases for each uncovered path
4. Run tests to verify they pass
5. Report coverage before and after
```

### Bullet Points (for unordered rules)
```
Rules for this codebase:
- All API endpoints must validate input with Zod
- Errors must use the AppError class
- Database queries go through the repository layer
- No direct imports from node_modules in components
```

### Conditional Instructions
```
If the function is pure (no side effects):
  - Add a unit test with property-based testing
  
If the function has side effects (API calls, DB writes):
  - Add an integration test with mocked dependencies
  - Add error handling for the external dependency
```

---

## The "New Employee" Test

Before sending a prompt, ask:

| Question | If No... |
|----------|----------|
| Would a new hire know what "fix this" means? | Be more specific |
| Would they know which file to change? | Specify the file path |
| Would they know our conventions? | Include relevant rules |
| Would they know when they're done? | Define success criteria |
| Would they know what NOT to change? | Add scope limits |

---

## Instruction Precision Scale

```
Vague ──────────────────────────────────── Precise
"improve this"  "add types"  "add return type  "add return type
                              annotations"      'Promise<User[]>' 
                                                to fetchUsers()"
```

**Rule**: Be as precise as the task requires. Simple tasks need simple instructions. Complex tasks need precise specifications.

---

## Combining with Other Techniques

| Technique | Adds |
|-----------|------|
| + Role | "As a security expert, review..." |
| + Examples | "Following this pattern: ..." |
| + CoT | "Think through each step..." |
| + Constraints | "Without changing the public API..." |
| + Format | "Respond as a JSON object with..." |

---

## Next Steps

- 🔗 [Clarity & Specificity](../09-best-practices/clarity-and-specificity.md) — Deep dive on precision
- 🔗 [Constraint Specification](../06-system-prompts/constraint-specification.md) — Defining boundaries
