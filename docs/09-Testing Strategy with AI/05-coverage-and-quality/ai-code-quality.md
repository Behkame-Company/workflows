# AI-Generated Code Quality

> Validating that AI code meets quality standards — catching the systematic issues AI introduces before they reach production.

## Common AI Code Quality Issues

AI-generated code has predictable failure patterns. Understanding them helps you review AI output more effectively and build automated guardrails.

### 1. Works for Happy Path, Fails on Edge Cases

AI tends to generate code that handles the "expected" scenario perfectly but falls apart on edge cases:

```typescript
// AI-generated — looks correct at first glance
function parseAge(input: string): number {
  return parseInt(input, 10);
}

// Problems the AI didn't handle:
parseAge('')         // → NaN (no empty check)
parseAge('abc')      // → NaN (no validation)
parseAge('-5')       // → -5 (negative age?)
parseAge('25.7')     // → 25 (silent truncation)
parseAge('999')      // → 999 (unreasonable value)
parseAge(null as any) // → NaN (no type guard at runtime)
```

```typescript
// What production-quality code looks like
function parseAge(input: string): number {
  if (!input || input.trim() === '') {
    throw new ValidationError('Age is required');
  }

  const parsed = Number(input);

  if (!Number.isFinite(parsed) || !Number.isInteger(parsed)) {
    throw new ValidationError(`Invalid age: "${input}"`);
  }

  if (parsed < 0 || parsed > 150) {
    throw new ValidationError(`Age out of range: ${parsed}`);
  }

  return parsed;
}
```

### 2. Over-Engineered Solutions

AI sometimes generates unnecessarily complex code, especially when context is ambiguous:

```typescript
// AI-generated: Asked to "create a configuration loader"
class ConfigurationManager {
  private static instance: ConfigurationManager;
  private config: Map<string, unknown>;
  private observers: Set<(key: string, value: unknown) => void>;
  private cache: WeakMap<object, unknown>;
  private encryptionKey: string;

  private constructor() { /* ... 200 lines of setup */ }

  static getInstance(): ConfigurationManager { /* ... */ }
  subscribe(observer: (key: string, value: unknown) => void): void { /* ... */ }
  decrypt(value: string): string { /* ... */ }
  // ... 15 more methods
}

// What was actually needed:
interface AppConfig {
  port: number;
  dbUrl: string;
  logLevel: 'debug' | 'info' | 'warn' | 'error';
}

function loadConfig(): AppConfig {
  return {
    port: parseInt(process.env.PORT ?? '3000', 10),
    dbUrl: process.env.DATABASE_URL ?? 'postgres://localhost:5432/app',
    logLevel: (process.env.LOG_LEVEL as AppConfig['logLevel']) ?? 'info',
  };
}
```

### 3. Inconsistent Patterns

AI doesn't inherently know your project conventions. It may mix patterns:

```typescript
// In the same PR, AI generated three different error handling patterns:

// File 1: Throws exceptions
async function getUser(id: string): Promise<User> {
  const user = await db.users.findById(id);
  if (!user) throw new NotFoundError('User not found');
  return user;
}

// File 2: Returns null
async function getOrder(id: string): Promise<Order | null> {
  return await db.orders.findById(id);
}

// File 3: Returns Result type
async function getProduct(id: string): Promise<Result<Product, Error>> {
  const product = await db.products.findById(id);
  if (!product) return { ok: false, error: new Error('Not found') };
  return { ok: true, value: product };
}
```

### 4. Subtle Logic Errors

AI-generated code can contain logic errors that look correct on casual review:

```typescript
// AI-generated — spot the bug
function calculateDiscountedPrices(items: CartItem[]): number {
  let total = 0;
  for (const item of items) {
    const discount = item.quantity > 10 ? 0.1 : 0;
    total += item.price * item.quantity * discount;  // Bug: applies ONLY the discount, not the discounted price
  }
  return total;
}

// Correct version
function calculateDiscountedPrices(items: CartItem[]): number {
  let total = 0;
  for (const item of items) {
    const discount = item.quantity > 10 ? 0.1 : 0;
    total += item.price * item.quantity * (1 - discount);
  }
  return total;
}
```

## Verification Checklist

Use this checklist for every AI-generated code change:

### Correctness

- [ ] **Null/undefined handling** — Does it handle null, undefined, and empty inputs?
- [ ] **Error handling** — Does it catch and handle errors appropriately?
- [ ] **Edge cases** — Does it work with empty arrays, zero values, max values, special characters?
- [ ] **Boundary conditions** — Are off-by-one errors possible? Are comparisons correct (`>` vs `>=`)?
- [ ] **Async correctness** — Are promises awaited? Are race conditions possible?

### Project Fit

- [ ] **Follows project patterns** — Does it use the same error handling, naming, and structure patterns as existing code?
- [ ] **Type safety** — Are types specific (not `any`)? Are generics used correctly?
- [ ] **Dependencies** — Does it import from the right places? Does it avoid unnecessary new dependencies?
- [ ] **Configuration** — Does it use the project's config system, not hardcoded values?

### Robustness

- [ ] **Passes existing tests** — Does it break anything?
- [ ] **Has new tests** — Are the new tests meaningful (not just `toBeDefined()`)?
- [ ] **Handles concurrent access** — If applicable, is it safe for concurrent use?
- [ ] **Logging/observability** — Does it log errors? Can you debug it in production?

## Using AI to Review AI Code (Self-Critique)

One of the most effective quality strategies: ask a different AI interaction to review the code generated by the first.

### Prompt for AI Code Review

```
Review this AI-generated code for production readiness.

[paste the code]

Check for:
1. Edge cases that aren't handled (null, empty, negative, overflow)
2. Error handling gaps (what happens when this fails?)
3. Type safety issues (any casts, missing types)
4. Logic errors (off-by-one, wrong operators, incorrect formulas)
5. Security issues (injection, data exposure, auth bypass)
6. Performance issues (N+1 queries, unnecessary allocations)

For each issue found, rate severity (critical/warning/info)
and provide the fix.
```

### Example Self-Critique Output

```
## Review of AI-generated OrderService.processRefund()

### 🔴 Critical: No idempotency check
Line 15: processRefund() can be called multiple times for the same order,
resulting in multiple refund charges.
Fix: Check if refund already exists before processing.

### 🔴 Critical: Race condition on balance update
Lines 22-24: Read balance, calculate new balance, write balance — not atomic.
Concurrent refunds could result in incorrect balance.
Fix: Use database transaction with row-level locking.

### 🟡 Warning: Missing input validation
Line 8: `amount` parameter is not validated. Negative amounts would
credit the customer's account.
Fix: Add `if (amount <= 0) throw new ValidationError(...)`.

### ℹ️ Info: Inconsistent error type
Line 30: Throws generic `Error` instead of project's `AppError` class.
Fix: Use `throw new AppError('REFUND_FAILED', message)`.
```

## Automated Quality Gates for AI PRs

### ESLint Rules That Catch AI Issues

```javascript
// .eslintrc.js — rules that catch common AI code problems
module.exports = {
  rules: {
    // Prevent `any` type (AI loves to use `any`)
    '@typescript-eslint/no-explicit-any': 'error',

    // Require error handling in promises
    '@typescript-eslint/no-floating-promises': 'error',

    // Prevent unused variables (AI generates dead code)
    '@typescript-eslint/no-unused-vars': 'error',

    // Require return types on functions
    '@typescript-eslint/explicit-function-return-type': 'warn',

    // Prevent console.log in production code
    'no-console': ['error', { allow: ['warn', 'error'] }],

    // Require strict equality
    'eqeqeq': 'error',
  },
};
```

### CI Pipeline Quality Gates

```yaml
# .github/workflows/ai-quality-gate.yml
name: AI Code Quality Gate
on: [pull_request]

jobs:
  quality-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci

      # Type checking
      - name: TypeScript strict check
        run: npx tsc --noEmit --strict

      # Linting
      - name: ESLint
        run: npx eslint 'src/**/*.ts' --max-warnings 0

      # Unit tests with coverage
      - name: Tests with coverage threshold
        run: npm test -- --coverage --ci

      # Check for common AI patterns
      - name: Check for any types
        run: |
          if grep -rn ': any' src/ --include='*.ts' | grep -v '.test.ts' | grep -v '.spec.ts'; then
            echo "::error::Found 'any' types in production code"
            exit 1
          fi

      # Mutation testing on changed files
      - name: Mutation test
        run: |
          CHANGED=$(git diff --name-only origin/main...HEAD -- 'src/**/*.ts' | grep -v '.test.' | tr '\n' ',')
          if [ -n "$CHANGED" ]; then
            npx stryker run --mutate "$CHANGED"
          fi
```

### Pre-Commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit — quick quality check before committing

# Run TypeScript compiler
npx tsc --noEmit 2>&1
if [ $? -ne 0 ]; then
  echo "❌ TypeScript errors found. Fix before committing."
  exit 1
fi

# Run tests for changed files
CHANGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep '\.ts$' | grep -v '\.test\.')
if [ -n "$CHANGED_FILES" ]; then
  npx jest --findRelatedTests $CHANGED_FILES --passWithNoTests 2>&1
  if [ $? -ne 0 ]; then
    echo "❌ Related tests failed. Fix before committing."
    exit 1
  fi
fi
```

## AI Quality Improvement Loop

```
┌─────────────────────────┐
│ AI generates code       │
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│ Automated checks        │  TypeScript, ESLint, tests
│ (fast, cheap)           │
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│ AI self-review          │  Different prompt reviews
│ (medium effort)         │  the generated code
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│ Mutation testing        │  Validates test quality
│ (thorough, slower)      │
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│ Human review            │  Focus on business logic,
│ (final gate)            │  architecture, security
└─────────────────────────┘
```

Each layer catches different categories of issues. Automated gates handle the mechanical checks so human reviewers can focus on the things only humans can evaluate: business correctness, architectural fit, and security implications.

## Next Steps

- [Coverage Analysis](./coverage-analysis.md) — Ensure AI-generated code is actually tested
- [Test Quality Metrics](./test-quality-metrics.md) — Measure whether tests for AI code are strong enough
- [Mutation Testing](./mutation-testing.md) — The ultimate validation that tests catch bugs in AI code
