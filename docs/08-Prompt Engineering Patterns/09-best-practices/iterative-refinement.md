# Iterative Refinement

> Improving prompts through testing, observation, and systematic tuning.

---

## The Refinement Loop

```
Write prompt → Test → Observe → Diagnose → Refine → Test again
     ▲                                                    │
     └────────────────────────────────────────────────────┘
```

Prompts are rarely perfect on the first try. The best prompts are refined through multiple iterations.

---

## Step 1: Establish Baseline

Test your initial prompt with 3-5 representative inputs:

```
Prompt v1: "Review this code for bugs"

Test 1: Simple function → Output: Style comments (not bugs)
Test 2: Function with null bug → Output: Found the bug ✓
Test 3: Async function → Output: Missing error handling noted ✓
Test 4: Complex class → Output: Missed race condition ✗
```

**Baseline score**: 2/4 success. Note failure patterns.

---

## Step 2: Diagnose Failure Patterns

| Symptom | Likely Cause | Fix Direction |
|---------|-------------|---------------|
| Wrong format | No format specification | Add output format template |
| Misses edge cases | Not prompted to check | Add specific checks to instruction |
| Too verbose | No length constraint | Add "respond in X bullets/lines" |
| Off-topic comments | Scope too broad | Add "ONLY flag [specific things]" |
| Inconsistent quality | Works sometimes | Add examples (few-shot) |
| Hallucinated info | Insufficient context | Provide source files/data |

---

## Step 3: Targeted Fixes

Apply **one fix at a time** so you can measure its impact:

```
v1: "Review this code for bugs"
    Score: 2/4

v2: "Review this code for bugs. Only flag logic errors, null references,
     and unhandled errors. Do not comment on style."
    Score: 3/4 (Fixed: stopped style comments)

v3: Added: "For async code, also check: missing try-catch, unhandled 
     promise rejections, race conditions in shared state."
    Score: 4/4 (Fixed: catches async issues)
```

---

## Step 4: Regression Test

After each improvement, re-run ALL previous test cases. Fixing one failure shouldn't break another.

```
v3 results:
Test 1: ✓ (still correct — no style comments)
Test 2: ✓ (null bug found)
Test 3: ✓ (error handling noted)
Test 4: ✓ (race condition found) ← NEW PASS
```

---

## Prompt Versioning

Treat prompts like code. Version and track them:

```markdown
# review-bugs.prompt.md

## Version History
- v1: Basic "review for bugs" — 50% accuracy
- v2: Added scope constraint (logic errors only) — 75% accuracy
- v3: Added async-specific checks — 95% accuracy
- v4: Added few-shot example for race conditions — 98% accuracy
```

Or use git to track changes to your .prompt.md files.

---

## Common Refinement Patterns

| Starting Point | Typical Refinement Path |
|----------------|----------------------|
| Too broad | Add scope constraints → add format → add examples |
| Too narrow | Remove over-specific rules → generalize examples |
| Inconsistent | Add examples → add format template → add self-check |
| Wrong tone | Add role/persona → add communication style instructions |

---

## When to Stop Refining

- **95%+ success rate** on representative test cases
- **Diminishing returns** — last 3 changes made no measurable difference
- **Complexity budget exceeded** — prompt is > 500 words and hard to maintain
- **Good enough for the use case** — not every prompt needs perfection

---

## Next Steps

- 🔗 [Tool-Specific Prompting](tool-specific-prompting.md) — Tips for specific AI tools
- 🔗 [Clarity & Specificity](clarity-and-specificity.md) — Precision techniques
