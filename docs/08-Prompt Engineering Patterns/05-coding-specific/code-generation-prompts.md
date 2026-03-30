# Code Generation Prompts

> Writing effective prompts that produce correct, production-ready code.

---

## The Code Generation Formula

```
[Context] + [Specification] + [Constraints] + [Pattern Reference] = Good Code
```

---

## Prompt Levels

### Level 1: Bare Request (Avoid)
```
Write a function to validate email
```
Result: Generic function, unknown language, no error handling.

### Level 2: Specified (Minimum)
```
Write a TypeScript function validateEmail(email: string): boolean
that checks for valid email format using a regex.
Return true for valid emails, false for invalid.
```
Result: Functional but may not match your patterns.

### Level 3: Contextualized (Recommended)
```
Create a TypeScript email validation function for our Express API.

Requirements:
- Input: string from request body
- Returns: { valid: boolean; error?: string }
- Validate: format, length < 254 chars, no consecutive dots
- Follow our validation pattern in src/validators/phone.ts

Constraints:
- Use Zod for schema validation
- Throw AppError for invalid input (not generic Error)
- Export from src/validators/email.ts
```
Result: Production-ready, matches your codebase.

---

## Key Principles

### 1. Reference Existing Patterns
```
Follow the pattern established in src/api/users.controller.ts
for error handling and response format.
```

The model will read and mirror the referenced file. This is more reliable than describing the pattern.

### 2. Specify the Interface First
```
Implement this interface:

interface CacheService {
  get<T>(key: string): Promise<T | null>;
  set<T>(key: string, value: T, ttl?: number): Promise<void>;
  delete(key: string): Promise<void>;
  clear(): Promise<void>;
}

Implementation requirements:
- Use Redis as backend
- Default TTL: 3600 seconds
- Serialize values as JSON
- Handle connection errors gracefully
```

### 3. Include Test Cases as Specification
```
Write a function that passes these tests:

test('parsePrice("$1,234.56") returns 1234.56', ...)
test('parsePrice("€99") returns 99', ...)
test('parsePrice("free") returns 0', ...)
test('parsePrice("") throws InvalidPriceError', ...)
```

Tests as specification is extremely effective — it's unambiguous.

### 4. One Function at a Time
```
❌ "Build the user service with CRUD operations"
✅ "Write the createUser function. We'll add others next."
```

---

## Anti-Patterns in Code Generation Prompts

| Anti-Pattern | Problem | Better Approach |
|-------------|---------|-----------------|
| "Make it production-ready" | Vague quality signal | Specify exact requirements |
| "Handle all edge cases" | Model invents unlikely cases | List the specific edge cases |
| "Optimize for performance" | Premature optimization | "This handles 10K items max" |
| "Use best practices" | Everyone's best practices differ | "Use early returns, Zod validation" |
| "Like the old version but better" | No clear target | Specify exactly what to change |

---

## Language-Specific Tips

### TypeScript
```
- Use strict types, no `any`
- Prefer interfaces over types for object shapes
- Use discriminated unions for state
- All public functions need explicit return types
```

### Python
```
- Type hints on all function signatures
- Use dataclasses or Pydantic models
- Follow PEP 8 naming conventions
- Include docstrings with Args/Returns
```

### React
```
- Functional components only
- Props as TypeScript interface (not inline)
- Custom hooks for logic extraction
- Error boundaries around async operations
```
