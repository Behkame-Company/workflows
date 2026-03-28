# Constraint Specification

> Defining boundaries and guardrails that keep AI output safe and useful.

---

## Types of Constraints

### 1. Hard Constraints (Never Violate)
```
- Never expose API keys or secrets in code or output
- Never delete data without explicit confirmation
- Never modify database migration files after they've been applied
- Always validate user input before database operations
```

### 2. Soft Constraints (Override with Reason)
```
- Functions should be under 25 lines (except for state machines)
- Prefer composition over inheritance (unless modeling a clear is-a relationship)
- One file per component (unless tightly coupled helper components)
```

### 3. Format Constraints
```
- Respond with code blocks only, no explanatory prose
- Use TypeScript, not JavaScript
- Include type annotations on all function signatures
```

### 4. Scope Constraints
```
- Only modify files in src/auth/ for this task
- Do not change any test files
- Do not add new dependencies without asking
```

---

## How to Specify Constraints

### Positive Over Negative

```
❌ "Don't use var"
   "Don't use any"
   "Don't use console.log"
   "Don't use == "

✅ "Use const for immutable bindings, let for reassignable ones"
   "Use explicit TypeScript types for all declarations"
   "Use the Logger service for all debug output"
   "Use === for all equality comparisons"
```

**Why**: Positive constraints tell the model what to do. Negative constraints only eliminate one option among infinite.

### Prioritized Constraints

```
When constraints conflict, follow this priority:
1. Security (never compromise)
2. Correctness (must work correctly)
3. Readability (next developer must understand it)
4. Performance (optimize only when measured)
5. Brevity (shorter is better, all else equal)
```

### Constraints with Escape Hatches

```
Rule: Keep functions under 25 lines.
Exception: Complex switch statements and state machines may 
exceed this if extracting cases would reduce readability.
When exceeding: Add a comment explaining why the function is long.
```

---

## Constraint Categories for Codebases

| Category | Example Constraints |
|----------|-------------------|
| **Types** | No `any`, explicit return types, strict null checks |
| **Error Handling** | Use custom error classes, always catch async errors |
| **Security** | Validate all input, parameterize queries, no eval() |
| **Testing** | No test.skip, no shared state between tests |
| **Dependencies** | No new deps without approval, pin versions |
| **Architecture** | Repository pattern for DB, service layer for logic |
| **API** | RESTful naming, standard error format, pagination |
| **Git** | Conventional commits, squash before merge |

---

## Constraints in Different Tools

### copilot-instructions.md
```markdown
## Hard Rules
- Use Zod for all API input validation
- Use AppError class (not Error) for all error throws
- Export from index.ts in each module directory
```

### CLAUDE.md
```markdown
## Constraints
- Run `npm test` after every change
- Never force-push to main
- Commit after each logical unit of work
```

### Prompt Files
```yaml
---
description: "Generate API endpoint"
---
Create an endpoint following these constraints:
- Input validation with Zod
- Error handling with AppError
- Response format: { data: T } for success, { error: string, code: string } for failure
```

---

## Next Steps

- 🔗 [System Prompt Design](system-prompt-design.md) — Where constraints live
- 🔗 [Instruction Hierarchy](instruction-hierarchy.md) — How constraints are prioritized
