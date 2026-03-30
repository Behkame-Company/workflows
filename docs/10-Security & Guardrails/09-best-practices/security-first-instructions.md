# Security-First AI Instructions

> Writing custom instruction files that make security the default — not an afterthought — for every AI coding tool in your workflow.

---

## Why Instructions Matter for Security

AI coding assistants follow the patterns and rules you establish in instruction files. Without explicit security guidance, AI defaults to "make it work" — which often means insecure defaults, hardcoded values, and missing validation.

Security-first instructions shift the baseline: every code suggestion starts from a secure foundation.

```
Without security instructions:
  AI suggests → code that works → developer reviews for security (maybe)

With security instructions:
  AI suggests → code that works AND follows security rules → developer verifies compliance
```

> ⚠️ **Critical Limitation**: Instruction-based security is **advisory, not enforceable**. AI models follow instructions probabilistically — they will sometimes ignore or misinterpret rules. Always pair instruction files with automated scanning (SAST, secret detection, linting).

---

## copilot-instructions.md Security Template

The `.github/copilot-instructions.md` file applies to all GitHub Copilot interactions across your repository.

### Complete Security-Focused Template

```markdown
# Copilot Instructions — Security Policy

## Security Rules (MANDATORY)

These rules apply to ALL code generation in this repository.

### Secrets & Credentials
- **Never hardcode secrets**, API keys, passwords, or tokens in source code
- Always read secrets from environment variables or a secrets manager
- Use placeholder values like `process.env.DATABASE_URL` — never example passwords
- Never commit .env files — reference .env.example with dummy values only

### Input Validation
- **Validate all user input** before processing
- Use allowlists over denylists for input validation
- Sanitize inputs for the specific context (HTML, SQL, shell, etc.)
- Validate on the server side — never trust client-side validation alone

### Database Security
- **Always use parameterized queries** — never string concatenation for SQL
- Use ORM methods that parameterize by default
- Apply least-privilege database users per service
- Never expose raw database errors to users

### Authentication & Authorization
- **Use CSRF tokens** for all state-changing operations
- Implement proper session management with secure cookie flags
- Check authorization on every request — not just at the UI level
- Use bcrypt/argon2 for password hashing — never MD5 or SHA for passwords

### Network Security
- **Use HTTPS for all external calls** — never plain HTTP
- Validate TLS certificates — never disable certificate verification
- Set appropriate CORS policies — never use `*` in production
- Implement rate limiting on all public endpoints

### Dangerous Functions
- **Never use eval() or exec() with user input**
- Never use os.system() or subprocess with shell=True and user input
- Never use innerHTML with unsanitized data
- Never deserialize untrusted data without validation

### Error Handling
- Never expose stack traces or internal details in production errors
- Log errors with context but without sensitive data
- Use generic error messages for users — detailed logs for developers

### Dependencies
- Prefer well-maintained packages with active security response
- Never suggest deprecated or unmaintained packages
- Check for known vulnerabilities before suggesting a dependency

## Code Patterns

### Required Patterns
When generating code, always use these patterns:

**SQL (any language):**
```sql
-- ✅ Always: parameterized queries
SELECT * FROM users WHERE id = ?

-- ❌ Never: string concatenation
SELECT * FROM users WHERE id = ' + userId + '
```

**Environment variables for config:**
```javascript
// ✅ Always
const apiKey = process.env.API_KEY;

// ❌ Never
const apiKey = "sk-abc123...";
```

**Input validation:**
```python
# ✅ Always
from pydantic import BaseModel, validator
class UserInput(BaseModel):
    email: str
    age: int = Field(ge=0, le=150)

# ❌ Never
email = request.form['email']  # unvalidated
```
```

---

## Path-Specific Security Rules

Different parts of your codebase need different security attention levels. Use path-specific instructions (supported via VS Code settings or instruction file conventions) to enforce stricter rules where they matter most.

### API Routes — Maximum Strictness

```markdown
<!-- .github/copilot-instructions.md — API section -->

## API Route Rules (src/api/**, routes/**)
- Every endpoint MUST validate input using the schema validator
- Every endpoint MUST check authentication before processing
- Every endpoint MUST check authorization for the specific resource
- Every response MUST sanitize output — no raw database objects
- Every error MUST use the standard error response format
- Rate limiting MUST be applied to all public endpoints
- CORS MUST be configured per-route — never wildcard
```

### Authentication Module — Zero Tolerance

```markdown
## Auth Module Rules (src/auth/**, lib/auth/**)
- NEVER log passwords, tokens, or session IDs — even at debug level
- NEVER compare secrets with == — use constant-time comparison
- NEVER store plaintext passwords — always hash with bcrypt/argon2
- NEVER generate tokens with Math.random() — use crypto-secure RNG
- Session tokens MUST have expiration
- Failed login attempts MUST be rate-limited
- Password reset tokens MUST be single-use and time-limited
```

### Database Layer — Injection Prevention

```markdown
## Database Rules (src/db/**, models/**, repositories/**)
- ALL queries MUST use parameterized statements or ORM methods
- NEVER construct SQL by string concatenation or template literals
- NEVER pass raw user input to query functions
- Database errors MUST be caught and logged — never exposed to callers
- Migration files MUST be reviewed for destructive operations
- Connection strings MUST come from environment variables
```

---

## AGENTS.md Security Conventions

For GitHub Copilot coding agent (and compatible agents), `AGENTS.md` defines how autonomous agents interact with your codebase.

### Security Section Template

```markdown
# AGENTS.md

## Security Conventions

### Before Writing Code
1. Check if the task involves user input — if yes, add validation
2. Check if the task involves data storage — if yes, use parameterized queries
3. Check if the task involves authentication — if yes, follow auth module patterns
4. Check if the task involves external calls — if yes, use HTTPS with timeout

### Forbidden Actions
- Do NOT add new dependencies without checking for known vulnerabilities
- Do NOT modify authentication or authorization logic without explicit approval
- Do NOT disable security features (CSRF, CORS, CSP) even temporarily
- Do NOT add endpoints without authentication middleware
- Do NOT store sensitive data in logs, comments, or client-side storage

### Required Testing
- Every new endpoint must have at least one test for unauthorized access
- Every input validation must have tests for malicious input
- Every authentication change must have tests for bypass attempts

### Code Review Requirements
- Security-related changes require review from a security-aware team member
- Changes to auth, permissions, or crypto MUST NOT be auto-merged
- All new dependencies must be justified in the PR description
```

---

## CLAUDE.md Security Section

For Claude Code (Anthropic's coding agent), security rules go in `CLAUDE.md` at the project root.

### Security Template

```markdown
# CLAUDE.md

## Security Requirements

You MUST follow these security rules for all code in this project:

### Hard Rules (Never Break)
- Never hardcode secrets, API keys, or credentials
- Never use string concatenation for SQL queries
- Never use eval(), exec(), or Function() with dynamic input
- Never disable TLS certificate verification
- Never use MD5 or SHA1 for password hashing
- Never expose stack traces in production error responses
- Never commit .env files or files matching *.pem, *.key

### Soft Rules (Follow Unless Explicitly Overridden)
- Prefer allowlists over denylists for input validation
- Prefer ORM methods over raw SQL
- Prefer established auth libraries over custom implementations
- Prefer environment variables over config files for secrets
- Add rate limiting to public-facing endpoints

### When Uncertain About Security
- If you're unsure whether something is secure, choose the more restrictive option
- If a task requires elevated privileges, ask before proceeding
- If you encounter hardcoded credentials in existing code, flag them — don't replicate the pattern

### File Handling
- Never read or write to paths outside the project directory
- Validate file types before processing uploads
- Use random filenames for user-uploaded content
- Set appropriate file permissions (no world-readable for sensitive files)
```

---

## Key Rules Every Instruction File Should Include

Regardless of format or tool, these rules belong in every AI instruction file:

| Rule | Why |
|------|-----|
| Never hardcode secrets | AI will cheerfully generate `password = "admin123"` |
| Always use parameterized queries | AI defaults to string concatenation in many languages |
| Validate all user input | AI often skips validation for brevity |
| Use HTTPS for all external calls | AI sometimes generates HTTP URLs in examples |
| Never use eval() with user input | AI may suggest eval for "flexibility" |
| Never disable certificate verification | AI adds `verify=False` to "fix" SSL errors |
| Hash passwords with bcrypt/argon2 | AI sometimes suggests MD5 or SHA for passwords |

---

## Tips

✅ **Do** include both the rule AND an example of the correct pattern
✅ **Do** explain *why* each rule exists — AI follows rules better with reasoning
✅ **Do** keep rules specific: "Use parameterized queries" > "Write secure code"
✅ **Do** update instruction files when new vulnerability patterns are discovered
✅ **Do** pair every instruction file with automated scanning to catch misses

❌ **Don't** rely on instruction files as your only security control
❌ **Don't** assume the AI will always follow every rule perfectly
❌ **Don't** write vague rules like "be secure" — AI needs specific patterns
❌ **Don't** forget to test that AI actually follows your instructions
❌ **Don't** copy security rules without understanding them — they need to match your stack
