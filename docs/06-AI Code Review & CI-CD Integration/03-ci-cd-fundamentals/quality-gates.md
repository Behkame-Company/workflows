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
| **Lint / Type Check** | Style, formatting, type safety | Zero errors |
| **Tests / Coverage** | Functionality, code coverage | All pass, ≥80% new code |
| **Security (SAST)** | Vulnerabilities, CVEs | Zero critical/high findings |
| **AI Review** | Code quality | No critical AI findings |
| **Build / Performance** | Compilation, response times | Successful build, no regression >10% |

> For detailed implementation of each gate type, see the [Quality Gates section](../05-quality-gates/). For CI/CD implementation patterns, see [GitHub Actions Basics](github-actions-basics.md) and [Pipeline Architecture](pipeline-architecture.md).

---

## Gate Severity Levels

| Severity | Action | Example |
|----------|--------|---------|
| 🔴 Critical / 🟠 High | **Block merge** | Security vuln, data loss, significant bug |
| 🟡 Medium | **Warn** | Code smell, minor improvement |
| 🟢 Low | **Inform** | Style preference, optional enhancement |

---

## Key Takeaways

1. **Gates are binary** — Pass or fail, no ambiguity
2. **Start strict on critical** — Block on security and critical bugs
3. **Warn on medium** — Don't block the pipeline for code smells
4. **Enforce via branch protection** — Make gates mandatory for merge
5. **Review gate thresholds regularly** — Adjust as team matures
