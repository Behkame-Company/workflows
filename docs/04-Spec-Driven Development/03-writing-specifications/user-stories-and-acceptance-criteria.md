# User Stories & Acceptance Criteria

> Writing testable requirements that both humans and AI agents can verify.

---

## User Stories

### The Standard Format

```
As a [role/persona],
I want [capability/goal],
So that [benefit/value].
```

### Why This Format Works for AI

Each component gives the AI critical context:
- **Role**: Who is the user? What permissions/context do they have?
- **Goal**: What specific functionality is needed?
- **Benefit**: Why does this matter? (helps AI make judgment calls)

### Examples by Quality

```markdown
❌  Terrible:   "User login"
❌  Bad:        "As a user, I want to log in"
⚠️  Mediocre:  "As a user, I want to log in so I can use the app"
✅  Good:       "As a registered user, I want to log in with email/password
                 so that I can access my personalized dashboard and settings"
✅  Excellent:  "As a registered user, I want to log in with email and password
                 (or social OAuth) so that I can access my personalized dashboard.
                 Sessions persist for 30 days unless explicitly logged out."
```

---

## Acceptance Criteria

### Format Options

#### Given/When/Then (Gherkin)
Best for behavior-focused criteria:
```markdown
Given I am on the login page
When I enter valid credentials and click "Sign in"
Then I am redirected to my dashboard
And my session is active for 30 days
```

#### Checklist Format
Best for verification-focused criteria:
```markdown
- [ ] Login form accepts email and password
- [ ] "Remember me" checkbox persists session for 30 days
- [ ] Invalid credentials show "Invalid email or password" error
- [ ] Account locked after 5 failed attempts for 15 minutes
- [ ] Social login (Google, GitHub) redirects and creates session
```

#### Rule-Based Format
Best for business logic:
```markdown
Rule: Session Duration
- Default session: 24 hours
- "Remember me" session: 30 days
- Session refreshed on activity within last hour
- Explicit logout invalidates session immediately
```

### When to Use Which

| Format | Best For | Audience |
|--------|----------|----------|
| Given/When/Then | User-facing behavior | QA, Product, BDD |
| Checklist | Quick verification | Developers, code review |
| Rule-Based | Complex business logic | Developers, architects |

---

## Writing Effective Acceptance Criteria

### The SMART-AC Framework

| Letter | Meaning | Example |
|--------|---------|---------|
| **S**pecific | One behavior per criterion | "Login succeeds with valid email format" |
| **M**easurable | Numeric or binary outcome | "Response time <200ms", "Error message shown" |
| **A**chievable | Technically feasible | Not "works offline with full DB sync" |
| **R**elevant | Tied to user story value | Not "button is blue" (unless it's a design spec) |
| **T**estable | Can write a test for it | "Given/When/Then can be automated" |

### Common Acceptance Criteria Mistakes

#### 1. Too Vague
```markdown
❌  "System should be responsive"
✅  "Login page renders correctly on viewports 320px to 1920px wide"
```

#### 2. Implementation Prescriptive
```markdown
❌  "Use bcrypt with 12 salt rounds to hash passwords"
✅  "Passwords are stored using industry-standard hashing (never plaintext)"
```

#### 3. Missing Negative Cases
```markdown
❌  Only: "User can log in with valid credentials"
✅  Also: "Invalid credentials show error without revealing which field is wrong"
✅  Also: "Account locks after 5 failed attempts within 15 minutes"
```

#### 4. Compound Criteria
```markdown
❌  "User can log in, see their dashboard, and update their profile"
✅  Separate into three criteria — one behavior each
```

---

## AI Agent Considerations

### What AI Agents Need from Acceptance Criteria

AI agents interpret acceptance criteria literally. Provide:

1. **Concrete values**: "Max 5 items" not "a reasonable number"
2. **Explicit behavior**: "Show error toast for 5 seconds" not "handle gracefully"
3. **Boundary conditions**: "If input > 1000 chars, truncate and show warning"
4. **Error messages**: Exact text or pattern: "Invalid email format" (not just "show error")

### Example: AI-Optimized Acceptance Criteria

```markdown
## Feature: User Registration

### Success Path
- AC-1: Given valid email (RFC 5322 format) and password (8+ chars, 1+ number, 1+ special)
  When form is submitted
  Then user record is created with status=pending
  And verification email sent within 60 seconds to provided address
  And success message "Check your email to verify your account" is displayed

### Failure Paths
- AC-2: Given email already registered
  When form is submitted
  Then error "An account with this email already exists" is shown
  And no duplicate user record is created

- AC-3: Given password doesn't meet requirements
  When password field loses focus
  Then show inline validation with specific unmet requirements
  (e.g., "Must include at least one number")

### Boundary Conditions
- AC-4: Given email field is empty
  When form is submitted
  Then "Email is required" validation error is shown
  And form is not submitted

- AC-5: Given 5 registration attempts from same IP within 1 hour
  When 6th attempt is made
  Then show "Too many attempts. Please try again in 1 hour"
```

---

## User Story Mapping for SDD

Organize stories into a map that shows the full user journey:

```
User Journey: New User Onboarding
─────────────────────────────────
Registration → Verification → First Login → Onboarding → Dashboard

Story 1: Register    Story 3: Verify      Story 5: Log in
Story 2: Validation  Story 4: Resend link  Story 6: Onboarding wizard
                                            Story 7: Dashboard tour
```

This helps identify gaps — stories that are needed but not yet written.

---

*Next: [Edge Cases & Error Handling →](edge-cases-and-errors.md)*
