# AI Test Generation

> Using AI to write tests — when it works, when it doesn't, and how to validate AI-generated tests.

---

## How AI Generates Tests

AI can generate tests from:
1. **Source code** — "Write tests for this function"
2. **Specifications** — "Write tests for this requirement"
3. **Examples** — "Write tests similar to these existing tests"
4. **Error reports** — "Write a regression test for this bug"

---

## When AI Test Generation Works Well

| Scenario | Quality | Why |
|----------|---------|-----|
| **Unit tests for pure functions** | ✅ High | Clear inputs/outputs |
| **CRUD API endpoint tests** | ✅ High | Standard patterns |
| **Snapshot tests** | ✅ High | Deterministic comparison |
| **Boilerplate test setup** | ✅ High | Repetitive, pattern-based |
| **Edge case expansion** | ⚠️ Medium | AI finds some, misses others |
| **Complex integration tests** | ⚠️ Medium | Needs more context |
| **Behavior-driven tests** | ❌ Low | AI often tests implementation, not behavior |
| **Performance/load tests** | ❌ Low | Requires domain-specific knowledge |

---

## Validating AI-Generated Tests

### The "Test Your Tests" Checklist

1. **Does each test have meaningful assertions?**
   ```javascript
   // ❌ Useless — tests nothing
   expect(result).toBeDefined();
   
   // ✅ Meaningful — tests actual behavior
   expect(result.total).toBe(42);
   expect(result.items).toHaveLength(3);
   ```

2. **Does the test fail when the code is wrong?**
   - Run mutation testing: change the code and verify tests fail
   ```bash
   npx stryker run  # JavaScript mutation testing
   ```

3. **Does the test cover edge cases?**
   - Empty inputs, null values, boundary conditions
   - Maximum sizes, special characters, concurrent access

4. **Does the test describe behavior, not implementation?**
   ```javascript
   // ❌ Tests implementation (fragile)
   expect(service.cache.has('key')).toBe(true);
   
   // ✅ Tests behavior (robust)
   expect(await service.get('key')).toBe('value');
   ```

---

## Best Practices

| Practice | Why |
|----------|-----|
| **Review AI tests as carefully as AI code** | Weak tests give false confidence |
| **Run mutation testing** | Verify tests actually catch bugs |
| **Use AI for boilerplate, humans for assertions** | AI sets up tests, humans define expectations |
| **Generate tests from specs, not code** | Spec-based tests verify requirements |
| **Include negative tests** | AI tends to generate happy-path only |

---

## Key Takeaways

1. **AI is good at test boilerplate** — Setup, teardown, standard patterns
2. **AI is weak at meaningful assertions** — Review every `expect()` statement
3. **Mutation testing validates AI tests** — If mutating code doesn't fail tests, tests are weak
4. **Spec-driven test generation is safest** — Tests from requirements, not from code
5. **Always add negative tests** — AI defaults to happy paths

---

## Next Steps

- 🔗 [Validation Framework](validation-framework.md) — Structured validation pipeline
- 🔗 [Metrics & Benchmarks](metrics-and-benchmarks.md) — Measuring quality
