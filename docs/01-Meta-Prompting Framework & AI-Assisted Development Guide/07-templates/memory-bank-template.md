# Memory Bank Template

> These templates correspond to the Memory Bank pattern described in [Memory Bank for Teams](../04-memory-bank/memory-bank-for-teams.md).

> Copy this structure to `memory-bank/` in your repository root and customize each file.

---

## File: `memory-bank/projectbrief.md`

```markdown
# Project Brief

## Overview
<!-- One paragraph: what this project is and who it's for -->
[Project name] is a [type of application] designed for [target users].
It [primary function/value proposition].

## Core Requirements
<!-- The non-negotiable features that define the product -->
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]
- [Requirement 4]

## Success Criteria
<!-- How you'll know the project is successful -->
- [Metric 1]
- [Metric 2]
- [Metric 3]

## Out of Scope
<!-- Explicitly excluded to prevent scope creep -->
- [Excluded feature 1]
- [Excluded feature 2]
```

---

## File: `memory-bank/productContext.md`

```markdown
# Product Context

## Problem Statement
<!-- What problem does this solve? For whom? -->
[Target users] currently [pain point]. This results in [negative outcome].

## Solution
<!-- How does this project address the problem? -->
[Project name] provides [solution approach] so that [positive outcome].

## User Experience Goals
<!-- How should the product feel to use? -->
- [UX goal 1, e.g., "First value within 5 minutes of sign-up"]
- [UX goal 2, e.g., "Core workflow completes in under 3 clicks"]
- [UX goal 3, e.g., "Accessible to users with no technical background"]

## Target Users
<!-- Who are the primary user personas? -->
- **[Persona 1]**: [Brief description and primary need]
- **[Persona 2]**: [Brief description and primary need]
```

---

## File: `memory-bank/systemPatterns.md`

```markdown
# System Patterns

## Architecture
<!-- High-level architectural decisions -->
- [Architecture pattern, e.g., "Next.js App Router with React Server Components"]
- [Data layer, e.g., "PostgreSQL via Prisma ORM"]
- [API pattern, e.g., "tRPC for type-safe client-server communication"]
- [Caching, e.g., "Redis for API response caching"]

## Key Design Patterns
<!-- Patterns the team has adopted -->
- [Pattern 1, e.g., "Repository pattern for data access (src/repositories/)"]
- [Pattern 2, e.g., "Service layer for business logic (src/services/)"]
- [Pattern 3, e.g., "Compound Component pattern for complex UI"]

## Critical Decisions Log
<!-- Major decisions with rationale — append as decisions are made -->
- **[Date]**: [Decision]. Rationale: [Why this over alternatives].
- **[Date]**: [Decision]. Rationale: [Why].
```

---

## File: `memory-bank/techContext.md`

```markdown
# Tech Context

## Stack
- **Runtime**: [e.g., Node.js 20 LTS]
- **Framework**: [e.g., Next.js 15.1]
- **Language**: [e.g., TypeScript 5.4 (strict mode)]
- **Styling**: [e.g., Tailwind CSS 3.4]
- **Database**: [e.g., PostgreSQL 16 via Prisma 5.x]
- **Testing**: [e.g., Jest + React Testing Library + Playwright]
- **CI/CD**: [e.g., GitHub Actions]

## Environment Setup
<!-- Step-by-step for a fresh checkout -->
1. [Prerequisite, e.g., "Install Node.js 20 via nvm"]
2. `npm install`
3. [Config step, e.g., "Copy .env.example to .env.local"]
4. [DB step, e.g., "npm run db:migrate && npm run db:seed"]
5. `npm run dev`

## Known Constraints
<!-- Technical limitations the AI should know about -->
- [Constraint 1, e.g., "Third-party API rate limits: 100 req/min"]
- [Constraint 2, e.g., "PDF export requires Chrome/Chromium in CI"]
- [Constraint 3, e.g., "Prisma Client must regenerate after schema changes"]

## Key Dependencies
<!-- Non-obvious dependencies worth calling out -->
- [Library]: [What it's used for and any gotchas]
```

---

## File: `memory-bank/activeContext.md`

```markdown
# Active Context

## Current Sprint: [Sprint Name] ([Date Range])

## In Progress
<!-- What's actively being worked on -->
- [ ] [Task 1]: [Brief status, e.g., "API done, frontend 60% complete"]
- [ ] [Task 2]: [Brief status]

## Recently Completed
<!-- Finished in the last 1-2 sprints -->
- ✅ [Completed task 1]
- ✅ [Completed task 2]

## Active Decisions
<!-- Decisions being discussed or recently made -->
- PENDING: [Decision being discussed]
- DECIDED: [Recent decision and brief rationale]

## Blockers
<!-- Anything blocking progress -->
- [Blocker description and who/what can unblock it]

## Next Up
<!-- Coming soon but not started -->
- [Upcoming task 1]
- [Upcoming task 2]
```

---

## File: `memory-bank/progress.md`

```markdown
# Progress

## Milestone Status

| Milestone | Status | Notes |
|-----------|--------|-------|
| [Milestone 1] | ✅ Done | [Brief note] |
| [Milestone 2] | 🔄 In Progress | [Brief note] |
| [Milestone 3] | 📋 Planned | [Target date or sprint] |
| [Milestone 4] | ❌ Deferred | [Reason] |

## Known Issues

| # | Issue | Severity | Status |
|---|-------|----------|--------|
| [#ID] | [Description] | [High/Med/Low] | [Open/In Progress/Workaround] |

## Technical Debt

- [Debt item 1]: [Impact and plan to address]
- [Debt item 2]: [Impact and plan to address]
```
