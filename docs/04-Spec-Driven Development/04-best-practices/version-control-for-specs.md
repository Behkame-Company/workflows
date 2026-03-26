# Version Control for Specs

> Treating specifications as first-class engineering artifacts in your repository.

---

## Why Version Control Specs?

Specs that live in Google Docs, Confluence, or Notion:
- Drift from the code they describe
- Have opaque change history
- Aren't discoverable alongside code
- Can't be referenced in commits or PRs
- Break the spec-to-code traceability chain

**Specs belong in the repository, version-controlled alongside the code they specify.**

---

## Repository Structure

### Spec Kit Default
```
.specify/
├── constitution.md          # Project-wide principles
├── specs/
│   ├── user-auth/
│   │   ├── spec.md          # Feature specification
│   │   ├── plan.md          # Technical plan
│   │   ├── tasks.md         # Task breakdown
│   │   └── research.md      # Optional research notes
│   ├── notifications/
│   │   ├── spec.md
│   │   ├── plan.md
│   │   └── tasks.md
│   └── shopping-cart/
│       └── spec.md          # In-progress (plan not yet created)
```

### Alternative Structure (No Spec Kit)
```
specs/
├── constitution.md
├── active/
│   ├── feature-notifications.md   # Currently being built
│   └── feature-cart.md
├── approved/
│   └── feature-search.md          # Approved, not yet started
├── completed/
│   └── feature-auth.md            # Implemented, now reference
└── templates/
    ├── feature-spec-template.md
    └── api-spec-template.md
```

---

## Git Workflow for Specs

### Spec Branch Strategy

```
main
├── spec/notifications    ← Spec creation and review
│   └── PR: "Spec: Notification system"
│       ├── .specify/specs/notifications/spec.md
│       └── Reviewed and merged
│
├── plan/notifications    ← Plan creation
│   └── PR: "Plan: Notification system"
│       └── .specify/specs/notifications/plan.md
│
└── feat/notifications    ← Implementation (references spec)
    └── PR: "Feat: Notification system (Spec: #42)"
        ├── src/features/notifications/...
        └── .specify/specs/notifications/tasks.md (updated)
```

### Commit Messages

Reference specs in commit messages:

```
spec: add notification system specification

Defines real-time notification system with in-app, email channels.
Includes 12 acceptance criteria, 8 edge cases, performance SLAs.

Spec: .specify/specs/notifications/spec.md
```

```
feat(notifications): implement real-time delivery

Implements Tasks 1-3 of notification spec.
WebSocket-based delivery with Redis pub/sub fallback.

Spec: .specify/specs/notifications/spec.md §2.1-2.3
Plan: .specify/specs/notifications/plan.md §3
```

---

## Code Review for Specs

### Spec PR Checklist

```markdown
## Spec Review: [Feature Name]

### Completeness
- [ ] Summary explains what and why
- [ ] User stories cover all personas
- [ ] Acceptance criteria are testable
- [ ] Edge cases documented
- [ ] Non-functional requirements specified
- [ ] Scope boundaries defined (in + out)

### Quality
- [ ] Language is clear and unambiguous
- [ ] Values are specific (not "fast" or "many")
- [ ] No implementation details (what, not how)
- [ ] AI agents could implement from this spec

### Consistency
- [ ] Follows constitution guidelines
- [ ] Follows spec template structure
- [ ] Naming aligns with existing features
```

---

## Spec Lifecycle States

Track spec status with git-friendly markers:

```markdown
<!-- Status: Draft | In Review | Approved | In Progress | Completed | Archived -->
<!-- Status: Approved -->
```

Or use a frontmatter block:
```yaml
---
status: approved
version: 1.2
author: jane
reviewers: [bob, alice]
approved_date: 2026-01-20
---
```

---

## Linking Specs to Code

### In PRs
```markdown
## Pull Request: Notification System

**Spec**: .specify/specs/notifications/spec.md
**Plan**: .specify/specs/notifications/plan.md
**Tasks**: .specify/specs/notifications/tasks.md

### Acceptance Criteria Status
- [x] AC-1.1: Real-time delivery within 3s ✅
- [x] AC-1.2: Unread count badge ✅
- [ ] AC-2.1: Preference settings — in next PR
```

### In Code Comments (sparingly)
```typescript
// Implements Spec §3.2: Rate limit to 100 notifications/hour/user
const RATE_LIMIT = 100;
const RATE_WINDOW = 60 * 60 * 1000; // 1 hour in ms
```

---

## Spec Archival

When features are complete and stable:

```
Option A: Keep specs in place (recommended)
  .specify/specs/notifications/spec.md  ← stays forever, becomes documentation

Option B: Move to archive
  .specify/archive/2026-Q1/notifications/spec.md

Option C: Tag and clean
  git tag spec/notifications/v1.0
  # Remove from active directory
```

**Recommendation**: Keep specs in place. They serve as documentation for anyone modifying the feature later.

---

*Next: [Human-in-the-Loop Review →](human-in-the-loop.md)*
