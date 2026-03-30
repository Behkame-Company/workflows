# System Prompt Optimization

> Writing system prompts that maximize instruction quality while minimizing token cost.

---

## The Right Altitude

Anthropic describes the "right altitude" as the Goldilocks zone between two failure modes:

```
TOO LOW (over-specified, brittle):
  "When the user asks to create a component, first check if
   the file exists using fs.existsSync, then use fs.writeFileSync
   to create it, then format with prettier --write, then..."

TOO HIGH (under-specified, vague):
  "Help the user with their code."

JUST RIGHT (clear heuristics):
  "Create React components following the project's established
   patterns. Use TypeScript interfaces for props. Include unit
   tests for new components. Follow the naming convention in
   src/components/."
```

---

## System Prompt Structure

Organize prompts into clear sections with XML tags or Markdown headers:

```markdown
## Role
You are a senior full-stack developer working on a React/Node.js project.

## Project Context
- Framework: Next.js 14 (App Router)
- Language: TypeScript (strict mode)
- Styling: Tailwind CSS
- Database: PostgreSQL with Prisma ORM
- Testing: Vitest + Playwright

## Code Standards
- All functions must have TypeScript return types
- Use Zod for runtime validation on API boundaries
- Error handling: use Result pattern, no bare try/catch
- Maximum function length: 40 lines

## What NOT to Do
- Do not use `any` type
- Do not use console.log (use the project logger)
- Do not modify database schema without migration files
```

---

## Token Budgets for System Prompts

| Use Case | Recommended Budget | Notes |
|----------|-------------------|-------|
| Simple chat assistant | 200-500 tokens | Basic persona + rules |
| Code review | 500-1,500 tokens | Standards + what to check |
| Agentic coding | 1,000-3,000 tokens | Role + tools guidance + rules |
| Complex enterprise agent | 2,000-5,000 tokens | Full context + compliance |

**Rule of thumb**: If your system prompt exceeds 5,000 tokens, you're probably over-specifying. Move examples and edge cases to retrieval.

---

## Optimization Techniques

### 1. Remove Redundancy
```
❌ "Make sure to always use TypeScript. All code should be
    written in TypeScript. Do not write JavaScript."

✅ "Use TypeScript for all code."
```

### 2. Use Structured Formats
```
❌ "The project uses React for the frontend, Node.js for the
    backend, PostgreSQL for the database, and Redis for caching."

✅ Stack: React | Node.js | PostgreSQL | Redis
```

### 3. Reference Instead of Inline
```
❌ [Entire style guide pasted in system prompt — 3,000 tokens]

✅ "Follow the code style defined in .eslintrc.json and
    .prettierrc. Read these files before making changes."
```

### 4. Prioritize by Impact
Put the most critical rules first (strongest recall position):
1. Security rules (never skip)
2. Architecture constraints
3. Code quality rules
4. Style preferences (weakest — let linters handle)

---

## Key Takeaways

1. **Right altitude** — Specific enough to guide, flexible enough to generalize
2. **Structure with sections** — XML tags or Markdown headers
3. **Budget: 1K-3K tokens** — For most coding use cases
4. **Reference, don't inline** — Let the agent read files on demand
5. **Critical rules first** — Strongest recall at the start of context
