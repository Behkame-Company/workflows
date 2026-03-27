# Quality Gates

> Automated policy checkpoints that block code from progressing until it meets defined standards.

---

## What Are Quality Gates?

Quality gates are **automated pass/fail checkpoints** in your CI/CD pipeline. Code that doesn't meet the threshold is blocked from merging or deploying.

```
PR Created → Lint ✅ → Test ✅ → Security ❌ (BLOCKED) → Review → Deploy
                                    │
                                    └── "Critical vulnerability found.
                                         Fix before merge."
```

---

## Common Quality Gates

| Gate | What It Checks | Threshold Example |
|------|---------------|-------------------|
| **Lint** | Code style, formatting | Zero lint errors |
| **Type Check** | Type safety | Zero type errors |
| **Unit Tests** | Functionality | All tests pass |
| **Test Coverage** | Code coverage | ≥80% line coverage for new code |
| **Security (SAST)** | Vulnerabilities | Zero critical/high findings |
| **Dependency Check** | Known CVEs | No high-severity CVEs in deps |
| **AI Review** | Code quality | No critical AI findings |
| **Build** | Compilation | Successful build |
| **Performance** | Response times | No regression >10% |

---

## Implementing in GitHub Actions

### Branch Protection Rules

```
Repository → Settings → Branches → Branch protection rules

✅ Require status checks to pass before merging
  ✅ lint
  ✅ test
  ✅ security-scan
  ✅ copilot-review

✅ Require pull request reviews before merging
  - Required approvals: 1
  
✅ Require conversation resolution before merging
```

### Status Checks as Gates

```yaml
jobs:
  quality-gate:
    runs-on: ubuntu-latest
    needs: [lint, test, security, ai-review]
    steps:
      - name: All gates passed
        run: echo "All quality gates passed ✅"
```

---

## Gate Severity Levels

| Finding Severity | Gate Action | Rationale |
|-----------------|-------------|-----------|
| 🔴 Critical | **Block merge** | Security vulnerability, data loss risk |
| 🟠 High | **Block merge** | Significant bug, performance regression |
| 🟡 Medium | **Warn** (don't block) | Code smell, minor improvement |
| 🟢 Low | **Inform** only | Style preference, optional enhancement |

---

## Key Takeaways

1. **Gates are binary** — Pass or fail, no ambiguity
2. **Start strict on critical** — Block on security and critical bugs
3. **Warn on medium** — Don't block the pipeline for code smells
4. **Enforce via branch protection** — Make gates mandatory for merge
5. **Review gate thresholds regularly** — Adjust as team matures

---

## Next Steps

- 🔗 [Pipeline Architecture](pipeline-architecture.md) — Full pipeline design
- 🔗 [Security Gates](../05-quality-gates/security-gates-sast-dast.md) — SAST/DAST specifics
