# Multi-Repo Review

> Cross-repository code review and governance for microservices and monorepo-adjacent architectures.

---

## When You Need Multi-Repo Review

- **Microservices**: API contract changes affect multiple repos
- **Shared libraries**: Breaking changes propagate downstream
- **Platform teams**: Infrastructure changes affect all consumers
- **Compliance**: Consistent security standards across repos

---

## Cross-Repo Analysis Patterns

### 1. API Contract Validation
When an API definition changes in repo A, validate consumers in repos B, C, D:

```yaml
# In API repo's CI
name: Contract Validation
on:
  pull_request:
    paths: ['api/openapi.yaml']
jobs:
  validate-consumers:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        consumer: [frontend, mobile-app, partner-api]
    steps:
      - uses: actions/checkout@v4
        with:
          repository: org/${{ matrix.consumer }}
      - name: Validate contract compatibility
        run: npx openapi-diff old-spec.yaml new-spec.yaml
```

### 2. Shared Library Impact Analysis
When a shared library PR is opened, analyze which repos depend on it:

```yaml
# Trigger downstream testing
name: Impact Analysis
on:
  pull_request:
    branches: [main]
steps:
  - name: Find dependents
    run: |
      # Query GitHub dependency graph API
      gh api /repos/org/shared-lib/dependents
  - name: Trigger dependent repo tests
    run: |
      for repo in $DEPENDENTS; do
        gh workflow run tests.yml -R org/$repo
      done
```

### 3. Org-Wide Security Scanning
Central workflow that scans all repos on schedule:

```yaml
name: Org Security Audit
on:
  schedule:
    - cron: '0 2 * * 1'  # Weekly Monday 2am
jobs:
  scan:
    strategy:
      matrix:
        repo: [api, frontend, worker, shared-lib]
    steps:
      - uses: actions/checkout@v4
        with:
          repository: org/${{ matrix.repo }}
      - name: Run CodeQL
        uses: github/codeql-action/analyze@v3
```

---

## Governance Across Repos

| Level | Tool | What It Enforces |
|-------|------|-----------------|
| **Organization** | Org rulesets | Branch protection, required reviews |
| **Repository** | Repo rulesets | Specific status checks per repo |
| **Code owners** | CODEOWNERS file | Per-path approval requirements |
| **Templates** | Template repos | Standard CI/CD configuration |

---

## Key Takeaways

1. **API contracts need cross-repo validation** — Changes aren't isolated
2. **Shared libraries need impact analysis** — Know who depends on you
3. **Org-level scanning is essential** — Consistent security posture
4. **Template repos standardize CI/CD** — New repos start correctly
5. **Automate dependency awareness** — GitHub Dependency Graph + Dependabot
