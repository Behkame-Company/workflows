# Pipeline Architecture

> Designing multi-stage CI/CD pipelines that integrate AI review, quality gates, and automated deployment. For core GitHub Actions concepts (workflows, jobs, steps, triggers), see [GitHub Actions Basics](github-actions-basics.md).

---

## The Full Pipeline

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  STAGE 1 │──▶│  STAGE 2 │──▶│  STAGE 3 │──▶│  STAGE 4 │──▶│  STAGE 5 │
│  Validate│   │  Test    │   │  Scan    │   │  Review  │   │  Deploy  │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
  Lint           Unit tests     SAST           AI review       Staging
  Format         Integration    DAST           Human review    Production
  Type check     E2E            SCA            Approval gate
  Build          Coverage
```

---

## Stage Design

### Stage 1: Validate (Fast — <2 min)
```yaml
validate:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - run: npm ci
    - run: npm run lint
    - run: npm run format:check
    - run: npm run type-check
    - run: npm run build
```
**Purpose**: Catch syntax/style issues immediately. Fails fast.

### Stage 2: Test (Medium — 2-10 min)
```yaml
test:
  needs: validate
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - run: npm ci
    - run: npm test -- --coverage
    - uses: actions/upload-artifact@v4
      with:
        name: coverage-report
        path: coverage/
```

### Stage 3: Security Scan (Medium — 2-10 min)
```yaml
security:
  needs: validate
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: github/codeql-action/init@v3
    - uses: github/codeql-action/analyze@v3
    - uses: semgrep/semgrep-action@v1
```

### Stage 4: Review (Variable — minutes to hours)
```yaml
review:
  needs: [test, security]
  # AI review happens automatically via Copilot rulesets
  # Human review is requested after all checks pass
```

### Stage 5: Deploy (Fast — <5 min)
```yaml
deploy:
  needs: review
  if: github.ref == 'refs/heads/main'
  runs-on: ubuntu-latest
  environment: production
  steps:
    - run: ./deploy.sh
```

---

## Parallelization

Run independent stages in parallel to minimize total pipeline time:

```
validate (2 min)
    ├── test (8 min)      ─┐
    └── security (5 min)  ─┼── review ── deploy
                           │
    Total: ~12 min (not 17 min sequential)
```

---

## Key Takeaways

1. **Fail fast** — Put cheap checks (lint, type-check) first
2. **Parallelize** — Run independent stages concurrently
3. **Gate between stages** — Don't scan if it doesn't build
4. **Deploy only from main** — Use `if:` conditions
5. **Keep total time under 15 minutes** — Faster feedback = better developer experience
