# Prompt Engineering Techniques for Copilot

> Advanced prompting strategies for better AI code output.

---

## The 4S Principle

Microsoft's recommended framework for Copilot prompts:

### 1. Single — One Task at a Time

```markdown
# ❌ Multi-task prompt
Create a user registration form with validation, hook it up to the API,
add error handling, write tests, and update the documentation.

# ✅ Single task
Create a user registration form component with email and password fields.
Use Zod for client-side validation.
```

Break complex requests into sequential steps.

### 2. Specific — Name Technologies and Patterns

```markdown
# ❌ Generic
Create a form with validation

# ✅ Specific
Create a React form component using react-hook-form with Zod resolver.
Fields: email (string, valid email format), password (string, min 8 chars).
On submit, call `registerUser()` from `src/lib/api/auth.ts`.
```

### 3. Short — Concise Instructions

```markdown
# ❌ Verbose (72 words)
I would like you to please create a utility function that takes in a date
string in various formats and converts it to our standard ISO 8601 format.
The function should handle edge cases like null, undefined, and invalid
date strings gracefully, and it should return null for invalid inputs
rather than throwing an error.

# ✅ Concise (24 words)
Create `parseDate(input: string | null): string | null` in `src/lib/date.ts`.
Convert to ISO 8601. Return null for invalid inputs.
```

### 4. Surround — Provide Context Files

Open relevant files in your editor before prompting. Copilot uses open files as context:

- Open the file where code will be added
- Open related files with similar patterns
- Open type definitions that the new code should use
- Open test files to show expected test patterns

---

## Chain-of-Thought Prompting

For complex tasks, break the prompt into thinking steps:

```markdown
# Step-by-step approach
1. First, read `src/lib/auth/session.ts` to understand the current session handling
2. Then, identify where the session refresh logic should be added
3. Implement the refresh: check expiry 5 min before token expiration
4. Add error handling for failed refresh attempts
5. Write a test that verifies refresh happens before expiry
```

### Why This Works

Chain-of-thought forces the model to:
- Consider existing code before writing new code
- Think about the approach before jumping to implementation
- Handle edge cases that emerge from the analysis

---

## Contrastive Prompting

Show both what you want AND what you don't want:

```markdown
Create an error handling utility.

✅ DO:
- Return Result type: `{ success: true, data: T } | { success: false, error: string }`
- Log errors using the project logger
- Include original error in logs but not in the response

❌ DON'T:
- Don't throw exceptions from API handlers
- Don't use `console.log`
- Don't expose stack traces in error responses
```

### Why This Works

Contrastive examples narrow the solution space. The model can eliminate wrong approaches before generating code.

---

## Few-Shot Prompting

Provide 1-2 examples of the desired output:

```markdown
Create a new API endpoint for listing products, following the same pattern as the users endpoint.

Example (existing pattern):
```typescript
// src/app/api/users/route.ts
export async function GET(req: NextRequest) {
  const session = await getSession();
  if (!session) return unauthorized();
  
  const users = await prisma.user.findMany({ take: 20 });
  return NextResponse.json({ data: users });
}
```

Now create the same pattern for products at `src/app/api/products/route.ts`.
```

### Why This Works

Examples are the most powerful form of instruction. One good example communicates more than a paragraph of rules.

---

## Iterative Refinement

Start broad, then refine:

```markdown
# Round 1 (initial)
Create a product search function.

# Round 2 (refine after seeing output)
Good start. Now add:
- Full-text search using Prisma's `contains` filter
- Pagination with cursor-based approach
- Price range filtering

# Round 3 (polish)
Almost there. Fix:
- Add input validation with Zod
- Handle the case where no results are found (return empty array, not error)
- Add JSDoc documentation
```

### Why This Works

Iterative refinement is more effective than trying to specify everything upfront. Each round builds on concrete output.

---

## Context Anchoring

Reference specific files and patterns to anchor Copilot's understanding:

```markdown
# Without anchoring
Create a service for managing products.

# With anchoring
Create `src/services/product.service.ts` following the same pattern as
`src/services/user.service.ts`:
- Export a class with static methods
- Each method uses Prisma for database access
- Input validated with Zod schemas from `src/validators/`
- Errors use the AppError class from `src/lib/errors.ts`
```

---

## Negative Space Prompting

Define what to exclude to narrow the output:

```markdown
Create a React form component.

Constraints:
- No form libraries (just controlled components with useState)
- No CSS-in-JS (use Tailwind classes)
- No client-side routing (it's a modal form)
- No external validation (use simple if/else checks)
```

### Why This Works

By closing off options, you force Copilot toward the specific implementation you want.

---

## Role Prompting

Frame the task from a specific perspective:

```markdown
You are a senior security engineer reviewing this authentication module.
Identify any vulnerabilities and suggest specific fixes.
Focus on: token handling, password storage, session management, rate limiting.
```

```markdown
You are a performance engineer optimizing this React component.
Current issue: the component re-renders 47 times on page load.
Identify unnecessary renders and suggest memoization strategies.
```

---

## Prompt Anti-Patterns

| Anti-Pattern | Example | Why It Fails |
|-------------|---------|--------------|
| Kitchen sink | "Build everything at once" | Too much for one prompt; results are shallow |
| Too abstract | "Make it better" | No actionable direction |
| Copy-paste specs | Pasting a 2-page requirements doc | Exceeds effective context; middle content lost |
| Redundant context | Repeating what's in instruction files | Wastes tokens on duplicate info |
| No success criteria | "Create a function" (no expected behavior) | Copilot guesses what success looks like |

---

## Combining Prompting with Instructions

### Instructions = Rules (Passive, Always-On)

```markdown
# copilot-instructions.md
Use TypeScript strict mode.
Validate input with Zod.
```

### Prompts = Tasks (Active, On-Demand)

```markdown
# User prompt
Create a product search endpoint that accepts name, category, and price range.
Return paginated results with cursor-based navigation.
```

### Combined Effect

Instructions set the **guardrails** (TypeScript, Zod). The prompt sets the **direction** (product search, pagination). Together, they produce focused, compliant code.

---

*Next: [Monorepo Strategies](monorepo-strategies.md) →*
