# Decomposition

> Breaking complex tasks into subtasks — the foundation of reliable AI-assisted development.

---

## Why Decompose?

AI models perform dramatically better on focused, well-scoped tasks than on broad, vague ones. Decomposition is the single most impactful technique for improving code generation quality.

```
❌ "Build me a user authentication system"
   → Model tries to do everything, misses details, inconsistent

✅ Decomposed into:
   1. "Design the User schema with Prisma"
   2. "Create the registration endpoint with Zod validation"
   3. "Implement password hashing with bcrypt"
   4. "Build the login endpoint that returns a JWT"
   5. "Write the auth middleware for protected routes"
   → Each task is focused, testable, correct
```

---

## Decomposition Strategies

### 1. Functional Decomposition
Break by function or feature:
```
Task: Build a file upload API
├── 1. File validation (size, type, name)
├── 2. Upload handler (multipart parsing)
├── 3. Storage service (S3 integration)
├── 4. Database record (track uploads)
└── 5. Cleanup job (delete expired files)
```

### 2. Layer Decomposition
Break by architectural layer:
```
Task: Add user search feature
├── 1. Database: Add search index, write query
├── 2. API: Create search endpoint with pagination
├── 3. Service: Business logic, filtering, sorting
├── 4. Frontend: Search component, debounced input
└── 5. Tests: Unit + integration tests
```

### 3. Incremental Decomposition
Break by complexity level:
```
Task: Complex form with validation
├── 1. Basic form with fields (no validation)
├── 2. Add client-side validation
├── 3. Add server-side validation
├── 4. Add error display
└── 5. Add edge cases (network errors, race conditions)
```

---

## Decomposition in Prompts

### Explicit Subtask Definition
```
I need to implement a caching layer. Let's break this into steps.
Do ONLY step 1 right now:

Step 1: Define the cache interface
- Create CacheService interface with get, set, delete, clear methods
- Define TTL options type
- Export from src/services/cache/types.ts

Do NOT implement the cache yet. Just the interface.
```

### Let the Model Decompose
```
I need to add WebSocket support to our Express server.
First, break this into 5-7 implementation steps, ordered by dependency.
Don't implement anything yet — just list the steps with:
- What the step produces
- What it depends on
- Estimated complexity (small/medium/large)
```

---

## The One-Task-at-a-Time Principle

Each prompt should have exactly **one clear deliverable**:

| ❌ Multi-Task | ✅ Single-Task |
|-------------|--------------|
| "Create the schema, API, and tests" | "Create the Prisma schema for User" |
| "Refactor and add error handling" | "Refactor the auth module to use async/await" |
| "Review and fix this code" | "Review this code and list the issues" |

Then in the next prompt: "Fix issue #1 from your review."

---

## Decomposition Patterns for Teams

### Feature Decomposition Template
```markdown
## Feature: [Name]

### Subtasks (one per prompt):
1. [ ] Types/Interfaces — Define data shapes
2. [ ] Database — Schema + migrations
3. [ ] Repository — Data access layer
4. [ ] Service — Business logic
5. [ ] API — Endpoint + validation
6. [ ] Frontend — UI components
7. [ ] Tests — Unit + integration
8. [ ] Documentation — API docs, README updates
```

### Complexity-Based Routing
```
Simple (1 subtask): Direct prompt
Medium (2-4 subtasks): Sequential prompts
Complex (5+ subtasks): Decompose → Plan → Execute sequentially
```
