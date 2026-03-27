# Templates

> Ready-to-use configuration templates for context window management.

---

## Template 1: CLAUDE.md (Context-Optimized)

```markdown
# Project: [Name]

## Quick Start
[one-line command to run the project]

## Architecture
- Frontend: [framework] in /[dir]
- Backend: [framework] in /[dir]
- Database: [type] with [ORM]
- Auth: [approach] in /[dir]

## Conventions
- [3-5 critical rules, not 30]
- Tests in [location] mirroring source structure

## Current State
See progress.txt for current work.
Read feature_list.json for scope.

## Context Management
When context runs low:
1. Write progress to progress.txt
2. Commit all work
3. Update feature_list.json
```

---

## Template 2: progress.txt

```
=== Session [N] ([Date]) ===

Completed:
- [Feature/task completed with file references]

State:
- All tests: [passing/failing count]
- Dev server: [running on port X]
- Feature progress: [X/Y features complete]

In Progress:
- [Current task, if any]

Next Priority:
- [Highest-priority remaining work]
- Files to modify: [specific paths]

Known Issues:
- [Any bugs or blockers]
```

---

## Template 3: feature_list.json

```json
{
  "features": [
    {
      "id": 1,
      "category": "authentication",
      "description": "User can register with email and password",
      "steps": [
        "Navigate to registration page",
        "Enter valid email and password",
        "Submit form",
        "Verify account created and redirect to dashboard"
      ],
      "passes": false
    },
    {
      "id": 2,
      "category": "authentication",
      "description": "User can log in with valid credentials",
      "steps": [
        "Navigate to login page",
        "Enter valid credentials",
        "Submit form",
        "Verify redirect to dashboard with user data"
      ],
      "passes": false
    }
  ]
}
```

---

## Template 4: Compaction Configuration (Anthropic API)

```python
context_management = {
    "edits": [{
        "type": "compact_20260112",
        "trigger": {
            "type": "token_count",
            "token_count": 150000  # 75% of 200K window
        },
        "pause_after_compaction": False,
        "instructions": """Summarize this conversation, preserving:
1. All architectural and design decisions with rationale
2. Current task state (done, in progress, next)
3. Unresolved errors or blockers
4. Modified file paths
5. Key technical constraints
6. Test results and coverage

Be concise but preserve all technical details needed
to continue this work session."""
    }]
}
```

---

## Template 5: copilot-instructions.md (Lean Version)

```markdown
# Project Instructions

## Stack
Next.js 14 (App Router), TypeScript strict, Prisma, PostgreSQL

## Rules (Critical)
- Zod for input validation on all API boundaries
- No `any` types — use proper generics
- Error format: { error: string, code: string, details?: object }
- All public functions need return type annotations

## File Structure
- app/ — Next.js pages and layouts
- src/lib/ — Shared utilities and types
- src/api/ — API route handlers
- prisma/ — Schema and migrations
- tests/ — Mirrors source structure

## Context
Read specific files when needed. Don't ask for file pastes.
```

---

## Template 6: Memory Bank (Minimal)

```
.github/memory-bank/
├── projectbrief.md        # 10 lines: what and why
├── techContext.md          # 10 lines: stack and setup
├── activeContext.md        # Current focus and recent changes
└── progress.md            # Completed / in-progress / next
```

---

## Next Steps

- 🔗 [Troubleshooting](troubleshooting.md) — Common problems
- 🔗 [FAQ](faq.md) — Quick answers
