# Phase 1: Specify

> Capturing what to build, why it matters, and what success looks like.

---

## Purpose

The Specify phase answers three questions:
1. **What** are we building?
2. **Why** does it matter?
3. **How** do we know it's done?

Everything else — architecture, technology, implementation details — comes later.

---

## The Specify Workflow

```
1. Product Vision / Feature Request
   │
2. Draft Specification
   │
3. Clarification Round (ask questions)
   │
4. Refine Specification
   │
5. Stakeholder Review
   │
6. Approved spec.md ──► Phase 2: Plan
```

---

## Spec Structure Template

```markdown
# Feature: [Feature Name]

## Summary
One paragraph describing the feature and its purpose.

## User Stories
- As a [user type], I want [goal], so that [benefit]
- As a [user type], I want [goal], so that [benefit]

## Acceptance Criteria
- [ ] Criterion 1: Specific, measurable condition
- [ ] Criterion 2: Specific, measurable condition
- [ ] Criterion 3: Specific, measurable condition

## Edge Cases
- What happens when [unusual condition]?
- What happens when [error condition]?
- What happens when [boundary condition]?

## Non-Functional Requirements
- Performance: [specific metrics]
- Security: [specific requirements]
- Accessibility: [specific standards]

## Out of Scope
- [Thing that might be confused as in-scope]
- [Feature that will be done later]

## Dependencies
- Depends on [existing feature/service]
- Requires [infrastructure/API]
```

---

## The Clarification Step

The most valuable part of Specify is the **clarification round** — systematically identifying and resolving ambiguity.

### Questions to Always Ask

**Users & Personas**
- Who are the primary users? Secondary users? Admin users?
- What permissions/roles affect this feature?

**Data & State**
- What data does this feature create, read, update, or delete?
- What happens to existing data?
- Are there data validation rules?

**Error Scenarios**
- What happens when the network is unavailable?
- What happens when the user provides invalid input?
- What happens when a dependent service is down?

**Boundaries**
- What are the limits (max items, max size, rate limits)?
- What is explicitly NOT included?
- When does this feature interact with other features?

**Business Rules**
- Are there time-based rules (expiration, scheduling)?
- Are there user-based rules (quotas, tiers, permissions)?
- Are there region/locale-specific behaviors?

---

## Acceptance Criteria Best Practices

### Format: Given-When-Then

```markdown
Given [precondition]
When [action]
Then [expected result]
```

### Examples

```markdown
## Acceptance Criteria

### Registration
- Given I am on the registration page
  When I submit valid email and password (8+ chars, 1 number)
  Then my account is created and I receive a verification email

- Given I submit an email that already exists
  When I click Register
  Then I see "This email is already registered" error

- Given I submit a password with fewer than 8 characters
  When I click Register
  Then I see password requirement hints
```

### Rules for Good Acceptance Criteria

| ✅ Do | ❌ Don't |
|-------|---------|
| Use specific, measurable conditions | Use vague words ("fast", "nice", "intuitive") |
| Include both success and failure paths | Only describe the happy path |
| One behavior per criterion | Bundle multiple behaviors |
| Use concrete values where possible | Leave values ambiguous ("reasonable time") |
| Make each criterion independently testable | Create criteria that depend on each other |

---

## Common Specify Mistakes

### 1. Solution Masquerading as Specification
```markdown
❌  "Use a PostgreSQL JSONB column to store user preferences"
✅  "Users can set and retrieve personal preferences (theme, language, notifications)"
```

### 2. Missing the "Why"
```markdown
❌  "Add a CSV export button to the dashboard"
✅  "Users need to export dashboard data for offline analysis and reporting.
     Export should include all visible columns in the current filter state."
```

### 3. Implicit Assumptions
```markdown
❌  "User can upload a profile photo"
✅  "User can upload a profile photo (JPEG/PNG, max 5MB, min 100x100px).
     Photos are cropped to square. Existing photo is replaced."
```

### 4. No Scope Boundaries
Without "Out of Scope," specs grow unbounded:
```markdown
## Out of Scope (v1)
- Social login (OAuth) — planned for v2
- Profile photo editing/cropping — upload only
- Admin ability to change other users' photos
```

---

## AI-Assisted Specification

You can use AI to help draft specs, but always review:

```
Prompt:  "Help me write a specification for a user notification system.
          Ask me clarifying questions before writing the spec."

AI:      "I have some questions:
          1. What types of notifications? (email, push, in-app, SMS?)
          2. Can users configure notification preferences?
          3. Are there notification groups or categories?
          ..."

You:     [Answer questions]

AI:      [Generates structured spec draft]

You:     [Review, refine, approve]
```
