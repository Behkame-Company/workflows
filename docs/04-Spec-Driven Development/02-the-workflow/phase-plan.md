# Phase 2: Plan

> Translating approved specifications into technical architecture and implementation strategy.

---

## Purpose

The Plan phase answers: **How will we build what the spec describes?**

It bridges the gap between *what* (spec) and *doing* (tasks + implementation) by defining architecture, technology choices, and integration strategy.

---

## Plan Structure Template

```markdown
# Plan: [Feature Name]

## Overview
Brief summary of the technical approach.

## Architecture
How components are structured and interact.

### Component Diagram
- [Component A] — handles [responsibility]
- [Component B] — handles [responsibility]
- [Component A] ──calls──► [Component B]

## Data Model
Tables, schemas, or data structures required.

### New Tables
| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK, auto-generated |
| name | VARCHAR(255) | NOT NULL |

### Modified Tables
- `users` table: Add `notification_preferences` JSONB column

## API Design
Endpoints and contracts.

### POST /api/notifications
- Request: `{ userId, message, type }`
- Response: `{ id, status, createdAt }`
- Errors: 400 (invalid input), 404 (user not found)

## Integration Points
How this connects to existing systems.

- Auth service: Validates user tokens
- Email service: Sends notification emails
- WebSocket server: Delivers real-time notifications

## Testing Strategy
How we'll verify the implementation.

- Unit tests: Service logic, validators
- Integration tests: API endpoints, database queries
- E2E tests: Full notification flow from trigger to delivery

## Technical Risks
Known uncertainties and mitigation plans.

| Risk | Impact | Mitigation |
|------|--------|------------|
| WebSocket scaling | High | Use Redis pub/sub for horizontal scaling |
| Email deliverability | Medium | Integrate with established provider (SendGrid) |

## File Structure
New files and directories to create.

```

---

## Planning Principles

### 1. Spec Traceability
Every section of the plan should trace back to a spec requirement:

```markdown
## Requirement: Users can set notification preferences (Spec §3.2)
### Plan: Add preferences API + settings UI component
```

### 2. Architecture Decision Records
Document *why* a choice was made, not just *what* was chosen:

```markdown
### Decision: Use WebSockets for real-time notifications
**Alternatives considered**: Polling, Server-Sent Events (SSE)
**Chosen because**: Bidirectional communication needed for read receipts
**Trade-off**: More complex infrastructure, but better UX
```

### 3. Minimal Viable Architecture
Plan the simplest architecture that fulfills the spec. Avoid over-engineering:

```
❌  "We'll build a microservices mesh with service discovery..."
     (for a feature used by 10 people)

✅  "Single service module with clear boundaries for future extraction"
```

### 4. Existing Code Awareness
The plan must account for the existing codebase:

```markdown
## Integration with Existing Code
- Extends existing `UserService` class (don't create new service)
- Follows established repository pattern in `/src/repositories/`
- Uses existing error handling middleware
- Adds to existing API router in `/src/routes/`
```

---

## AI-Assisted Planning

When using AI to generate plans:

### Good Prompts
```
"Given this spec [paste spec.md], generate a technical plan.
Consider our existing architecture:
- Next.js 14 app router
- PostgreSQL with Prisma ORM
- Existing auth via NextAuth
The plan should reuse existing patterns where possible."
```

### What to Review
- [ ] Does the plan address every spec requirement?
- [ ] Are architecture choices appropriate for the project scale?
- [ ] Does it integrate properly with existing code?
- [ ] Are there any unnecessary new dependencies?
- [ ] Is the testing strategy comprehensive?

---

## Plan Artifacts

The Plan phase may produce additional artifacts alongside `plan.md`:

| Artifact | When Needed |
|----------|-------------|
| `data-model.md` | Complex database changes |
| `api-contract.md` | New API surface area |
| `research.md` | When tech choices need investigation |
| `migration-plan.md` | Data or schema migrations required |
| `security-review.md` | Features with auth/data sensitivity |

---

## Quality Gate: Plan Review

Before proceeding to Tasks:

### Coverage Check
For each spec requirement, verify a plan section exists:
```
Spec Requirement 1  ──►  Plan §2.1 Architecture  ✅
Spec Requirement 2  ──►  Plan §2.3 API Design    ✅
Spec Requirement 3  ──►  ???                      ❌ Gap!
```

### Feasibility Check
- Can the planned architecture handle the non-functional requirements?
- Are all integration points available?
- Are there any blocking dependencies?

### Consistency Check
- Does the plan contradict itself?
- Are naming conventions consistent?
- Does the data model support all the API operations?

---

*Next: [Phase 3: Tasks →](phase-tasks.md)*
