# Instructions That Backfire

> Real-world examples of instructions that made Copilot's output worse, not better.

---

## Why Instructions Can Backfire

Good intentions + poor execution = worse AI output. Instructions backfire when they:
1. Create ambiguity instead of clarity
2. Overconstrain, forcing unnatural code
3. Conflict with the codebase reality
4. Waste context on low-value rules

---

## Case 1: The "Be Comprehensive" Instruction

### The Instruction

```markdown
Always write comprehensive error handling for every function.
Include try/catch blocks with detailed logging for all operations.
```

### What Happened

Every function Copilot generated looked like:

```typescript
function add(a: number, b: number): number {
  try {
    const result = a + b;
    logger.info('Addition successful', { a, b, result });
    return result;
  } catch (error) {
    logger.error('Addition failed', { a, b, error });
    throw new AppError('Addition operation failed', 500);
  }
}
```

**Problem**: Pure arithmetic can't throw. The try/catch is useless noise. Every simple function became 10 lines of boilerplate.

### The Fix

```markdown
Add error handling for operations that can fail:
- Network requests, database queries, file I/O
- Parse operations (JSON.parse, parseInt)
- Not needed for: pure calculations, string operations, type conversions
```

---

## Case 2: The "Never Use X" Overreach

### The Instruction

```markdown
Never use `any` in TypeScript. Every value must have an explicit type.
```

### What Happened

When dealing with third-party library types or complex generics, Copilot produced:

```typescript
// Instead of: const data = JSON.parse(response)
const data: Record<string, unknown> = JSON.parse(response) as Record<string, unknown>;
const items: Array<Record<string, unknown>> = (data as Record<string, Array<Record<string, unknown>>>).items as Array<Record<string, unknown>>;
```

**Problem**: Replacing `any` with chains of type assertions is **worse** — it's less readable and less safe.

### The Fix

```markdown
Prefer explicit types over `any`. When dealing with unknown external data,
use `unknown` with type guards or Zod parsing, not type assertions.
```

---

## Case 3: The Aspirational Architecture

### The Instruction

```markdown
All business logic must follow the Clean Architecture pattern:
- Entities → Use Cases → Interface Adapters → Frameworks
- Dependencies always point inward
- Use dependency injection for all services
```

### What Happened

A simple CRUD endpoint became:

```
src/
  entities/User.ts
  use-cases/CreateUserUseCase.ts
  interfaces/UserRepository.ts
  adapters/UserRepositoryImpl.ts
  frameworks/express/UserController.ts
  di/UserModule.ts
```

For 20 lines of actual logic, Copilot created 6 files with ~150 lines of boilerplate.

**Problem**: Clean Architecture is valid for complex domains but overkill for simple CRUD. The instruction applied uniformly to everything.

### The Fix

```markdown
Simple CRUD: route → service → database (minimal abstraction)
Complex domain logic: use the service pattern in src/services/ with interfaces
```

---

## Case 4: The Conflicting Migration

### The Instruction

```markdown
## Styling
Use CSS Modules for component styling.
Import styles as: import styles from './Component.module.css'
```

### The Reality

The team had started migrating to Tailwind CSS. Half the components used CSS Modules, half used Tailwind. The instruction hadn't been updated.

### What Happened

Copilot created CSS Module files for new components, even when the surrounding code used Tailwind. New code was inconsistent with its neighbors.

### The Fix

```markdown
## Styling
Use Tailwind CSS for all new components.
Legacy CSS Modules exist in some components — leave them as-is, do not convert.
```

---

## Case 5: The "Always Add Tests" Without Context

### The Instruction

```markdown
Always create unit tests for every file you modify.
```

### What Happened

When Copilot was asked to fix a typo in a constant file:

```typescript
// constants.ts
export const API_VERSION = 'v2'; // changed from 'v1'
```

Copilot also created:

```typescript
// constants.test.ts
describe('constants', () => {
  it('should export API_VERSION as v2', () => {
    expect(API_VERSION).toBe('v2');
  });
});
```

**Problem**: Testing that a constant equals a hardcoded value is meaningless.

### The Fix

```markdown
Write tests for logic and behavior:
- Functions with conditional logic
- API route handlers
- Data transformations
Do NOT write tests for: constants, type definitions, re-exports, config files.
```

---

## Case 6: The Overly Detailed Structure Map

### The Instruction

```markdown
## Project Structure
src/
  app/
    (auth)/
      login/
        page.tsx           # Login page
        actions.ts         # Login server actions
        schema.ts          # Zod validation schema
      register/
        page.tsx           # Registration page
        actions.ts         # Register server actions
    (dashboard)/
      page.tsx             # Dashboard home
      analytics/
        page.tsx           # Analytics view
        components/
          Chart.tsx         # Chart component
          ... [200 more lines] ...
```

### What Happened

250 lines of directory listing consumed ~2,000 tokens — leaving almost no budget for actual conventions and commands.

**Problem**: File-level detail wastes context and becomes stale after every new file.

### The Fix

```markdown
## Structure
src/app/          # Next.js App Router — route groups: (auth), (dashboard)
src/components/   # Shared React components
src/lib/          # Utilities and business logic
src/server/       # Server-only code (DB, auth)
prisma/           # Database schema and migrations
```

5 lines instead of 250. Budget preserved.

---

## Case 7: The Security Theater Instruction

### The Instruction

```markdown
Always sanitize all input to prevent security vulnerabilities.
```

### What Happened

Copilot added HTML escaping to database field names, URL-encoded internal constants, and sanitized integer IDs:

```typescript
const userId = sanitizeInput(req.params.id); // ID is already validated as UUID by middleware
const query = sanitizeSQL(`SELECT * FROM ${sanitizeInput('users')}`); // Table name is hardcoded
```

**Problem**: "Sanitize everything" creates noise and false security. Real security requires targeted measures.

### The Fix

```markdown
## Security
- User-facing text output: escape HTML using `escapeHtml()` from `src/lib/security`
- Database queries: always use Prisma Client (parameterized by default)
- API input: validate with Zod schemas (handles type coercion and bounds)
- Internal strings and constants: no sanitization needed
```

---

## Pattern Recognition: When Instructions Backfire

| Sign | Root Cause | Fix |
|------|-----------|-----|
| Copilot adds boilerplate everywhere | Instruction is too broad | Add scope limitations ("only for X") |
| Output is more complex than input | Architecture instruction too rigid | Match instruction complexity to task complexity |
| Code doesn't match surrounding files | Instruction conflicts with codebase reality | Update instruction or add migration note |
| Meaningless tests generated | "Always write tests" without scope | Specify when tests are/aren't needed |
| Context budget blown on low-value | Detailed structure/explanation in instructions | Compress to essentials |
| Over-engineered simple tasks | Instruction mandates heavy patterns | "Simple tasks use simple patterns" |

---

## The Backfire Prevention Checklist

Before committing a new instruction, ask:

- [ ] Does this instruction have a **scope limit**? (When does it NOT apply?)
- [ ] Is this instruction **proportional**? (Does it force overhead on simple tasks?)
- [ ] Does this instruction match **current codebase reality**?
- [ ] Have you **tested** it on both simple and complex tasks?
- [ ] Would a new developer **understand** the intent without extra context?
