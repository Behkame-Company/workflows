# Self-Healing CI

> AI agents that diagnose CI failures and automatically propose fixes.

---

## The Problem

When CI fails, the typical workflow is:
1. Developer sees red ❌ in GitHub
2. Clicks through to find the failing job
3. Scrolls through logs to find the error
4. Figures out the fix
5. Pushes a fix commit
6. Waits for CI to run again

This takes **15-60 minutes** per failure. For large teams, CI failures are a constant drain.

---

## The Self-Healing Approach

```
CI Fails ❌
    │
    ▼
┌─────────────────────────────┐
│  AI AGENT ACTIVATES         │
│                             │
│  1. Read CI failure logs    │
│  2. Identify root cause     │
│  3. Search codebase for fix │
│  4. Generate fix commit     │
│  5. Open fix PR             │
└────────────┬────────────────┘
             │
             ▼
  Human reviews fix PR → Merge ✅
```

---

## Implementation with Agentic Workflows

```markdown
---
on:
  workflow_run:
    workflows: ["CI Pipeline"]
    types: [completed]
    conclusions: [failure]
permissions:
  contents: write
  pull-requests: write
  actions: read
safe-outputs:
  create-branch: {}
  create-pull-request: {}
  add-comment: {}
tools:
  github:
    toolsets: [actions, pull_requests, contents]
---

# Self-Healing CI Agent

When the CI pipeline fails:
1. Fetch the failed job logs
2. Identify the specific error (test failure, lint error, build error)
3. Analyze the error message and stack trace
4. Search the codebase for the relevant code
5. Generate a minimal fix
6. Create a new branch and open a PR with the fix
7. Link the fix PR to the failed run
```

---

## What Self-Healing CI Can Fix

| Failure Type | AI Can Fix? | Example |
|-------------|-------------|---------|
| Lint errors | ✅ Usually | Missing semicolons, import order |
| Type errors | ✅ Often | Missing type annotations, wrong types |
| Simple test failures | ⚠️ Sometimes | Updated snapshot, changed expected value |
| Build failures | ⚠️ Sometimes | Missing dependency, import path |
| Complex logic bugs | ❌ Rarely | Business logic, race conditions |
| Infra failures | ❌ No | Network timeouts, runner issues |

---

## Guardrails

| Control | Why |
|---------|-----|
| **Fix PRs, not direct commits** | Human must review before merge |
| **Limit to specific failure types** | Don't try to fix infrastructure issues |
| **Rate limit** | Prevent infinite fix loops |
| **Escalate after N attempts** | If AI can't fix after 2 tries, alert a human |
| **No production deploys** | Self-healing is for CI, not production |

---

## Key Takeaways

1. **AI reads logs and proposes fixes** — Saves 15-60 min per CI failure
2. **Fixes come as PRs** — Human always reviews before merge
3. **Best for mechanical fixes** — Lint, type, and simple test failures
4. **Not for complex bugs** — Business logic still needs human debugging
5. **Guardrails are essential** — Rate limits, escalation, no auto-merge
