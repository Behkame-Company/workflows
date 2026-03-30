# Writing Specifications for AI Agents

> How to write specs that AI coding agents follow accurately and completely.

---

## The AI Interpretation Problem

AI agents interpret specs differently than humans. Understanding these differences is critical to writing effective specs.

### How Humans Read Specs
- Fill in gaps with domain knowledge and experience
- Infer intent from context and tone
- Ask clarifying questions when unsure
- Apply judgment to unstated edge cases

### How AI Agents Read Specs
- Take instructions literally
- Implement exactly what's written (and nothing more)
- Miss implicit requirements
- Make "reasonable" but potentially wrong assumptions for gaps
- Apply general patterns that may not fit your specific context

**Key Insight**: Write specs for AI as if writing for a very literal, very fast junior developer who has never seen your codebase.

---

## The 5 Rules of AI-Readable Specs

### Rule 1: Be Explicit, Not Implicit

```markdown
❌  Implicit:
"Users can manage their profile"

✅  Explicit:
"Users can:
- View their profile (name, email, avatar, bio)
- Edit name, avatar, and bio (email requires re-verification)
- Delete their account (with 30-day grace period)
- Download their data (GDPR export in JSON format)"
```

### Rule 2: Specify Exact Values

```markdown
❌  Vague:
"The system should respond quickly"
"Show a reasonable number of items"
"Use appropriate error messages"

✅  Specific:
"API responses return within 200ms at p95"
"Show 20 items per page, with pagination"
"Show 'Unable to save. Please try again.' on save failure"
```

### Rule 3: Define Boundaries

```markdown
❌  Unbounded:
"Users can upload files"

✅  Bounded:
"Users can upload files:
- Max size: 10MB
- Allowed types: .jpg, .png, .pdf, .docx
- Max files per upload: 10
- Filename max length: 255 characters
- Storage: S3 bucket defined in env vars"
```

### Rule 4: State the Negative

Tell the AI what NOT to do:

```markdown
## Constraints
- Do NOT implement social login in this version
- Do NOT store passwords in plaintext
- Do NOT expose internal error details to users
- Do NOT create new database tables — use existing User table
- Notification content must NOT contain user PII in logs
```

### Rule 5: Provide Context About the Codebase

```markdown
## Implementation Context
- This project uses Next.js 14 app router (not pages router)
- Database access is through Prisma ORM (see prisma/schema.prisma)
- Auth is handled by NextAuth (see src/lib/auth.ts)
- Existing notification-like pattern: see src/features/alerts/ for reference
- All API routes are in src/app/api/ following REST conventions
```

---

## Spec Patterns That AI Agents Love

### Pattern 1: Decision Tables

AI agents handle decision tables exceptionally well:

```markdown
## Notification Routing Rules

| Event Type | In-App | Email | Push | Conditions |
|-----------|--------|-------|------|------------|
| New message | Always | If offline >5min | If mobile app installed | |
| Mention | Always | Always | Always | |
| System alert | Always | Never | Never | |
| Weekly digest | Never | Every Monday 9am | Never | If subscribed |
```

### Pattern 2: State Machines

```markdown
## Order Status Flow

[created] ──► [confirmed] ──► [processing] ──► [shipped] ──► [delivered]
     │              │                                             │
     ▼              ▼                                             ▼
 [cancelled]   [cancelled]                                   [returned]

Transitions:
- created → confirmed: When payment succeeds
- created → cancelled: When user cancels within 1 hour
- confirmed → cancelled: When admin cancels (full refund)
- confirmed → processing: When warehouse begins fulfillment
- shipped → delivered: When carrier confirms delivery
- delivered → returned: When user initiates return within 30 days
```

### Pattern 3: Data Contracts

```markdown
## Notification Object Schema

```typescript
interface Notification {
  id: string;           // UUID v4
  userId: string;       // References User.id
  type: 'info' | 'warning' | 'error' | 'success';
  title: string;        // Max 100 chars
  body: string;         // Max 500 chars, supports markdown
  link?: string;        // Optional deep link URL
  read: boolean;        // Default: false
  createdAt: Date;      // ISO 8601
  expiresAt?: Date;     // Auto-delete after this date
}
```

### Pattern 4: Behavioral Examples

```markdown
## Search Behavior

Example 1: "john smith"
→ Searches first_name AND last_name fields
→ Results: exact matches first, then partial matches
→ Case-insensitive

Example 2: "john@email.com"
→ Searches email field (detected by @ symbol)
→ Results: exact match only

Example 3: ""  (empty string)
→ Returns all results (no filter applied)
→ Paginated, 20 per page
```

---

## Anti-Patterns for AI Specs

### 1. Assuming Context
```markdown
❌  "Follow the existing pattern"
✅  "Follow the pattern in src/features/auth/AuthService.ts — 
     service class with constructor injection, async methods, error wrapping"
```

### 2. Open-Ended Requirements
```markdown
❌  "Make it user-friendly"
✅  "Form fields have labels, placeholder text, and inline validation messages.
     Tab order follows visual order. Submit button is disabled until form is valid."
```

### 3. Ambiguous Pronouns
```markdown
❌  "When it fails, show them the error"
✅  "When the API request fails, show the user an error toast with the message"
```
