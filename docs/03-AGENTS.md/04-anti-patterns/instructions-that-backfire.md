# Instructions That Backfire

> Real-world examples of AGENTS.md instructions that cause more harm than good — and how to fix each one.

---

## Category 1: Over-Constraining

### Backfire: "Never create new files"

```markdown
❌ ## Boundaries
   - NEVER create new files without explicit permission
```

**What happens**: The agent can't extract helper functions, create test files, add type definitions, or split large files. It stuffs everything into existing files, creating bloated, unmaintainable code.

**Fix**:
```markdown
✅ ## Boundaries
   - NEVER create files outside of src/, tests/, or prisma/
   - New files must follow the existing directory structure
```

Constrain *where*, not *whether*.

---

### Backfire: "Never modify existing tests"

```markdown
❌ ## Boundaries
   - NEVER modify existing test files
```

**What happens**: When the agent changes function signatures or behavior, existing tests break. The agent can't update them, so the PR ships with failing tests.

**Fix**:
```markdown
✅ ## Testing
   - Update existing tests when function signatures or behavior change
   - NEVER delete existing test cases without replacement
   - NEVER modify test snapshots manually (run `pnpm test:update-snapshots`)
```

---

### Backfire: "Always add comprehensive error handling"

```markdown
❌ ## Conventions
   - Always add comprehensive error handling to every function
```

**What happens**: The agent wraps every function in try-catch, including pure functions that can't throw. Code becomes bloated with unnecessary error handling. Errors get swallowed silently.

**Fix**:
```markdown
✅ ## Error Handling
   - Use try-catch only for I/O operations (API calls, DB queries, file access)
   - Pure functions: let errors propagate naturally
   - Pattern: catch → log → rethrow or return typed error
```

---

## Category 2: Ambiguous Instructions

### Backfire: "Follow existing patterns"

```markdown
❌ ## Conventions
   - Follow existing patterns in the codebase
```

**What happens**: Your codebase has multiple patterns — some legacy, some current. The agent may follow either one. Legacy code becomes the template for new code.

**Fix**:
```markdown
✅ ## Conventions
   - Follow patterns in src/features/users/ as the reference implementation
   - IGNORE patterns in src/legacy/ — those are deprecated
```

Name the specific directory that represents your current standard.

---

### Backfire: "Use appropriate error codes"

```markdown
❌ ## API
   - Return appropriate HTTP status codes
```

**What happens**: "Appropriate" is subjective. The agent returns 200 for everything, or uses obscure status codes like 418 (I'm a teapot) or 422 (Unprocessable Entity) inconsistently.

**Fix**:
```markdown
✅ ## API Status Codes
   - 200: Successful GET, PUT, PATCH
   - 201: Successful POST (resource created)
   - 204: Successful DELETE (no content)
   - 400: Invalid input (validation error)
   - 401: Not authenticated
   - 403: Not authorized
   - 404: Resource not found
   - 500: Server error (never intentionally returned)
```

---

### Backfire: "Keep code DRY"

```markdown
❌ ## Conventions
   - Follow DRY principles — don't repeat yourself
```

**What happens**: The agent over-abstracts. Creates shared utilities for two similar but not identical pieces of code. Introduces premature abstractions that make the code harder to understand and maintain.

**Fix**:
```markdown
✅ ## Conventions
   - Extract shared logic only after 3+ identical usages
   - Prefer duplication over wrong abstraction
   - Utilities go in src/lib/; feature-specific helpers stay in feature directory
```

---

## Category 3: Performance-Killing Instructions

### Backfire: "Add detailed comments to all code"

```markdown
❌ ## Conventions
   - Add detailed comments explaining what each function does
```

**What happens**: The agent adds obvious comments everywhere:
```typescript
// Get the user by ID
function getUserById(id: string) { ... }

// Add one to the count
count += 1;
```

Comments become noise. Actual complex logic remains uncommented.

**Fix**:
```markdown
✅ ## Comments
   - JSDoc on exported functions only
   - Inline comments only for non-obvious business logic
   - NEVER comment obvious operations
```

---

### Backfire: "Always write the most performant code"

```markdown
❌ ## Conventions
   - Always optimize for performance
```

**What happens**: The agent premature-optimizes everything. Uses bitwise operations instead of math. Avoids array methods in favor of manual loops. Creates unreadable code for negligible performance gains.

**Fix**:
```markdown
✅ ## Performance
   - Optimize only hot paths identified by profiling
   - Prefer readability over micro-optimization
   - Use database indexes for query performance, not application-level caching
```

---

### Backfire: "Use the latest features"

```markdown
❌ ## Stack
   - Always use the latest TypeScript / React features
```

**What happens**: The agent uses experimental APIs, stage-3 proposals, or features not yet supported by your build tools. Builds break.

**Fix**:
```markdown
✅ ## Stack
   TypeScript 5.4, React 18, Next.js 15.
   - Use only stable, GA features
   - No experimental React APIs (useOptimistic, useFormState, etc.)
   - No TypeScript decorators (not stable in our build config)
```

---

## Category 4: Security-Damaging Instructions

### Backfire: "Use environment variables for configuration"

```markdown
❌ ## Configuration
   - Store all configuration in environment variables
```

**What happens**: The agent creates `.env` files with hardcoded secrets and commits them. Or it reads sensitive environment variables and logs them for debugging.

**Fix**:
```markdown
✅ ## Configuration
   - Read config from process.env (already loaded by dotenv)
   - NEVER create .env files
   - NEVER log or expose environment variable values
   - NEVER hardcode secrets, tokens, or credentials
   - Use .env.example for documenting required variables (no real values)
```

---

### Backfire: "Handle authentication flexibly"

```markdown
❌ ## Auth
   - Support multiple authentication methods flexibly
```

**What happens**: The agent implements authentication with bypass options, fallback to no-auth mode, or optional token validation — creating security vulnerabilities.

**Fix**:
```markdown
✅ ## Auth
   - All /api/ routes (except /api/health) require authentication
   - Auth: NextAuth.js v5 session middleware
   - NEVER bypass auth, even in development
   - NEVER create "admin" backdoor endpoints
   - NEVER accept tokens from query parameters
```

---

## Category 5: Workflow-Breaking Instructions

### Backfire: "Always create a new branch"

```markdown
❌ ## Git
   - Always create a new branch for your changes
```

**What happens**: Agent is already on the correct feature branch but creates a sub-branch from it. Orphaned branches multiply. PR targeting becomes confused.

**Fix**:
```markdown
✅ ## Git
   - Work on the branch provided by the task/issue
   - If no branch specified, create: `feat/description` from `main`
   - NEVER push directly to `main` or `develop`
```

---

### Backfire: "Run all tests before committing"

```markdown
❌ ## Testing
   - Run the full test suite before every commit
```

**What happens**: In a monorepo with 10,000 tests, the agent runs the full suite before each of its 15 commits. Builds take hours. Agent times out.

**Fix**:
```markdown
✅ ## Testing
   - Before committing: run tests for changed files only: `pnpm test --changed`
   - Before opening PR: run full suite: `pnpm test`
```

---

## The Backfire Checklist

Before adding any rule to AGENTS.md, ask:

| Question | Risk If "No" |
|----------|-------------|
| Can this rule be followed literally? | Agent over-applies it |
| Does this rule have edge cases? | Agent applies it blindly |
| Could this rule prevent necessary actions? | Agent gets stuck |
| Is this rule testable/measurable? | Agent interprets subjectively |
| Does this rule conflict with any other rule? | Agent chooses randomly |

If any answer is "no," refine the rule before adding it.
