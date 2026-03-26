# Domain Guide: Security-Sensitive Code

> Instruction patterns for authentication, authorization, cryptography, and PII handling.

---

## Sample: auth.instructions.md

```yaml
---
applyTo: "src/auth/**"
---
```

```markdown
# Auth Module Instructions

## Authentication
- Passwords: hash with bcrypt, minimum 12 rounds
- JWT access tokens: 15 minute expiration
- Refresh tokens: 7 day expiration, stored in httpOnly secure cookie
- Session validation: check token signature AND expiration on every request
- Login: validate credentials, issue token pair, set refresh cookie

## Authorization
- Role-based access control (RBAC) with roles in `src/auth/roles.ts`
- Check permissions at the route level, not in components
- Deny by default — all routes require auth unless explicitly marked public
- Admin actions: require re-authentication (password confirmation)

## Error Responses
- Failed login: "Invalid credentials" (NEVER specify which field is wrong)
- Failed auth: "Unauthorized" (don't reveal why)
- Rate limited: "Too many attempts. Try again in X minutes"
- NEVER reveal whether a username/email exists in the system

## Rate Limiting
- Login endpoint: 5 attempts per IP per minute
- Password reset: 3 attempts per email per hour
- API endpoints: 100 requests per user per minute
- Use the rate limiter from `src/lib/rate-limit`

## Do Not
- Never log passwords, tokens, session IDs, or PII
- Never store passwords in plain text
- Never send tokens in URL query parameters
- Never disable HTTPS for auth endpoints
- Never trust client-side auth checks alone
- Never use MD5 or SHA for password hashing (use bcrypt)
- Never roll your own crypto — use established libraries
```

---

## Sample: pii.instructions.md

```yaml
---
applyTo: "src/**/{user,profile,account,customer}*.{ts,tsx}"
---
```

```markdown
# PII Handling Instructions

## Data Classification
- PII: email, name, phone, address, date of birth, IP address
- Sensitive PII: SSN, financial data, health data, passwords
- Non-PII: system IDs, product names, timestamps

## Rules
- PII fields in database must be marked in Prisma schema with `/// @pii` comment
- Never log PII (mask or redact: `user@****.com`, `John D.`)
- API responses: only return PII the client needs (select specific fields)
- Search features: never expose PII in autocomplete suggestions
- Exports: PII exports require audit logging
- Deletion: support GDPR right-to-erasure for all PII

## Masking Utility
Use `maskPII()` from `src/lib/privacy` for any logging:
- `maskEmail("user@example.com")` → `u***@example.com`
- `maskPhone("+1234567890")` → `+1***890`
- `maskName("John Doe")` → `J*** D***`
```

---

## Sample: crypto.instructions.md

```yaml
---
applyTo: "src/lib/crypto/**"
---
```

```markdown
# Cryptography Instructions

- Use Node.js built-in `crypto` module
- Random values: `crypto.randomBytes(32)` or `crypto.randomUUID()`
- Encryption: AES-256-GCM with unique IV per encryption
- Hashing (non-password): SHA-256
- Hashing (passwords): bcrypt with 12+ rounds
- Key derivation: PBKDF2 with 100,000+ iterations or Argon2id

## Do Not
- Never use: MD5, SHA-1, DES, RC4 (broken algorithms)
- Never reuse initialization vectors (IVs)
- Never hardcode encryption keys — use environment variables
- Never implement custom encryption algorithms
- Never use `Math.random()` for security-sensitive values
```

---

## Security Review Focus

When creating review-specific instructions for security code:

```markdown
## Security Code Review Priorities
1. Authentication bypass possibilities
2. Authorization escalation paths
3. Injection vulnerabilities (SQL, XSS, command injection)
4. Sensitive data exposure in logs or responses
5. Cryptographic weakness (weak algorithms, hardcoded keys)
6. Race conditions in auth/payment flows
7. Missing rate limiting on sensitive endpoints
```

---

*Next: [DevOps & CI/CD Domain Guide](devops-cicd.md) →*
