# Minimal Viable Context

> The smallest possible set of high-signal tokens that maximizes the likelihood of the desired outcome.

---

## The Guiding Principle

Anthropic's core context engineering principle:

> **Find the smallest possible set of high-signal tokens that maximize the likelihood of your desired outcome.**

This means:
- Every token should earn its place
- If it doesn't help, it actively hurts (dilutes attention)
- Context is a finite resource with diminishing returns
- Less noise = more signal = better results

---

## The MVC Framework

Like MVP (Minimum Viable Product), **MVC (Minimum Viable Context)** asks: what's the absolute minimum context needed for the model to do this task well?

### For a Bug Fix
```
MVC:
  1. Error message (50 tokens)
  2. Stack trace location (100 tokens)
  3. The buggy function (200 tokens)
  4. Expected behavior (50 tokens)
  Total: ~400 tokens

Over-context:
  1. Entire project README (2,000 tokens)
  2. All auth module files (5,000 tokens)
  3. Test suite output (3,000 tokens)
  4. Package.json (500 tokens)
  5. Previous conversation (10,000 tokens)
  Total: ~20,500 tokens

Same fix. 50x more context. Worse results.
```

### For a New Feature
```
MVC:
  1. Feature requirements (200 tokens)
  2. Existing patterns to follow (300 tokens)
  3. Files to modify/create (100 tokens)
  4. Constraints and standards (200 tokens)
  Total: ~800 tokens

The agent can retrieve additional context (file contents,
API docs) on demand as it implements.
```

---

## How to Find Your MVC

### Step 1: Start Minimal
Give the model just the task description and constraints. See what it asks for.

### Step 2: Add What's Missing
If the model makes wrong assumptions, add context to correct them. But add the **specific** context, not "everything."

### Step 3: Remove What's Unused
After the task, audit: which context tokens did the model actually reference in its response? Remove what it ignored.

### Step 4: Iterate
Over time, you learn the MVC for your common tasks.

---

## MVC for Common Development Tasks

| Task | Minimum Viable Context |
|------|----------------------|
| **Fix a bug** | Error + location + buggy code + expected behavior |
| **Add a feature** | Requirements + existing patterns + file structure |
| **Write tests** | Function to test + existing test patterns |
| **Code review** | Diff + coding standards + critical rules |
| **Refactoring** | Current code + target pattern + constraints |
| **Architecture** | Project overview + requirements + constraints |

---

## Key Takeaways

1. **Less is more** — 400 focused tokens > 20,000 unfocused tokens
2. **Start minimal, add as needed** — Don't front-load everything
3. **Let the agent retrieve details** — MVC + on-demand retrieval
4. **Audit unused context** — Remove what the model didn't reference
5. **MVC varies by task** — Bug fixes need less than architecture decisions

---

## Next Steps

- 🔗 [Context Hygiene](context-hygiene.md) — Keeping context clean over time
- 🔗 [Debugging Context Issues](debugging-context-issues.md) — When things go wrong
