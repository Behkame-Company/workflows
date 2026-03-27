# Automated PR Review Pipelines

> Setting up AI-powered code review as an automated step in your CI/CD pipeline.

---

## Architecture

```
PR Opened/Updated
        │
        ├── GitHub Actions triggers
        │
        ▼
┌───────────────────┐  ┌───────────────────┐  ┌───────────────────┐
│  Lint + Build     │  │  Test + Coverage   │  │  Security Scan    │
│  (deterministic)  │  │  (deterministic)   │  │  (deterministic)  │
└────────┬──────────┘  └────────┬───────────┘  └────────┬──────────┘
         │                      │                       │
         └──────────────────────┼───────────────────────┘
                                │
                                ▼
                  ┌───────────────────────┐
                  │   AI Code Review      │
                  │   (Copilot / CodeRabbit │
                  │    / PR-Agent)         │
                  └───────────┬───────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │   Human Review        │
                  │   (AI pre-filtered)   │
                  └───────────┬───────────┘
                              │
                              ▼
                        ✅ Merge
```

---

## Implementation Options

### Option 1: Copilot Native (Zero Config)

Set up via repository rulesets — no Actions needed:
1. Settings → Rules → New Ruleset
2. Enable "Automatically request Copilot code review"
3. Done — Copilot reviews every PR automatically

### Option 2: PR-Agent via GitHub Actions

```yaml
name: AI Code Review
on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - name: PR-Agent Review
        uses: qodo-ai/pr-agent@main
        env:
          OPENAI_KEY: ${{ secrets.OPENAI_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          command: review
```

### Option 3: Custom AI Review Action

```yaml
- name: Custom AI Review
  uses: your-org/ai-review-action@v1
  with:
    model: gpt-4
    instructions: |
      Focus on security and performance.
      Ignore formatting (handled by Prettier).
      Flag any new API endpoint without input validation.
    severity-threshold: medium
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| **Run AI review after tests pass** | Don't waste AI tokens on broken code |
| **Set severity thresholds** | Only block merge for critical/high findings |
| **Use custom instructions** | Guide AI to your team's standards |
| **Post results as PR comments** | Visible, actionable, discussable |
| **Track acceptance rate** | Measure how often AI suggestions are applied |

---

## Key Takeaways

1. **Copilot native is easiest** — Zero config, works out of the box
2. **PR-Agent for customization** — Open source, configurable
3. **Run after deterministic checks** — AI reviews clean, passing code
4. **Custom instructions matter** — Guide the AI to what your team cares about
5. **Don't block on low-severity** — Only critical/high should prevent merge

---

## Next Steps

- 🔗 [Self-Healing CI](self-healing-ci.md) — AI that fixes failures
- 🔗 [Review Instructions](../08-best-practices/review-instructions.md) — Writing effective rules
