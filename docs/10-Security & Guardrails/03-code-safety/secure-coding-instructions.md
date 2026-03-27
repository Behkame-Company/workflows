# Secure Coding Instructions for AI Agents

> Teach AI to write secure code through instruction files — but always remember that instructions are guidance, not enforcement.

---

## The Role of Instructions in Security

Instruction files (`.github/copilot-instructions.md`, `AGENTS.md`, `CLAUDE.md`) are your first line of defense — they tell AI agents *how* you expect code to be written. But they have an important limitation:

> **Instructions are not enforceable.** AI may ignore them, misinterpret them, or fail to apply them in edge cases. This is why you also need automated scanning (defense in depth).

Think of instructions as **training** and static analysis as **testing**:

| Layer | Mechanism | Reliability |
|-------|-----------|-------------|
| Instructions | Tell AI what to do | Medium — AI may not follow |
| Static analysis | Verify what AI did | High — deterministic checks |
| Code review | Human evaluation | Medium — depends on reviewer attention |
| Runtime protection | WAF, CSP, rate limits | High — enforced at infrastructure level |

---

## Complete Security Template for copilot-instructions.md

This template addresses the top AI-generated vulnerabilities:

```markdown
<!-- .github/copilot-instructions.md -->

## Security Requirements

### Database Queries
- ALWAYS use parameterized queries or prepared statements
- NEVER construct SQL using string concatenation or template literals
- Use the ORM query builder when available (e.g., Prisma, SQLAlchemy, GORM)
- Example — correct:
  ```python
  cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
  ```
- Example — incorrect (NEVER do this):
  ```python
  cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
  ```

### Input Validation
- Validate ALL external input at the boundary (API routes, form handlers, CLI args)
- Use schema validation libraries: zod (TypeScript), pydantic (Python), govalidator (Go)
- Reject unexpected fields — do not pass raw request bodies to database operations
- Enforce type, length, and format constraints
- Never trust user input for authorization decisions (roles, permissions, ownership)

### Authentication & Authorization
- Hash passwords with bcrypt (cost factor ≥ 12), scrypt, or argon2
- NEVER store or compare plain-text passwords
- Generate tokens with `crypto.randomBytes(32)` or `secrets.token_urlsafe(32)`
- NEVER use `Math.random()`, `random.randint()`, or similar for security tokens
- Implement rate limiting on authentication endpoints (max 5 attempts/minute)
- Include CSRF protection on all state-changing endpoints
- Validate authorization on every request — do not assume authenticated = authorized

### Secrets Management
- NEVER hardcode API keys, passwords, tokens, or connection strings in source code
- Load secrets from environment variables: `os.environ["KEY"]` / `process.env.KEY`
- Use `.env` files locally (listed in .gitignore) and secret managers in production
- If you need a placeholder in code, use `"PLACEHOLDER_REPLACE_ME"` to make it obvious

### Error Handling
- Log detailed errors server-side (with stack traces, context)
- Return generic error messages to clients: `{ "error": "An internal error occurred" }`
- NEVER expose stack traces, SQL queries, file paths, or internal IPs in API responses
- Use structured logging (JSON format) for server-side error logs

### Shell Commands
- NEVER use `os.system()`, `subprocess.call(shell=True)`, or backtick execution
- Use `subprocess.run()` with a list of arguments: `subprocess.run(["cmd", "arg1", "arg2"])`
- NEVER pass user input into shell commands without strict validation

### HTML and Frontend
- NEVER use `innerHTML`, `document.write()`, or `v-html` with user-provided content
- Use `textContent` for plain text or a sanitization library (DOMPurify) for HTML
- Set Content-Security-Policy headers to prevent inline script execution
- Escape all dynamic content in templates

### Dependencies
- Do not add new dependencies without justification
- Check for known vulnerabilities: `npm audit`, `pip audit`, `govulncheck`
- Prefer well-maintained packages with large user bases
```

---

## Path-Specific Security Instructions

Different code areas have different security needs. Use path-scoped instructions to provide context-specific rules.

### API Routes — `.github/instructions/api.instructions.md`

```markdown
---
applyTo: "src/api/**"
---

## API Route Security

Every API route MUST:
1. Validate request body with a zod/pydantic schema before processing
2. Check authentication via middleware — do not check auth inside the handler
3. Verify authorization — confirm the authenticated user has permission for this action
4. Return appropriate HTTP status codes (401 for unauthenticated, 403 for unauthorized)
5. Rate limit sensitive endpoints (login, password reset, API key generation)

NEVER:
- Access `req.body` properties directly without schema validation
- Trust `req.params.userId` to match the authenticated user
- Return database objects directly — use a DTO/response schema
```

### Database Layer — `.github/instructions/database.instructions.md`

```markdown
---
applyTo: "src/db/**"
---

## Database Security

- ALL queries MUST use parameterized placeholders — never string interpolation
- Use transactions for multi-step operations
- Apply `FOR UPDATE` locks when reading-then-writing to prevent race conditions
- Sanitize LIKE patterns to escape `%` and `_` characters from user input
- Set query timeouts to prevent long-running queries from blocking the database
- Never log full query results — log query metadata only (table, operation, row count)
```

### Frontend — `.github/instructions/frontend.instructions.md`

```markdown
---
applyTo: "src/components/**"
---

## Frontend Security

- NEVER use `dangerouslySetInnerHTML` unless the content is sanitized with DOMPurify
- Avoid inline event handlers — use React's event system
- Do not store tokens or secrets in localStorage — use httpOnly cookies
- Validate all form inputs client-side AND server-side
- Use `rel="noopener noreferrer"` on external links
```

---

## AGENTS.md Security Conventions

For multi-agent workflows that span tools:

```markdown
<!-- AGENTS.md -->

## Security Conventions

All agents working in this repository MUST follow these security rules:

### Non-Negotiable Rules
1. **Parameterized queries only** — no string concatenation in SQL
2. **No hardcoded secrets** — use environment variables exclusively
3. **No os.system() or shell=True** — use subprocess with argument lists
4. **No innerHTML with user data** — use textContent or sanitized HTML
5. **No plain-text passwords** — hash with bcrypt (cost ≥ 12)

### Before Committing
Run the security linter and fix all findings:
```bash
semgrep --config auto --config .semgrep/ai-code-patterns.yml --error .
```

### When Unsure
If you are unsure whether a pattern is secure, choose the more restrictive option.
Do not introduce new authentication, encryption, or authorization patterns —
use the existing patterns in src/auth/ and src/middleware/.
```

---

## CLAUDE.md Security Rules

```markdown
<!-- CLAUDE.md -->

## Security

### Mandatory Checks Before Committing
```bash
semgrep --config .semgrep/ai-code-patterns.yml --error .
npm audit --audit-level=moderate
```

### Security Patterns in This Codebase
- Auth: see src/middleware/auth.ts for JWT validation pattern
- Input validation: see src/schemas/ for zod schemas — every endpoint has one
- Database: all queries go through src/db/queries/ using parameterized knex queries
- Error handling: use src/lib/AppError.ts — never throw raw errors in handlers

### Forbidden Patterns
- eval(), Function(), new Function()
- innerHTML, document.write()
- os.system(), subprocess with shell=True
- String concatenation in SQL queries
- console.log() with sensitive data (tokens, passwords, emails)
```

---

## Instructions That Prevent the Top 5 AI Vulnerabilities

### 1. SQL Injection Prevention

```markdown
## SQL Safety Rule
ALWAYS use parameterized queries. The ORM handles this automatically for standard
operations. For raw queries, use:
- Node.js: `db.raw('SELECT * FROM users WHERE id = ?', [userId])`
- Python: `cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))`
- Go: `db.Query("SELECT * FROM users WHERE id = $1", userID)`

NEVER write: `f"SELECT * FROM users WHERE id = {user_id}"`
```

### 2. XSS Prevention

```markdown
## XSS Safety Rule
NEVER insert user-provided content into the DOM without sanitization.
- Use React's JSX (auto-escapes by default) — never use dangerouslySetInnerHTML
- For vanilla JS: use textContent, not innerHTML
- If HTML rendering is required: sanitize with DOMPurify first
- Set CSP headers: Content-Security-Policy: default-src 'self'; script-src 'self'
```

### 3. CSRF Prevention

```markdown
## CSRF Safety Rule
All state-changing endpoints (POST, PUT, DELETE) must include CSRF protection.
- Use the csrf middleware already configured in src/middleware/csrf.ts
- Include the CSRF token in forms: `<input type="hidden" name="_csrf" value="{{csrfToken}}">`
- For API calls from SPA: include X-CSRF-Token header from the cookie
```

### 4. Hardcoded Secrets Prevention

```markdown
## Secrets Rule
NEVER hardcode secrets, API keys, passwords, or tokens in source code.
- Use `process.env.VARIABLE_NAME` to read secrets
- Reference .env.example for required environment variables
- If a test needs a fake secret, use an obviously fake value: "test-key-not-real"
- The pre-commit hook will block commits containing detected secrets
```

### 5. Insecure Deserialization Prevention

```markdown
## Deserialization Rule
NEVER deserialize untrusted data with:
- Python: pickle.loads(), yaml.load() (use yaml.safe_load())
- Node.js: eval(JSON.parse(...)), or unserialize libraries
- Java: ObjectInputStream on untrusted data

Always validate and schema-check deserialized data before using it.
```

---

## The Limitation You Must Accept

Instructions improve AI output **on average** but cannot **guarantee** secure code:

```
Instruction compliance ≠ 100%

With good instructions:    ~80-90% compliance
Without instructions:      ~40-60% compliance
With instructions + SAST:  ~95-99% catching rate
```

This is why **defense in depth** matters:

1. **Instructions** — Reduce the frequency of vulnerabilities (prevention)
2. **Static analysis** — Catch what instructions missed (detection)
3. **Code review** — Catch logic errors that tools miss (judgment)
4. **Runtime protection** — Stop exploits in production (containment)

---

## Tips

- ✅ Keep security instructions specific and example-driven — "use parameterized queries" with a code example
- ✅ Include both the correct pattern AND the forbidden pattern in instructions
- ✅ Reference existing secure code in your codebase — "follow the pattern in src/auth/"
- ✅ Use path-specific instructions for different security contexts (API vs frontend vs DB)
- ❌ Don't write vague instructions like "write secure code" — be explicit about what that means
- ❌ Don't rely on instructions alone — always pair with automated scanning
- ❌ Don't assume instructions will be followed every time — verify with tests

---

## Next Steps

- 🔗 [Static Analysis for AI-Generated Code](static-analysis.md) — Automated enforcement of security rules
- 🔗 [AI-Generated Code Risks](ai-generated-code-risks.md) — What vulnerabilities to instruct against
- 🔗 [Code Review Practices for AI-Generated Code](code-review-ai-output.md) — Human review layer
- 🔗 [Preventing Secret Leaks](../04-secrets-management/preventing-leaks.md) — Instructions for secrets handling
