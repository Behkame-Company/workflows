# Spec-Code Drift

> When specifications and code diverge — causes, detection, and remediation.

---

## What Is Spec-Code Drift?

Spec-code drift occurs when the **specification says one thing** but the **code does another**. It's the SDD equivalent of documentation rot.

```
Initial State:
  Spec: "Sessions expire after 30 minutes" ───── Code: timeout = 30 * 60
  
After Drift:
  Spec: "Sessions expire after 30 minutes" ───── Code: timeout = 60 * 60
  (Spec still says 30 min, but someone changed code to 60 min)
```

---

## Why Drift Is Dangerous

### For Humans
- New team members read the spec and build wrong mental models
- QA tests against the spec, passes, but code behaves differently
- Stakeholders expect spec behavior but get something else

### For AI Agents
- AI uses the stale spec as context → generates code matching the WRONG behavior
- AI "fixes" drift by reverting code to match the stale spec
- Drift compounds as AI builds on top of incorrect assumptions

---

## Common Causes of Drift

### Cause 1: Hotfixes Without Spec Updates
```
Production bug → Emergency fix → Code changed → Nobody updates spec
```

### Cause 2: "Quick" Changes
```
"Just change the timeout to 60 minutes"
→ Developer changes code
→ PR merged without spec reference
→ Spec still says 30 minutes
```

### Cause 3: Iterative Implementation Divergence
```
During implementation:
  "The spec says X but I found Y works better"
  → Developer implements Y
  → Doesn't update spec to reflect Y
  → Spec still says X
```

### Cause 4: Multi-Team Ownership
```
Team A owns the spec
Team B implements the code
Team B makes changes
Team A doesn't know
```

### Cause 5: Time Pressure
```
"We'll update the spec later"
→ "Later" never comes
→ Drift accumulates
```

---

## Detecting Drift

### Manual Detection

#### The Spec Walk-Through
Quarterly, walk through each spec section and verify against code:

```
Spec §1.1: "Login requires email and password"
Code check: ✅ LoginForm component requires both fields

Spec §1.2: "Sessions expire after 30 minutes"
Code check: ❌ AUTH_TIMEOUT = 3600000 (60 minutes)
→ DRIFT DETECTED
```

#### The Test Derivation Check
For each acceptance criterion, verify a matching test exists:

```
AC-1: "User sees error for invalid email"     → test/auth.test.ts:45 ✅
AC-2: "Account locks after 5 failed attempts"  → No matching test ⚠️
AC-3: "Session extends on activity"            → test/session.test.ts:89 ✅
```

### Automated Detection

#### Approach 1: Spec-Linked Constants
```typescript
// Spec: .specify/specs/auth/spec.md §2.3
// AC: "Sessions expire after 30 minutes of inactivity"
export const SESSION_TIMEOUT_MS = 30 * 60 * 1000; // 30 minutes
```

A script can verify the constant matches the spec value.

#### Approach 2: Acceptance Test Tags
```typescript
// @spec .specify/specs/auth/spec.md#AC-2.3
test('session expires after 30 minutes', () => {
  expect(AUTH_CONFIG.sessionTimeout).toBe(30 * 60 * 1000);
});
```

If the test fails, either the code or the spec has drifted.

#### Approach 3: CI/CD Spec Validation
```yaml
steps:
  - name: Check spec coverage
    run: |
      # Extract all AC- references from specs
      # Check each has a corresponding test
      # Report missing coverage
```

---

## Remediating Drift

When drift is detected, determine the source of truth:

### Decision: Is the Code or Spec Correct?

| Scenario | Action |
|----------|--------|
| Code was changed intentionally for good reason | Update spec to match code |
| Code was changed accidentally or carelessly | Fix code to match spec |
| Both are wrong — requirements evolved | Update both to match current intent |
| Unknown which is correct | Ask stakeholders for current intent |

### Remediation Process
```
1. Document the drift: "Spec says X, code does Y"
2. Determine which is correct (ask stakeholders if needed)
3. Create a PR that updates the wrong artifact
4. Update tests to match the corrected behavior
5. Add a drift-prevention measure (test, CI check)
```

---

## Prevention Strategies

| Strategy | Effort | Effectiveness |
|----------|--------|--------------|
| Include spec updates in PRs | Low | High |
| Acceptance tests linked to spec | Medium | Very High |
| Quarterly spec audits | Low | Medium |
| Automated drift detection in CI | Medium | High |
| Culture of "spec first, code second" | High | Very High |

---

## The Cost of Drift

| Drift Duration | Cost |
|---------------|------|
| Same PR | Free (catch in review) |
| Same sprint | 1 hour (locate and fix) |
| Same quarter | 1 day (investigate, fix, re-test) |
| 6+ months | Days to weeks (archaeology required) |
| Never caught | Unknown behavior in production |

**Key insight**: The cost of fixing drift grows exponentially with time.
