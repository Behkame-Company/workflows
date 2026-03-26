# Memory Bank for Teams

> Persistent project context that survives across sessions and context window resets.

---

## The Problem

AI coding tools are **stateless by default** — each new session or context window reset starts from scratch. The AI doesn't remember:

- What architecture decisions were made yesterday
- What feature is currently in progress
- Which approaches were tried and rejected
- What the team's active priorities are

**Instruction files** (copilot-instructions.md, AGENTS.md) solve the *conventions* problem. **Memory bank** solves the *continuity* problem.

---

## What Is a Memory Bank?

A memory bank is a set of structured Markdown files in your repository that capture persistent project knowledge. The AI reads these files at the start of each session to restore context.

```
memory-bank/
├── projectbrief.md      # What this project is (stable, rarely changes)
├── productContext.md     # Why it exists, who it's for
├── systemPatterns.md     # Architecture and design patterns
├── techContext.md        # Stack, setup, constraints
├── activeContext.md      # Current work focus (changes frequently)
└── progress.md           # Status, milestones, known issues
```

---

## Core Files Explained

### `projectbrief.md` — The Foundation

The source of truth for project scope. Created once, updated only for major pivots.

```markdown
# Project Brief

## Overview
Marketing analytics SaaS dashboard that aggregates data from Google Ads,
Meta Ads, and TikTok Ads into unified reporting views.

## Core Requirements
- Multi-tenant architecture (each customer sees only their data)
- Real-time data sync from ad platform APIs
- Customizable dashboard with drag-and-drop widgets
- Export reports to PDF and CSV

## Out of Scope (for now)
- Mobile app (web-responsive only)
- White-labeling
- Self-hosted deployment option
```

### `productContext.md` — The Why

Why the project exists and how it should feel to users.

```markdown
# Product Context

## Problem
Marketing teams juggle 3-5 ad platforms, each with different reporting UIs.
They waste hours manually combining data for stakeholder reports.

## Solution
Unified dashboard that auto-syncs data and provides cross-platform insights.

## User Experience Goals
- First dashboard visible within 5 minutes of sign-up
- Data refresh within 15 minutes of ad platform changes
- Export a report in 3 clicks or fewer
```

### `systemPatterns.md` — The Architecture

Key technical decisions that shape implementation.

```markdown
# System Patterns

## Architecture
- Next.js App Router with React Server Components
- API layer: tRPC for type-safe client-server communication
- Database: PostgreSQL with Prisma ORM
- Queue: BullMQ for background sync jobs
- Cache: Redis for API response caching

## Key Patterns
- Repository pattern for data access (`src/repositories/`)
- Service layer for business logic (`src/services/`)
- Components follow Compound Component pattern for complex UI
- All forms use React Hook Form + Zod validation

## Critical Decisions
- DECISION: Use RSC over client-side fetching for initial loads (performance)
- DECISION: Multi-tenant via `organizationId` column, not schema separation
- DECISION: Background sync over real-time webhooks (platform API limitations)
```

### `techContext.md` — The Stack

Technical details the AI needs to avoid guessing.

```markdown
# Tech Context

## Stack
- Runtime: Node.js 20 LTS
- Framework: Next.js 15.1
- Language: TypeScript 5.4 (strict mode)
- Styling: Tailwind CSS 3.4
- Database: PostgreSQL 16 via Prisma 5.x
- Testing: Jest + React Testing Library + Playwright

## Environment Setup
1. `nvm use 20` (or equivalent)
2. `npm install`
3. Copy `.env.example` to `.env.local` and fill in values
4. `npm run db:migrate` to apply migrations
5. `npm run db:seed` for development data
6. `npm run dev` to start

## Known Constraints
- Ad platform APIs have rate limits (documented in `src/lib/api-clients/`)
- PDF export uses Puppeteer — requires Chrome/Chromium in CI
- Prisma Client must be regenerated after schema changes
```

### `activeContext.md` — The Current Focus ⚡

**This file changes most frequently.** It tracks what's happening right now.

```markdown
# Active Context

## Current Sprint: Q1 Sprint 3 (March 10–24)

## In Progress
- [ ] TikTok Ads integration (API client + sync job)
  - API client done, sync job 60% complete
  - Blocked: TikTok sandbox rate limits during testing
- [ ] Dashboard widget reordering (drag-and-drop)
  - Using @dnd-kit/sortable — prototype working

## Recently Completed
- ✅ Google Ads sync reliability fix (retry logic added)
- ✅ CSV export performance optimization (streaming)

## Active Decisions
- PENDING: Whether to use SSE or polling for real-time sync status
- DECIDED: Use @dnd-kit over react-beautiful-dnd (better RSC compat)

## Next Up
- PDF export redesign (Puppeteer → React-PDF)
- User permissions (role-based access control)
```

### `progress.md` — The Big Picture

High-level milestones and known issues.

```markdown
# Progress

## Milestone Status
| Milestone | Status | Notes |
|-----------|--------|-------|
| Core dashboard | ✅ Done | Shipped v1.0 |
| Google Ads integration | ✅ Done | |
| Meta Ads integration | ✅ Done | |
| TikTok Ads integration | 🔄 In Progress | Sprint 3 |
| Export (CSV/PDF) | 🔄 In Progress | CSV done, PDF redesign pending |
| User permissions | 📋 Planned | Sprint 4 |
| White-labeling | ❌ Deferred | Out of scope for v1 |

## Known Issues
- #142: PDF export crashes on dashboards with >20 widgets
- #156: Meta Ads sync occasionally misses data for accounts with >50 campaigns
- #161: Drag-and-drop jitters on Safari 17
```

---

## Team Workflow

### When to Update Memory Bank

| Trigger | Files to Update |
|---------|-----------------|
| Start of sprint | `activeContext.md`, `progress.md` |
| Feature shipped | `activeContext.md`, `progress.md` |
| Architecture decision made | `systemPatterns.md`, `activeContext.md` |
| New dependency added | `techContext.md` |
| Major pivot or scope change | `projectbrief.md`, `productContext.md` |
| End of session / context reset | `activeContext.md` (capture state) |

### Who Updates What

| Role | Primary Files |
|------|---------------|
| Tech Lead | `systemPatterns.md`, `techContext.md` |
| Product Owner | `projectbrief.md`, `productContext.md` |
| Sprint Lead | `activeContext.md`, `progress.md` |
| Individual Dev | `activeContext.md` (their current work) |

### Trigger Phrase

When starting a new session, prompt the AI:

> "Read the memory bank files in `memory-bank/` to understand current project context, then continue from where we left off."

Before ending a session:

> "Update `memory-bank/activeContext.md` with the current state of our work."

---

## Integration with Copilot

### Option 1: Reference in copilot-instructions.md

```markdown
## Project Context
For current project status and architecture decisions, refer to the
memory-bank/ directory. Always read memory-bank/activeContext.md before
starting work to understand current priorities and in-progress items.
```

### Option 2: Attach as Context

In Copilot Chat, attach `memory-bank/activeContext.md` and `memory-bank/systemPatterns.md` as files when starting complex tasks.

### Option 3: Use in Prompt Files

```markdown
# Continue Feature Work

## Context
- [Active Context](../../memory-bank/activeContext.md)
- [System Patterns](../../memory-bank/systemPatterns.md)
- [Tech Context](../../memory-bank/techContext.md)

## Instructions
Review the active context to understand what's in progress.
Continue the current work, following established system patterns.
Update activeContext.md when done.
```

---

## Best Practices

### Keep Files Concise

| File | Target Size | Max Size |
|------|-------------|----------|
| `projectbrief.md` | Half page | 1 page |
| `productContext.md` | Half page | 1 page |
| `systemPatterns.md` | 1 page | 2 pages |
| `techContext.md` | 1 page | 2 pages |
| `activeContext.md` | 1 page | 2 pages |
| `progress.md` | 1 page | 2 pages |

**Total budget**: ~6–8 pages of Markdown. Beyond this, compress or archive.

### Archive Completed Context

When a sprint ends or a major feature ships, move completed items from `activeContext.md` to `progress.md` or archive them entirely. Don't let active context accumulate indefinitely.

### Version Control

- ✅ Commit memory bank files to Git — they're project documentation
- ✅ Review changes in PRs just like code
- ⚠️ Consider `.gitignore`-ing personal scratch notes within the memory bank
- ❌ Don't store secrets, credentials, or sensitive customer data in memory bank files

---

## Optional: Additional Context Files

For complex projects, add domain-specific files:

```
memory-bank/
├── ... (core files) ...
├── integrations/
│   ├── google-ads.md       # Integration-specific context
│   └── tiktok-ads.md
├── api-contracts/
│   └── dashboard-api.md    # API design decisions
└── decisions/
    └── 2024-q4-decisions.md  # Historical decision log
```

---

*📋 Template: [Memory Bank Template](../07-templates/memory-bank-template.md)*

*Next: [Framework Architecture](../05-meta-prompting-framework/framework-architecture.md) →*
