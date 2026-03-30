# Domain Guide: Backend & API

> Instruction patterns for REST APIs, server-side logic, and middleware.

---

## Sample: api.instructions.md

```yaml
---
applyTo: "src/app/api/**/*.ts"
---
```

```markdown
# API Route Instructions

## Route Handlers
- Export named functions: GET, POST, PUT, DELETE, PATCH
- Validate all request input with Zod schemas from `src/validators/`
- Return typed responses using helper functions from `src/lib/api-response`
- Wrap handler body in try/catch; never let unhandled errors reach the client

## Request Handling
- Parse body: `const body = await req.json()`
- Parse params: `const { id } = params`
- Parse query: `const url = new URL(req.url); const page = url.searchParams.get('page')`
- Always validate before processing: `const validated = schema.parse(body)`

## Response Format
Success: `NextResponse.json({ data: result }, { status: 200 })`
Created: `NextResponse.json({ data: result }, { status: 201 })`
Error: `NextResponse.json({ error: message }, { status: code })`
No content: `new NextResponse(null, { status: 204 })`

## Authentication
- Protected routes: call `getServerSession()` at the top
- Return 401 for unauthenticated requests
- Return 403 for unauthorized requests (wrong role)
- Public routes: document why auth is not required in a comment

## Error Handling
- Catch Zod validation errors → return 400 with field-level errors
- Catch Prisma known errors → return appropriate 4xx status
- Catch unknown errors → log full error, return generic 500 message
- NEVER expose stack traces, SQL queries, or internal details in responses

## Do Not
- Never use `console.log` — use the logger from `src/lib/logger`
- Never write raw SQL — use Prisma Client
- Never trust client-provided IDs without ownership verification
- Never return more data than the client needs (select specific fields)
```

---

## Sample: middleware.instructions.md

```yaml
---
applyTo: "src/middleware/**/*.ts"
---
```

```markdown
# Middleware Instructions
- Middleware functions follow the pattern: `(req, res, next) => {}`
- Always call `next()` unless intentionally ending the chain
- Rate limiting middleware uses `src/lib/rate-limit`
- Auth middleware sets `req.user` after token verification
- Error middleware is always the LAST middleware registered
```

---

## Sample: services.instructions.md

```yaml
---
applyTo: "src/services/**/*.ts"
---
```

```markdown
# Service Layer Instructions

## Structure
- One service per domain entity (UserService, ProductService, OrderService)
- Services contain business logic; routes handle HTTP concerns
- Services receive dependencies via constructor (for testability)

## Patterns
- Methods are async, return typed results
- Validate business rules (not input format — that's the route's job)
- Throw domain-specific errors using AppError class
- Never reference HTTP concepts (status codes, headers) in services

## Transactions
- Multi-step operations use Prisma transactions: `prisma.$transaction()`
- If any step fails, the entire operation rolls back
- Log transaction boundaries for debugging
```

---

## Key API Conventions to Document

| Convention | Why |
|-----------|-----|
| Response format | Consistency across all endpoints |
| Error shape | Clients can handle errors predictably |
| Auth pattern | Every endpoint has clear auth requirements |
| Validation approach | Prevents invalid data from reaching business logic |
| Logging strategy | Consistent observability across services |
