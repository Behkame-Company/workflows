# Memory Bank Pattern

> The Cline-inspired approach to persistent project knowledge across sessions.

---

## What Is the Memory Bank?

The Memory Bank pattern (popularized by Cline) uses a structured set of markdown files to maintain project context across AI sessions. Each file has a specific purpose and is read at session start.

```
.memory-bank/
├── projectbrief.md        # Foundation: what the project is
├── productContext.md       # Why it exists, user problems
├── systemPatterns.md       # Architecture and design patterns
├── techContext.md          # Technologies, dependencies, setup
├── activeContext.md        # Current focus, recent changes
└── progress.md            # What works, what doesn't, what's next
```

---

## File Purposes

### projectbrief.md
The foundation document. Everything else builds on this:
```markdown
# Project Brief
## Overview
E-commerce platform for artisan products with real-time inventory.

## Core Requirements
- User authentication and profiles
- Product catalog with search
- Shopping cart and checkout
- Real-time inventory tracking
- Seller dashboard
```

### activeContext.md
**Most frequently updated** — current working state:
```markdown
# Active Context
## Current Focus
Implementing payment processing with Stripe.

## Recent Changes
- Added Stripe SDK integration (src/payments/)
- Created webhook handler for payment events
- Updated order model with payment status

## Active Decisions
- Using Stripe Payment Intents (not Checkout Sessions)
- Webhook secret stored in environment variables

## Next Steps
1. Complete refund handling
2. Add payment method management
3. Write integration tests
```

### progress.md
High-level project status:
```markdown
# Progress

## Completed
- ✅ User auth (JWT + refresh tokens)
- ✅ Product CRUD API
- ✅ Search with Elasticsearch
- ✅ Shopping cart

## In Progress
- 🔄 Payment processing (Stripe)

## Not Started
- ⬜ Seller dashboard
- ⬜ Real-time inventory
- ⬜ Email notifications
```

---

## The Update Cycle

```
Session Start:
  1. Read ALL memory bank files
  2. Understand current state
  3. Begin work

During Session:
  4. Make progress on task
  5. Update activeContext.md with changes

Session End:
  6. Update progress.md
  7. Update activeContext.md with next steps
  8. Update other files if architecture/tech changed
```

---

## Memory Bank vs. Other Approaches

| Approach | Persistence | Structure | Best For |
|----------|------------|-----------|----------|
| **Memory Bank** | Files in repo | Pre-defined templates | Project continuity |
| **Memory Tool** | External storage | Agent-managed | Cross-session learning |
| **CLAUDE.md** | Single file in repo | Freeform | Quick project context |
| **Copilot Instructions** | .github/ directory | AI tool config | Tool-specific rules |

---

## Adapting for Copilot CLI

The Memory Bank pattern works with any AI tool. For Copilot CLI:

```
.github/
├── copilot-instructions.md    # Tool-specific instructions
└── memory-bank/               # Project knowledge (alternative location)
    ├── projectbrief.md
    ├── activeContext.md
    └── progress.md
```

Reference in copilot-instructions.md:
```markdown
Before starting work, read the memory bank files in .github/memory-bank/
to understand the current project state and context.
```

---

## Key Takeaways

1. **Structured files beat freeform notes** — Each file has a clear purpose
2. **activeContext.md is the heartbeat** — Updated most frequently
3. **Read all files at session start** — Full context restoration
4. **Update at natural breakpoints** — After each feature/task completion
5. **Works with any AI tool** — Not specific to Cline
