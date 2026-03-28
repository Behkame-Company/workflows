# CLAUDE.md for Agentic Workflows

> Configuring Claude Code as an effective agent through CLAUDE.md — the persistent memory file that tells the agent everything it needs to know about your project.

---

## What Is CLAUDE.md?

CLAUDE.md is Claude Code's persistent configuration file. It's read at the start of every session, giving the agent immediate context about your project without you explaining it each time.

```
┌──────────────────────────────────────────────────────────────┐
│                CLAUDE.md Memory System                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   CLAUDE.md (Manual Memory)                                  │
│   ─────────────────────────                                  │
│   • You write and maintain                                   │
│   • Project context, commands, conventions                   │
│   • Checked into git, shared with team                       │
│   • Read at session start                                    │
│                                                              │
│   Auto-Memory (Deduced Memory)                               │
│   ────────────────────────────                               │
│   • Claude learns from your corrections                      │
│   • Stored locally (not in git)                              │
│   • Preferences, patterns, habits                            │
│   • Complements CLAUDE.md                                    │
│                                                              │
│   Hierarchy:                                                 │
│   Project CLAUDE.md  >  User ~/CLAUDE.md                     │
│   (repo-specific)       (personal defaults)                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Key Sections for Agents

An effective CLAUDE.md for agentic work covers these areas:

### 1. Project Context

```markdown
# Project: Acme API Gateway

API gateway service that handles authentication, rate limiting,
and request routing for the Acme microservices platform.
Built with Go 1.22, uses Chi router, PostgreSQL for config storage.
```

Keep this to 2-3 sentences. The agent needs to know *what* the project is, not its full history.

### 2. Build and Test Commands

```markdown
## Commands
- Build: `go build ./cmd/gateway`
- Test all: `go test ./...`
- Test single package: `go test ./internal/auth/...`
- Test single function: `go test ./internal/auth -run TestValidateToken`
- Lint: `golangci-lint run`
- Run locally: `go run ./cmd/gateway -config config.dev.yaml`
- Generate mocks: `go generate ./...`
```

Be exact. "Run tests" is vague. `go test ./... -race -count=1` is actionable.

### 3. Code Conventions

```markdown
## Conventions
- Error handling: always wrap errors with context using fmt.Errorf("operation: %w", err)
- Logging: use slog package, structured logging, never fmt.Println
- Naming: use Go conventions (exported = PascalCase, unexported = camelCase)
- Tests: table-driven tests, use testify/assert for assertions
- HTTP handlers: accept (w http.ResponseWriter, r *http.Request), return nothing
- Config: environment variables with GATEWAY_ prefix, parsed in cmd/gateway/main.go
```

### 4. File Structure

```markdown
## Structure
- cmd/gateway/          Entry point and CLI
- internal/auth/        Authentication middleware
- internal/proxy/       Request proxying and routing
- internal/ratelimit/   Rate limiting logic
- internal/config/      Configuration loading
- pkg/middleware/       Reusable HTTP middleware
- migrations/           SQL migration files
- api/                  OpenAPI specs
```

### 5. Common Tasks

```markdown
## Common Tasks

### Adding a new middleware
1. Create file in pkg/middleware/
2. Implement http.Handler interface
3. Add to middleware chain in internal/proxy/router.go
4. Write tests with httptest.NewRecorder
5. Update api/openapi.yaml if it affects request/response

### Adding a new migration
1. Create file: migrations/NNNN_description.up.sql
2. Create matching: migrations/NNNN_description.down.sql
3. Test: go run ./cmd/migrate up && go run ./cmd/migrate down
```

## Writing Effective Instructions

### Tip 1: Keep Instructions Concise

Models follow shorter, focused documents better than long ones:

```markdown
<!-- Good: concise and actionable -->
## Testing
- Run: `go test ./...`
- Use table-driven tests
- Mock external services with interfaces

<!-- Bad: verbose and wandering -->
## Testing
We use Go's built-in testing framework. Tests should be written for all
new code. We prefer table-driven tests because they make it easy to add
new test cases. When testing external services, you should create
interfaces and mock implementations. The mock should be in the same
package as the test. We used to use gomock but switched to manual mocks
because they're simpler. Here's the full history of why we made that
decision...
```

### Tip 2: Use Positive Instructions

Tell the agent what to do, not what to avoid:

```markdown
<!-- Good: positive instruction -->
- Use slog for all logging
- Wrap errors with fmt.Errorf("context: %w", err)
- Return early on error

<!-- Less effective: negative instruction -->
- Don't use fmt.Println for logging
- Don't return bare errors without context
- Don't nest error handling deeply
```

### Tip 3: Specify Exact Commands

```markdown
<!-- Good: exact and copy-pasteable -->
- Test: `go test ./... -race -count=1`
- Lint: `golangci-lint run --fix`
- Build: `CGO_ENABLED=0 go build -o bin/gateway ./cmd/gateway`

<!-- Bad: vague -->
- Run the tests
- Use the linter
- Build the project
```

## Complete CLAUDE.md Template

```markdown
# Project: [Name]

[One-sentence description of what this project does]

## Commands
- Install: [exact command]
- Build: [exact command]
- Test all: [exact command]
- Test single: [exact command with placeholder]
- Lint: [exact command]
- Dev server: [exact command]
- Type check: [exact command if applicable]

## Structure
- [dir/]  [purpose]
- [dir/]  [purpose]
- [dir/]  [purpose]

## Conventions
- [Convention 1 — specific and actionable]
- [Convention 2 — specific and actionable]
- [Convention 3 — specific and actionable]
- [Convention 4 — specific and actionable]

## Patterns
- [Pattern name]: [brief description with file reference]
- [Pattern name]: [brief description with file reference]

## Testing
- Framework: [name]
- Test location: [where tests go relative to source]
- Mocking: [approach used]
- Required coverage: [if any]

## Common Tasks
### [Task 1 name]
1. [Step]
2. [Step]
3. [Step]

### [Task 2 name]
1. [Step]
2. [Step]
3. [Step]

## Do Not
- [Hard rule 1 — things that would break the project]
- [Hard rule 2 — security or data safety rules]
```

## CLAUDE.md for Different Agent Roles

Tailor CLAUDE.md sections for how you use the agent:

### For Feature Development

```markdown
## Feature Workflow
1. Read the spec in docs/specs/ first
2. Create types in pkg/types/ before implementation
3. Implement in small, testable functions
4. Run tests after each file change
5. Commit at each stable checkpoint
```

### For Code Review

```markdown
## Review Standards
- Check error handling completeness
- Verify no sensitive data in logs
- Ensure new public APIs have documentation
- Flag any raw SQL (must use query builder)
```

### For Bug Investigation

```markdown
## Debugging
- Logs: `kubectl logs -f deploy/gateway -n production`
- Metrics: Grafana dashboard at /d/gateway-overview
- Traces: Jaeger UI at http://localhost:16686
- Reproduce locally: `make run-local && curl http://localhost:8080/health`
```

## Personal vs. Project CLAUDE.md

```
┌──────────────────────────────────────────────────┐
│          CLAUDE.md Hierarchy                      │
├──────────────────────────────────────────────────┤
│                                                   │
│   ~/CLAUDE.md (personal)                          │
│   ──────────────────────                          │
│   • Your preferences across all projects          │
│   • Editor settings, preferred patterns           │
│   • Not shared with team                          │
│                                                   │
│   ./CLAUDE.md (project)                           │
│   ─────────────────────                           │
│   • Project-specific conventions                  │
│   • Shared with team via git                      │
│   • Overrides personal where they conflict        │
│                                                   │
│   ./src/feature/CLAUDE.md (subdirectory)          │
│   ──────────────────────────────────              │
│   • Directory-specific context                    │
│   • Additional context for that subtree           │
│                                                   │
└──────────────────────────────────────────────────┘
```

**Personal ~/CLAUDE.md example:**

```markdown
# Personal Preferences
- I prefer functional programming patterns
- Use async/await over callbacks or .then()
- Always explain your reasoning before making changes
- Commit messages: conventional commits format
```

## Maintaining CLAUDE.md

- **Update when conventions change** — stale instructions cause agent confusion
- **Remove outdated entries** — less is better than wrong
- **Review quarterly** — does the file still reflect reality?
- **Get team input** — CLAUDE.md works best when the whole team contributes

## Next Steps

- [Claude Code Workflows](./workflows.md) — the workflow patterns that use CLAUDE.md
- [Multi-Window Development](./multi-window.md) — all windows share the same CLAUDE.md
- [Custom Instructions for Copilot](../06-copilot-agent-patterns/custom-instructions.md) — the Copilot equivalent
- [Headless and CI Mode](./headless-ci.md) — CLAUDE.md applies in CI too
