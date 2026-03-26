# Backend AGENTS.md Examples

> Complete AGENTS.md files for API services, microservices, and backend-focused projects.

---

## Node.js + Express + TypeScript

```markdown
# AGENTS.md

## Setup
- Package manager: pnpm
- Install: `pnpm install`
- Dev: `pnpm dev` (http://localhost:4000)
- Test: `pnpm test`
- Test single: `pnpm test -- --testPathPattern=path/to/test`
- Lint: `pnpm lint`
- Build: `pnpm build`
- DB migrate: `pnpm db:migrate`
- DB seed: `pnpm db:seed`

## Stack
TypeScript 5.4, Node.js 20, Express 5, Prisma 5.x, PostgreSQL 16,
Zod validation, JWT auth, Vitest, Supertest.

NOT: Mongoose, MongoDB, Sequelize, callback-style code.

## Structure
src/routes/      — Express route handlers (grouped by domain)
src/services/    — Business logic (one service per domain)
src/middleware/   — Express middleware (auth, validation, errors)
src/lib/         — Utilities, logger, config
src/types/       — TypeScript types and Zod schemas
prisma/          — Database schema and migrations
tests/           — Test files (mirrors src/ structure)

## Conventions
- async/await everywhere; never callbacks or .then()
- Validate all input with Zod schemas from src/types/
- Service layer pattern: routes → validation → service → response
- Custom AppError class for all errors (never raw throw)
- Return typed responses: `{ data, meta }` for success, `{ error }` for failure
- HTTP status codes: 200 GET, 201 POST, 204 DELETE, 400 invalid, 401 unauth, 404 missing

## API Route Pattern
```typescript
// src/routes/users.ts
import { Router } from 'express';
import { CreateUserSchema } from '../types/user';
import { userService } from '../services/user';
import { validate } from '../middleware/validate';
import { requireAuth } from '../middleware/auth';

const router = Router();

router.post('/', requireAuth, validate(CreateUserSchema), async (req, res) => {
  const user = await userService.create(req.validated);
  res.status(201).json({ data: user });
});

export { router as userRoutes };
```

## Boundaries
- NEVER use raw SQL (use Prisma)
- NEVER commit .env files or secrets
- NEVER return stack traces in responses (log them, return generic error)
- NEVER modify existing migration files
- NEVER disable authentication middleware
- NEVER use console.log (use src/lib/logger)

## Testing
- Framework: Vitest + Supertest
- Unit tests for services, integration tests for routes
- Mock: database (Prisma mock), external APIs
- Pattern: Arrange → Act → Assert
- Each test file uses isolated test database transaction
```

---

## Python + FastAPI

```markdown
# AGENTS.md

## Setup
- Python: 3.12+
- Package manager: uv
- Install: `uv sync`
- Dev: `uv run uvicorn app.main:app --reload`
- Test: `uv run pytest -xvs`
- Test single: `uv run pytest -xvs tests/test_users.py`
- Lint: `uv run ruff check .`
- Format: `uv run ruff format .`
- Type check: `uv run mypy .`
- DB migrate: `uv run alembic upgrade head`

## Stack
Python 3.12, FastAPI 0.115, Pydantic v2, SQLAlchemy 2.0,
Alembic, PostgreSQL 16, pytest, httpx.

NOT: Django, Flask, Marshmallow, raw psycopg2.

## Structure
app/
├── main.py          — FastAPI app creation and startup
├── routers/         — API route handlers
├── services/        — Business logic
├── models/          — SQLAlchemy ORM models
├── schemas/         — Pydantic request/response schemas
├── dependencies/    — FastAPI dependencies (auth, db session)
└── core/            — Config, security, logging
tests/               — pytest tests (mirrors app/ structure)
alembic/             — Database migrations

## Conventions
- Type hints on all functions (params and return)
- Pydantic v2 models for all API input/output
- Dependency injection for DB sessions and auth
- async everywhere; never sync DB calls in async routes
- Raise HTTPException for client errors, let middleware handle server errors

## Endpoint Pattern
```python
@router.post("/users", response_model=UserResponse, status_code=201)
async def create_user(
    body: CreateUserRequest,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(require_admin),
) -> UserResponse:
    user = await user_service.create(db, body)
    return UserResponse.model_validate(user)
```

## Boundaries
- NEVER use raw SQL (use SQLAlchemy ORM)
- NEVER commit .env files
- NEVER modify existing Alembic migration files
- NEVER use print() for logging (use structlog)
- NEVER disable authentication dependencies
- NEVER use * imports

## Testing
- Framework: pytest + httpx (async)
- Fixtures: database session, auth tokens, test client
- Pattern: Arrange → Act → Assert
- Mock: external APIs only (test against real DB with transactions)
```

---

## Go Microservice

```markdown
# AGENTS.md

## Setup
- Go: 1.22+
- Build: `go build ./...`
- Test: `go test ./... -v`
- Test single: `go test -v -run TestFunctionName ./path/to/package`
- Lint: `golangci-lint run`
- Run: `go run cmd/server/main.go`

## Stack
Go 1.22, Chi router, sqlc (generated SQL), PostgreSQL 16,
testify, mockgen, Docker.

NOT: GORM, Gin, Fiber, global state.

## Structure
cmd/server/       — Application entry point
internal/
├── handler/      — HTTP handlers (Chi routes)
├── service/      — Business logic
├── repository/   — Database access (sqlc generated)
├── model/        — Domain types
└── middleware/    — HTTP middleware (auth, logging, recovery)
pkg/              — Shared libraries (logger, config)
sql/
├── queries/      — sqlc query files (.sql)
└── migrations/   — Database migrations

## Conventions
- Error values returned, never panic (except truly unrecoverable)
- Context as first parameter: `func DoThing(ctx context.Context, ...) error`
- Interfaces defined by consumers, not producers
- Table-driven tests
- Explicit error wrapping: `fmt.Errorf("operation: %w", err)`

## Boundaries
- NEVER use panic for error handling
- NEVER use global variables (pass dependencies explicitly)
- NEVER use GORM or reflection-based ORMs
- NEVER modify generated sqlc files (modify .sql queries instead)
- NEVER commit .env files

## Testing
- Framework: testing + testify
- Table-driven tests for all business logic
- Mock: interfaces with mockgen
- Integration: test against real PostgreSQL (dockerized)
```

---

## Key Backend Anti-Patterns to Prevent

| Anti-Pattern | AGENTS.md Rule |
|-------------|----------------|
| N+1 queries | "Use eager loading / batch queries for related data" |
| Raw SQL injection | "NEVER use string concatenation in queries; use parameterized queries" |
| Secrets in code | "NEVER hardcode credentials; use environment variables" |
| Missing auth | "All routes except /health require authentication middleware" |
| Logging PII | "NEVER log user email, name, or other PII" |
| Missing validation | "All API input validated with [Zod/Pydantic/struct tags]" |
| Sync in async | "NEVER use blocking I/O in async handlers" |

---

*Next: [DevOps & Infrastructure](devops-infrastructure.md) →*
