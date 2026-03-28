# Review Checklists for AI-Generated Code

> Structured review guides for different types of AI-generated code. Use these checklists to ensure consistent, thorough reviews that catch the issues AI commonly introduces.

---

## General AI Code Review Checklist

Use this for any AI-generated code, regardless of type:

### Correctness
- [ ] **Null handling** — Does the code handle null, undefined, and empty values?
- [ ] **Input validation** — Are all inputs validated before use?
- [ ] **Parameterized queries** — Are database queries using parameters (not string concatenation)?
- [ ] **Error handling** — Are errors caught, logged, and handled without leaking internal details?
- [ ] **Race conditions** — Could concurrent execution cause issues?
- [ ] **Boundary values** — Does the code handle min, max, zero, negative values correctly?

### Security
- [ ] **No hardcoded secrets** — No API keys, passwords, or tokens in code
- [ ] **Authentication** — Is auth required where expected?
- [ ] **Authorization** — Does the user have permission for this action?
- [ ] **Output sanitization** — Is output escaped to prevent XSS?
- [ ] **Dependency safety** — Are all imports real packages (not hallucinated)?

### Project Patterns
- [ ] **Matches existing code** — Does it follow the patterns in neighboring files?
- [ ] **Uses project utilities** — Does it use existing helpers instead of reimplementing?
- [ ] **Follows naming conventions** — Do names match the instruction file rules?
- [ ] **Error pattern** — Does it use the team's error handling approach?

### Testing
- [ ] **Tests exist** — Are there tests for the new code?
- [ ] **Tests are meaningful** — Do tests verify behavior, not just exercise code?
- [ ] **Edge cases tested** — Null, empty, boundary, error paths?
- [ ] **Tests pass** — Do all tests pass in CI?

---

## API Endpoint Checklist

For AI-generated REST API endpoints, route handlers, or controllers:

### Authentication and Authorization
- [ ] Endpoint requires authentication (unless explicitly public)
- [ ] Authorization checks verify the user can perform this action
- [ ] Token validation is complete (expiry, issuer, audience)
- [ ] No Insecure Direct Object Reference (IDOR) — users can only access their own resources

### Input Validation
- [ ] Request body validated against a schema (e.g., zod)
- [ ] URL parameters validated (type, format, range)
- [ ] Query parameters validated with defaults
- [ ] File uploads validated (type, size, content)
- [ ] No unexpected fields accepted (strict schema)

### Response
- [ ] Correct HTTP status codes (201 for creation, 204 for deletion, etc.)
- [ ] Error responses follow consistent format (e.g., RFC 7807)
- [ ] No internal details leaked in error responses
- [ ] Sensitive data filtered from responses
- [ ] Pagination for list endpoints

### Performance and Safety
- [ ] Rate limiting considered
- [ ] Request size limits set
- [ ] Timeout handling for external calls
- [ ] Database queries are efficient (indexes, no N+1)
- [ ] Large result sets are paginated

### Example Review

```typescript
// AI generated this endpoint — review findings:

app.get('/api/users/:id/documents', async (req, res) => {
  // ❌ MISSING: Authorization — any user can access any user's documents
  // ❌ MISSING: Pagination — could return thousands of documents
  // ⚠️ CHECK: Is req.params.id validated as UUID?
  
  const documents = await documentRepo.findByUserId(req.params.id);
  
  // ❌ MISSING: Null check — what if user doesn't exist?
  // ❌ ISSUE: Returns all document fields — may include internal metadata
  
  res.json(documents);
  // ❌ MISSING: Consistent response wrapper
});
```

---

## UI Component Checklist

For AI-generated React, Vue, Angular, or other frontend components:

### Accessibility
- [ ] Interactive elements have accessible labels (`aria-label`, `aria-labelledby`)
- [ ] Color is not the only way to convey information
- [ ] Keyboard navigation works (tab order, focus management)
- [ ] Screen reader text for icons and images (`alt` text)
- [ ] Focus is managed on modals and dynamic content

### State Management
- [ ] Loading state handled (skeleton, spinner, placeholder)
- [ ] Error state handled (error message, retry option)
- [ ] Empty state handled (no data, first use)
- [ ] Optimistic updates are correct (rollback on failure)
- [ ] No unnecessary re-renders (memoization where appropriate)

### Error Handling
- [ ] API call failures show user-friendly messages
- [ ] Network errors are caught and displayed
- [ ] Form validation errors are clear and accessible
- [ ] Error boundaries prevent full app crashes

### Data and Props
- [ ] Props are fully typed
- [ ] Required vs. optional props are correct
- [ ] Default values are reasonable
- [ ] No props drilling (use context or state management for deep passing)

### Example Review

```tsx
// AI generated this component — review findings:

function UserProfile({ userId }) {
  // ❌ MISSING: TypeScript types for props
  // ❌ MISSING: Loading state
  // ❌ MISSING: Error state
  
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(r => r.json())
      .then(setUser);
    // ❌ MISSING: Error handling in fetch
    // ❌ MISSING: Cleanup on unmount (race condition)
    // ❌ MISSING: userId in dependency array is correct ✅
  }, [userId]);
  
  return (
    <div>
      {/* ❌ MISSING: Null check on user */}
      <h1>{user.name}</h1>
      <img src={user.avatar} />
      {/* ❌ MISSING: alt text for accessibility */}
    </div>
  );
}
```

---

## Database Checklist

For AI-generated database migrations, models, queries, or data access code:

### Migrations
- [ ] Migration has both `up` and `down` (rollback) functions
- [ ] Column types are appropriate (don't use `text` for everything)
- [ ] NOT NULL constraints applied where appropriate
- [ ] Default values set where needed
- [ ] Indexes created for frequently queried columns
- [ ] Foreign key constraints defined
- [ ] Migration is idempotent (safe to run multiple times)

### Queries
- [ ] All queries use parameterized statements (never string concatenation)
- [ ] SELECT queries specify columns (never `SELECT *` in production)
- [ ] JOINs are correct (LEFT vs INNER vs appropriate type)
- [ ] WHERE clauses use indexed columns
- [ ] Pagination is implemented for large result sets
- [ ] Transactions used for multi-table operations

### Data Integrity
- [ ] Unique constraints where needed
- [ ] Cascading deletes are intentional (not accidental data loss)
- [ ] Soft delete vs. hard delete matches team convention
- [ ] Timestamps (created_at, updated_at) are included
- [ ] Audit fields included where required by compliance

### Example Review

```sql
-- AI generated this migration — review findings:

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER,         -- ❌ MISSING: NOT NULL, FOREIGN KEY
  product_name TEXT,        -- ⚠️ Should this be a product_id reference?
  amount FLOAT,             -- ❌ ISSUE: Use DECIMAL for money, not FLOAT
  status TEXT,              -- ⚠️ Consider ENUM or CHECK constraint
  created_at TIMESTAMP      -- ✅ Good
  -- ❌ MISSING: updated_at
  -- ❌ MISSING: indexes on user_id, status
);

-- ❌ MISSING: Down/rollback migration
```

---

## Severity Tiers

When logging review findings, categorize by severity:

### 🔴 Critical (Must fix before merge)
- Security vulnerabilities (injection, auth bypass, data exposure)
- Data loss potential (cascading deletes, missing transactions)
- Breaking changes to public APIs
- Missing authentication or authorization

### 🟡 Major (Should fix before merge)
- Logic errors in business rules
- Missing error handling on critical paths
- Missing input validation
- Performance issues (N+1 queries, unbounded results)
- Accessibility violations (no keyboard navigation, missing labels)

### 🔵 Minor (Can fix in follow-up)
- Non-critical style inconsistencies
- Missing documentation
- Suboptimal but correct code
- Test coverage below threshold but not at zero

---

## Copy-Paste Checklists

### Quick Review (< 50 lines of AI code)
```
□ Inputs validated    □ Nulls handled     □ Errors caught
□ No secrets          □ Tests exist       □ Patterns match
```

### Standard Review (50-200 lines)
```
□ Inputs validated    □ Nulls handled     □ Errors caught
□ No secrets          □ Tests exist       □ Patterns match
□ Auth correct        □ Edge cases        □ Dependencies real
□ No IDOR            □ Performance OK     □ Docs updated
```

### Deep Review (> 200 lines or security-sensitive)
```
□ Full general checklist    □ Type-specific checklist
□ Security threat model     □ Integration points verified
□ Performance profiled      □ Rollback plan exists
```

## Next Steps

- [Return to the full review workflow →](ai-code-review-workflow.md)
- [Understand the human-AI review balance →](human-ai-review-balance.md)
- [Use PR templates for disclosure →](pr-templates.md)
- [Plan your AI tool rollout →](../05-scaling-ai-across-teams/rollout-strategy.md)
