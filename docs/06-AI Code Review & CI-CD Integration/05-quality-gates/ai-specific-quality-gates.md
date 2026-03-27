# AI-Specific Quality Gates

> Quality gates designed to catch the unique risks of AI-generated code.

---

## Why AI Code Needs Special Gates

AI-generated code has specific failure modes that traditional gates don't catch:

| AI Risk | Traditional Gate? | Needs Special Gate |
|---------|------------------|-------------------|
| Looks correct but wrong logic | ❌ Tests may pass | ✅ AI review + manual review |
| Uses deprecated APIs | ⚠️ Sometimes | ✅ Deprecation scanner |
| Duplicates existing code | ⚠️ Duplication check | ✅ Semantic duplication |
| Hallucinated imports/APIs | ❌ Build may catch | ✅ Import validation |
| Insecure patterns from training data | ⚠️ SAST catches some | ✅ AI-specific security scan |
| Missing error handling | ❌ Not detected | ✅ Error handling check |
| Overly verbose/complex | ⚠️ Complexity check | ✅ AI review for simplification |

---

## Recommended AI-Specific Gates

### 1. Import Validation Gate
AI can hallucinate packages that don't exist:
```yaml
- name: Validate imports
  run: |
    npm install --dry-run 2>&1 | grep -i "not found" && exit 1 || exit 0
```

### 2. Deprecation Gate
AI training data may include deprecated APIs:
```yaml
- name: Check for deprecated APIs
  run: npx eslint src/ --rule '{"no-restricted-syntax": "error"}'
```

### 3. Error Handling Gate
AI often generates happy-path-only code:
```yaml
- name: Check error handling
  run: |
    # Custom script that checks all async functions have try/catch
    node scripts/check-error-handling.js
```

### 4. AI Review Gate
Dedicated AI review as a required status check:
```yaml
# Branch protection rule:
# Required status checks: [copilot-review, lint, test, security]
```

### 5. Semantic Duplication Gate
AI may regenerate existing utilities instead of importing them:
```yaml
- name: Check semantic duplication
  run: npx jscpd src/ --min-tokens 50 --threshold 3
```

---

## Defense-in-Depth for AI Code

```
Layer 1: Type checking        → Catches hallucinated types
Layer 2: Lint + format        → Catches style issues  
Layer 3: SAST                 → Catches security patterns from training data
Layer 4: Test + coverage      → Catches logic errors
Layer 5: AI review            → Catches context-aware issues
Layer 6: Human review         → Catches business logic, architecture
```

---

## Key Takeaways

1. **AI code has unique failure modes** — Traditional gates are necessary but not sufficient
2. **Import validation** — AI hallucinates packages; verify they exist
3. **Deprecation scanning** — AI uses outdated APIs from training data
4. **Error handling checks** — AI generates happy-path code; enforce error handling
5. **Defense-in-depth** — Layer all gates for comprehensive coverage

---

## Next Steps

- 🔗 [Testing Strategies](../06-testing-ai-code/testing-strategies.md) — Testing AI-generated code
- 🔗 [Validation Framework](../06-testing-ai-code/validation-framework.md) — Structured validation
