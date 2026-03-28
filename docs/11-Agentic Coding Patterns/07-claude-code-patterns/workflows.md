# Claude Code Workflows

> Common agentic patterns with Claude Code — from explore-plan-code-test loops to full feature implementation, available across terminal, VS Code, JetBrains, desktop, and web.

---

## Where Claude Code Runs

```
┌──────────────────────────────────────────────────────────┐
│              Claude Code Availability                     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   Terminal CLI      VS Code Extension     JetBrains      │
│   ────────────      ─────────────────     ─────────      │
│   claude            Sidebar panel         Plugin         │
│   Full agent        Editor-integrated     IDE-native     │
│                                                          │
│   Desktop App       Web Interface                        │
│   ───────────       ─────────────                        │
│   Standalone        Browser-based                        │
│   Multi-project     No install needed                    │
│                                                          │
│   Same agent capabilities across all interfaces          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## The Core Loop: Explore → Plan → Code → Test

Every effective agentic workflow follows this fundamental cycle:

```
┌──────────────────────────────────────────────────────────┐
│          Explore → Plan → Code → Test                    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   ┌──────────┐    ┌──────────┐                           │
│   │ EXPLORE  │───▶│   PLAN   │                           │
│   │          │    │          │                           │
│   │ Read code│    │ Outline  │                           │
│   │ Find     │    │ approach │                           │
│   │ patterns │    │ Identify │                           │
│   │ Understand│   │ files    │                           │
│   └──────────┘    └────┬─────┘                           │
│                        │                                 │
│                        ▼                                 │
│   ┌──────────┐    ┌──────────┐                           │
│   │   TEST   │◀───│   CODE   │                           │
│   │          │    │          │                           │
│   │ Run tests│    │ Write    │                           │
│   │ Verify   │    │ Implement│                           │
│   │ Iterate  │    │ Refactor │                           │
│   └────┬─────┘    └──────────┘                           │
│        │                                                 │
│        │ Fail                                            │
│        └─────────▶ Back to EXPLORE/CODE                  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Prompting the Loop

```bash
$ claude
> I need to add pagination to the /api/products endpoint.
> First, explore the current implementation and related patterns
> in the codebase. Then plan your approach before coding.
> Write tests first, then implement.
```

Claude Code naturally follows this loop, but being explicit about it produces better results.

## Workflow: Bug Fix

```
┌──────────────────────────────────────────────────────────┐
│              Bug Fix Workflow                             │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   1. REPRODUCE                                           │
│      ├── Understand the bug report                       │
│      ├── Find the relevant code                          │
│      └── Confirm the failure locally                     │
│                                                          │
│   2. WRITE FAILING TEST                                  │
│      ├── Create test that demonstrates the bug           │
│      ├── Run test — confirm it fails                     │
│      └── Test captures the exact failure condition       │
│                                                          │
│   3. FIX                                                 │
│      ├── Make minimal change to fix the bug              │
│      ├── Don't refactor (that's a separate task)         │
│      └── Run failing test — confirm it passes            │
│                                                          │
│   4. VERIFY                                              │
│      ├── Run full test suite                             │
│      ├── Check for regressions                           │
│      └── Verify edge cases                               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

```bash
$ claude
> Bug: Users with special characters in their email (like user+tag@example.com)
> can't log in. The login endpoint returns 400.
>
> 1. Find the login handler and email validation logic
> 2. Write a failing test with "user+tag@example.com"
> 3. Fix the validation
> 4. Run all auth tests to verify no regressions
```

## Workflow: Refactor

```
┌──────────────────────────────────────────────────────────┐
│              Refactor Workflow                            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   1. ANALYZE                                             │
│      ├── Identify the code smell or problem              │
│      ├── Map all usages and dependencies                 │
│      └── Verify existing test coverage                   │
│                                                          │
│   2. PLAN                                                │
│      ├── Design the target structure                     │
│      ├── Identify intermediate safe steps                │
│      └── Define "done" criteria                          │
│                                                          │
│   3. IMPLEMENT (incrementally)                           │
│      ├── Make one safe change at a time                  │
│      ├── Run tests after each change                     │
│      └── Commit at each green state                      │
│                                                          │
│   4. TEST                                                │
│      ├── Run full test suite                             │
│      ├── Verify behavior is unchanged                    │
│      └── Check performance if applicable                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

```bash
$ claude
> Refactor: The UserService class in src/services/user.ts is 800 lines.
> 1. Analyze what it does and map its dependencies
> 2. Plan how to split it into focused modules
> 3. Implement the split incrementally, running tests after each step
> 4. Ensure all existing tests still pass without modification
```

## Workflow: New Feature

```
┌──────────────────────────────────────────────────────────┐
│              New Feature Workflow                         │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   1. SPEC                                                │
│      ├── Define requirements and acceptance criteria     │
│      ├── Identify affected systems                       │
│      └── List dependencies and constraints               │
│                                                          │
│   2. INTERFACE                                           │
│      ├── Design the API / component interface            │
│      ├── Define types and contracts                      │
│      └── Write interface tests (TDD)                     │
│                                                          │
│   3. IMPLEMENT                                           │
│      ├── Build against the defined interface             │
│      ├── Follow existing patterns in the codebase        │
│      └── Handle error cases and edge conditions          │
│                                                          │
│   4. TEST                                                │
│      ├── Unit tests for logic                            │
│      ├── Integration tests for connections               │
│      └── Edge case tests for robustness                  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

```bash
$ claude
> Feature: Add webhook support for order events.
>
> Requirements:
> - POST /api/webhooks to register a webhook URL
> - Events: order.created, order.updated, order.cancelled
> - Retry failed deliveries 3 times with exponential backoff
> - Include HMAC signature for verification
>
> Start by designing the interface and types,
> then implement and test.
```

## CLAUDE.md as Persistent Memory

CLAUDE.md is Claude Code's memory file — it persists across sessions and tells the agent about your project:

```markdown
<!-- CLAUDE.md -->

# Project: OrderFlow API

## Quick Start
- Install: `npm ci`
- Test: `npm test`
- Dev: `npm run dev`
- Build: `npm run build`

## Architecture
- Express.js REST API with TypeScript
- PostgreSQL via Knex.js (not Prisma)
- Redis for caching and job queues
- Bull for background job processing

## Conventions
- Use Knex query builder, never raw SQL
- All endpoints return { data } or { error }
- Services are classes with dependency injection
- Use Winston logger, never console.log

## Common Tasks
- New endpoint: create route, service, validation schema, tests
- New migration: `npx knex migrate:make name`
- Run single test: `npm test -- --grep "pattern"`
```

### Auto-Memory

Claude Code also builds **auto-memory** — preferences it deduces from your interactions. If you consistently prefer one approach, it remembers without you writing it down.

## Tips for Effective Claude Code Workflows

1. **Be explicit about the workflow stage** — "First explore, then plan, then code"
2. **Reference CLAUDE.md** — keep it updated so the agent always has context
3. **Use concrete examples** — "Follow the pattern in `src/services/OrderService.ts`"
4. **Commit frequently** — ask Claude to commit at stable checkpoints
5. **Break large tasks** — one feature per session keeps context focused

## Next Steps

- [CLAUDE.md for Agentic Workflows](./claude-md-agents.md) — optimize your CLAUDE.md for agent performance
- [Multi-Window Development](./multi-window.md) — run parallel Claude Code sessions
- [Headless and CI Mode](./headless-ci.md) — automate Claude Code in CI pipelines
- [Start Simple](../09-best-practices/start-simple.md) — choose the right level of complexity
