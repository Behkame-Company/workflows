# Anatomy of a Spec

> The structure, sections, and essential components of a well-written specification.

---

## The Spec at a Glance

A complete specification has **7 core sections** and **3 optional sections**:

```
spec.md
├── Header (name, version, status)
├── Summary                          ← What and why (1 paragraph)
├── User Stories                     ← Who wants what
├── Acceptance Criteria              ← How to verify success
├── Edge Cases & Error Handling      ← What can go wrong
├── Non-Functional Requirements      ← Performance, security, a11y
├── Scope Boundaries                 ← What's in AND what's out
├── Dependencies (optional)          ← What this relies on
├── Open Questions (optional)        ← Unresolved decisions
└── Glossary (optional)              ← Domain-specific terms
```

---

## Section-by-Section Breakdown

### 1. Header

```markdown
# Feature: User Notification System

| Field | Value |
|-------|-------|
| Version | 1.0 |
| Status | Draft → In Review → Approved |
| Author | @jane |
| Reviewers | @bob, @alice |
| Created | 2026-01-15 |
| Last Updated | 2026-01-20 |
| Spec Kit Path | .specify/specs/notifications/spec.md |
```

**Why**: Metadata enables tracking, accountability, and version history.

### 2. Summary

```markdown
## Summary

Users currently have no way to know when important events occur in the
application. This feature adds a real-time notification system that delivers
in-app, email, and push notifications based on user preferences. The goal is
to increase user engagement by 20% and reduce support tickets about missed
events by 50%.
```

**Rules**:
- One paragraph (3–5 sentences)
- Covers *what* it is and *why* it matters
- Includes success metric if possible
- A busy executive should understand the feature from this alone

### 3. User Stories

```markdown
## User Stories

### US-1: Receive in-app notifications
As a logged-in user, I want to receive real-time in-app notifications
so that I'm aware of important events without leaving the application.

### US-2: Configure notification preferences
As a user, I want to choose which types of notifications I receive
and through which channels so that I'm not overwhelmed by irrelevant alerts.

### US-3: View notification history
As a user, I want to view my past notifications in a notification center
so that I can catch up on events I may have missed.
```

**Rules**:
- Use standard format: "As a [role], I want [goal], so that [benefit]"
- Each story should be independently valuable
- Number stories for easy reference (US-1, US-2, etc.)

### 4. Acceptance Criteria

```markdown
## Acceptance Criteria

### US-1: Receive in-app notifications
- AC-1.1: Given a triggering event occurs
  When the user is logged in
  Then a notification appears within 3 seconds
- AC-1.2: Given multiple unread notifications exist
  When the user views the notification bell
  Then the unread count badge shows the correct number
- AC-1.3: Given the user clicks a notification
  When it links to a specific page
  Then the user is navigated to that page and the notification is marked read
```

**Rules**:
- Given/When/Then format for clarity
- Number criteria to match their user story (AC-1.1, AC-1.2)
- Each criterion is independently testable
- Include both success and failure paths

### 5. Edge Cases & Error Handling

```markdown
## Edge Cases & Error Handling

| Scenario | Expected Behavior |
|----------|------------------|
| User receives 100+ notifications at once | Show latest 50, "Load more" button |
| WebSocket connection drops | Fall back to polling every 30s, show connection indicator |
| Notification references deleted content | Show "This content is no longer available" |
| User disables all notification channels | Show warning: "You won't receive any notifications" |
| Database write fails during notification creation | Retry 3 times with exponential backoff, log failure |
| Notification payload exceeds size limit | Truncate body to 500 chars, link to full content |
```

**Rules**:
- At least 3 edge cases per spec (more for complex features)
- Include: data boundaries, failure modes, concurrent access, empty/null states
- Define expected behavior, not just the scenario

### 6. Non-Functional Requirements

```markdown
## Non-Functional Requirements

### Performance
- Notification delivery latency: <3s for in-app, <60s for email
- Notification center loads <500ms with 1000+ notifications
- System supports 10,000 concurrent WebSocket connections

### Security
- Notifications only visible to intended recipient
- Notification content sanitized (no XSS via notification body)
- API endpoints require authentication

### Accessibility
- Notification toast announcements via ARIA live regions
- Notification center navigable by keyboard
- Color-coded indicators have text alternatives
```

### 7. Scope Boundaries

```markdown
## Scope

### In Scope
- In-app real-time notifications
- Email notifications
- User preference settings
- Notification history (last 90 days)

### Out of Scope (v1)
- Push notifications (mobile) — planned for v2
- Notification scheduling — planned for v2
- Custom notification sounds
- Notification grouping/threading
- Admin broadcast notifications
```

**Why this matters**: Without explicit scope boundaries, features grow unbounded.

---

## Spec Quality Checklist

| Question | Score |
|----------|-------|
| Can someone who's never seen this project understand the feature? | /5 |
| Is every acceptance criterion independently testable? | /5 |
| Are edge cases and error scenarios documented? | /5 |
| Are non-functional requirements specific (not "fast" or "secure")? | /5 |
| Is scope clearly bounded (in scope AND out of scope)? | /5 |
| **Total** | **/25** |

- **20–25**: Excellent spec, ready for Plan phase
- **15–19**: Good but needs refinement in weak areas
- **<15**: Needs significant rework before proceeding

---

*Next: [User Stories & Acceptance Criteria →](user-stories-and-acceptance-criteria.md)*
