# GitHub Actions Basics

> The fundamentals of GitHub Actions — workflows, triggers, jobs, and steps.

---

## Core Concepts

| Concept | Definition |
|---------|-----------|
| **Workflow** | A YAML file in `.github/workflows/` that defines an automation |
| **Trigger (on:)** | The event that starts the workflow (push, PR, schedule, etc.) |
| **Job** | A set of steps that runs on a single runner |
| **Step** | An individual task within a job (run a command or use an action) |
| **Action** | A reusable unit of automation (e.g., `actions/checkout@v4`) |
| **Runner** | The machine that executes the job (GitHub-hosted or self-hosted) |
| **Artifact** | A file produced by a job (test reports, build outputs) |
| **Secret** | An encrypted variable for sensitive data (API keys, tokens) |

---

## Basic Workflow Structure

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run type-check

      - name: Test
        run: npm test -- --coverage

      - name: Build
        run: npm run build
```

---

## Common Triggers

| Trigger | When It Fires |
|---------|--------------|
| `push` | Code pushed to a branch |
| `pull_request` | PR opened, updated, or reopened |
| `schedule` | Cron-based schedule (e.g., nightly) |
| `workflow_dispatch` | Manual trigger from GitHub UI |
| `issue_comment` | Comment on an issue or PR |
| `release` | New release published |

---

## Multi-Job Pipeline

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint                    # Runs after lint passes
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test

  security:
    runs-on: ubuntu-latest
    needs: lint                    # Runs in parallel with test
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
      - uses: github/codeql-action/analyze@v3

  deploy:
    runs-on: ubuntu-latest
    needs: [test, security]        # Runs after both pass
    if: github.ref == 'refs/heads/main'
    steps:
      - run: echo "Deploying to production..."
```

---

## Key Takeaways

1. **Workflows are YAML** — Defined in `.github/workflows/`
2. **Trigger on events** — push, PR, schedule, manual
3. **Jobs run on runners** — Parallel by default, use `needs:` for ordering
4. **Actions are reusable** — Thousands available on GitHub Marketplace
5. **Secrets for sensitive data** — Never hardcode credentials

---

## Next Steps

- 🔗 [Quality Gates](quality-gates.md) — Adding policy enforcement
- 🔗 [Agentic Workflows](../04-ai-in-ci-cd/agentic-workflows.md) — AI-powered automation
