# Security Gates (SAST/DAST)

> Integrating static and dynamic security testing into your CI/CD pipeline.

---

## SAST vs DAST

| | SAST | DAST |
|---|------|------|
| **What** | Analyzes source code | Tests running application |
| **When** | During CI (before deploy) | After deploy (staging) |
| **Finds** | SQL injection patterns, XSS, hardcoded secrets | Runtime vulnerabilities, auth flaws, config issues |
| **Speed** | Fast (seconds to minutes) | Slower (minutes to hours) |
| **False positives** | Higher | Lower |
| **Coverage** | All code paths (including dead code) | Only reachable paths |

**Use both**: SAST catches code-level issues; DAST catches runtime issues SAST can't see.

---

## SAST Integration

### CodeQL (GitHub Native)
```yaml
name: Security - SAST
on: [push, pull_request]
jobs:
  codeql:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with:
          languages: javascript, python
          queries: security-and-quality
      - uses: github/codeql-action/analyze@v3
```

### Semgrep
```yaml
  semgrep:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: semgrep/semgrep-action@v1
        with:
          config: >-
            p/owasp-top-ten
            p/secrets
            p/typescript
```

---

## DAST Integration

```yaml
name: Security - DAST
on:
  deployment_status:
    # Run after staging deployment
jobs:
  dast:
    if: github.event.deployment_status.state == 'success'
    runs-on: ubuntu-latest
    steps:
      - name: OWASP ZAP Scan
        uses: zaproxy/action-full-scan@v0.10.0
        with:
          target: ${{ github.event.deployment_status.target_url }}
          rules_file_name: '.zap-rules.tsv'
```

---

## Security Gate Thresholds

| Finding Severity | Action | Rationale |
|-----------------|--------|-----------|
| 🔴 Critical | **Block merge** | Active exploit risk |
| 🟠 High | **Block merge** | Significant vulnerability |
| 🟡 Medium | **Warn** | Track for remediation |
| 🟢 Low/Informational | **Log** | Reference only |

---

## SCA (Software Composition Analysis)

Don't forget dependency vulnerabilities:
```yaml
  dependency-check:
    steps:
      - uses: actions/checkout@v4
      - name: Dependabot / npm audit
        run: npm audit --audit-level=high
        # Fails if high-severity dependency vulnerabilities
```

---

## Key Takeaways

1. **SAST in CI, DAST in staging** — Different stages, complementary coverage
2. **Block on critical/high** — Zero tolerance for serious vulnerabilities
3. **CodeQL + Semgrep** — Combine for defense-in-depth
4. **Include SCA** — Dependency vulnerabilities are real attack vectors
5. **Tune regularly** — Reduce false positives to prevent alert fatigue

---

## Next Steps

- 🔗 [Test Coverage Gates](test-coverage-gates.md) — Ensuring adequate testing
- 🔗 [AI-Specific Quality Gates](ai-specific-quality-gates.md) — Catching AI-specific risks
