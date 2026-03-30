# Edge Cases & Error Handling in Specs

> The requirements that AI agents miss without explicit prompting.

---

## Why Edge Cases Matter More in SDD

AI agents are **optimistic by default**. Given a spec for "user login," an agent will implement the happy path flawlessly — and completely ignore:
- What happens when the database is down
- What happens with concurrent logins from two devices
- What happens when the session token is expired but the refresh token is valid

**If it's not in the spec, the AI won't build it.**

---

## The Edge Case Taxonomy

### Category 1: Input Boundaries

| Edge Case | Example |
|-----------|---------|
| Empty input | User submits blank form |
| Maximum length | User pastes 100,000 characters |
| Minimum values | Quantity of 0 or negative numbers |
| Special characters | `<script>alert('xss')</script>` in name field |
| Unicode | Emoji in usernames, RTL text in descriptions |
| Whitespace | Leading/trailing spaces, tabs, newlines |
| Type coercion | "12" vs 12, "true" vs true |

### Category 2: State Transitions

| Edge Case | Example |
|-----------|---------|
| Double submission | User clicks "Submit" twice quickly |
| Stale data | User edits a record that was updated by someone else |
| Concurrent access | Two users edit the same document simultaneously |
| Mid-process failure | Network drops during multi-step workflow |
| Session expiry | Token expires during long form entry |
| Browser back/forward | User navigates away and returns |

### Category 3: Infrastructure Failures

| Edge Case | Example |
|-----------|---------|
| Database unavailable | Connection pool exhausted |
| External API timeout | Payment provider takes >30s |
| File system full | Upload directory reaches capacity |
| Memory pressure | Large file processing exceeds limits |
| Rate limiting triggered | User exceeds API rate limits |
| DNS resolution failure | External service hostname not resolving |

### Category 4: Data Conditions

| Edge Case | Example |
|-----------|---------|
| Empty collections | Dashboard with zero items |
| Single item | List with exactly one element |
| Large collections | 10,000+ items in a paginated list |
| Deleted references | Notification linking to deleted content |
| Orphaned records | Child records whose parent was deleted |
| Data migration artifacts | Old data format in new system |

### Category 5: User Behavior

| Edge Case | Example |
|-----------|---------|
| Rapid interactions | Clicking 50 times in 1 second |
| Copy-paste exploits | Pasting formatted HTML into plain text field |
| Tab switching | Starting a flow, switching tabs, returning |
| Multiple accounts | User logs into second account in same browser |
| Accessibility tools | Screen reader, keyboard-only navigation |
| Device switching | Starting on mobile, finishing on desktop |

---

## How to Specify Edge Cases

### Template

```markdown
## Edge Cases & Error Handling

### Input Boundaries
| Condition | Expected Behavior | Priority |
|-----------|------------------|----------|
| [condition] | [what the system should do] | High/Medium/Low |

### State Transitions
| Condition | Expected Behavior | Priority |
|-----------|------------------|----------|

### Failure Modes
| Condition | Expected Behavior | Priority |
|-----------|------------------|----------|
```

### Example: File Upload Feature

```markdown
## Edge Cases

### Input Boundaries
| Condition | Expected Behavior |
|-----------|------------------|
| File is 0 bytes | Show "File is empty" error, don't upload |
| File exceeds 10MB | Show "File too large (max 10MB)" before upload starts |
| File type not in allowlist | Show "Only JPEG, PNG, and PDF files are accepted" |
| Filename contains special chars | Sanitize filename, replace with underscores |
| Filename is 255+ characters | Truncate to 200 chars, preserve extension |

### State Transitions
| Condition | Expected Behavior |
|-----------|------------------|
| Upload in progress, user navigates away | Show "Upload in progress" confirmation dialog |
| Upload in progress, network drops | Retry up to 3 times, then show "Upload failed, please retry" |
| User uploads same file twice | Replace existing file, show "File updated" |

### Failure Modes
| Condition | Expected Behavior |
|-----------|------------------|
| Storage service unavailable | Show "Upload temporarily unavailable, try again in a few minutes" |
| Virus scan flags file | Reject upload, show "File could not be uploaded for security reasons" |
```

---

## The "What If" Checklist

Run through these questions for every spec:

```
□ What if the user does nothing? (empty state)
□ What if the user does it twice? (duplicate actions)
□ What if two users do it simultaneously? (concurrency)
□ What if the network drops mid-action? (partial failure)
□ What if the data is invalid? (validation)
□ What if the data is too large? (boundaries)
□ What if the data is malicious? (security)
□ What if a dependency is unavailable? (failure handling)
□ What if the user has no permission? (authorization)
□ What if the user is on a slow device? (performance)
```

---

## Error Response Standards

Define once in the constitution, reference in specs:

```markdown
## Error Response Format (Constitution §5)

All errors return:
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human-readable description",
    "details": [{ "field": "email", "issue": "Invalid format" }]
  }
}

HTTP Status Codes:
- 400: Client error (validation, bad request)
- 401: Authentication required
- 403: Insufficient permissions
- 404: Resource not found
- 409: Conflict (duplicate, stale data)
- 429: Rate limit exceeded
- 500: Server error (log, alert, retry where safe)
```
