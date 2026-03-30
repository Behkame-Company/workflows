# Code Quality Gates

> Automated checkpoints that enforce coding standards, complexity limits, and quality metrics.

---

## Types of Code Quality Gates

### 1. Style Gates
Enforce consistent code formatting:
```yaml
- name: Check formatting
  run: npx prettier --check "src/**/*.{ts,tsx}"
  # Fails if any file isn't formatted
```

### 2. Lint Gates
Catch code smells and potential bugs:
```yaml
- name: Lint
  run: npx eslint src/ --max-warnings 0
  # Fails if any warnings or errors
```

### 3. Complexity Gates
Prevent overly complex code:
```yaml
- name: Check complexity
  run: npx eslint src/ --rule '{"complexity": ["error", 10]}'
  # Fails if any function has cyclomatic complexity > 10
```

### 4. Duplication Gates
Detect copy-paste code:
```yaml
- name: Check duplication
  run: npx jscpd src/ --threshold 5
  # Fails if >5% code duplication
```

### 5. Bundle Size Gates
Prevent size regressions:
```yaml
- name: Check bundle size
  run: npx bundlesize
  # Fails if bundle exceeds configured limits
```

---

## Gate Configuration Strategy

| Gate | New Code | Legacy Code | Rationale |
|------|----------|-------------|-----------|
| **Lint (errors)** | Block | Block | Errors are always unacceptable |
| **Lint (warnings)** | Block | Warn | New code should be clean |
| **Complexity** | Block (>10) | Warn (>15) | Gradually reduce legacy complexity |
| **Coverage** | Block (<80%) | Warn (<60%) | New code must be tested |
| **Duplication** | Block (>3%) | Warn (>10%) | Prevent new duplication |

### Ratchet Pattern
Gradually tighten thresholds over time:
```
Month 1: Coverage gate at 60%
Month 3: Coverage gate at 70%  
Month 6: Coverage gate at 80%
```

---

## Key Takeaways

1. **Gates are binary** — Pass or fail, no "soft" failures for critical checks
2. **Different thresholds for new vs legacy** — Don't block all work on tech debt
3. **Ratchet pattern** — Gradually tighten standards over time
4. **Automate everything** — No manual quality checks
5. **Block on critical, warn on nice-to-have** — Severity-based gating
