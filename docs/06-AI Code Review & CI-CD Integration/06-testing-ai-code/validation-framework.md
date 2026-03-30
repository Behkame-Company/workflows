# Validation Framework

> A structured pipeline for validating AI-generated code before it reaches production.

---

## The 6-Stage Validation Pipeline

```
Stage 1          Stage 2          Stage 3
PROMPT           GENERATION       STATIC
VALIDATION       REVIEW           ANALYSIS
                                  
"Is the          "Does the        "Is it safe
 prompt           code look        and clean?"
 complete?"       right?"         
                                  
    │                │                │
    ▼                ▼                ▼
Stage 4          Stage 5          Stage 6
TESTING          AI REVIEW        HUMAN
                                  REVIEW
                                  
"Does it          "What did       "Is this
 work?"           we miss?"       right for
                                   the system?"
```

---

## Stage Details

### Stage 1: Prompt Validation
Before AI generates code, ensure the prompt is complete:
- [ ] Requirements are specific and unambiguous
- [ ] Input/output types are defined
- [ ] Error handling expectations are stated
- [ ] Performance constraints are specified
- [ ] Existing patterns/utilities to use are referenced

### Stage 2: Generation Review
Read the AI-generated code immediately:
- [ ] Code solves the right problem
- [ ] Logic flow is correct
- [ ] No hallucinated imports or APIs
- [ ] Error handling is present
- [ ] Existing utilities are reused (not duplicated)

### Stage 3: Static Analysis
Run automated tools:
- [ ] Linter passes (zero errors)
- [ ] Type checker passes
- [ ] SAST scan passes (zero critical/high)
- [ ] Dependency check passes
- [ ] Build succeeds

### Stage 4: Testing
Verify functionality:
- [ ] Existing tests still pass
- [ ] New unit tests written and passing
- [ ] Integration tests verify boundaries
- [ ] Edge cases covered (null, empty, max values)
- [ ] Coverage threshold met (≥80% for new code)

### Stage 5: AI Review
Automated second opinion:
- [ ] AI reviewer analyzes the diff
- [ ] No critical findings
- [ ] Suggestions evaluated and addressed

### Stage 6: Human Review
Final human judgment:
- [ ] Business logic is correct
- [ ] Architecture is appropriate
- [ ] Code is maintainable
- [ ] Approved by required reviewers

---

## Validation Checklist (Quick Reference)

```
□ Prompt is complete and specific
□ Code compiles/builds
□ Linter passes
□ Type checks pass
□ SAST scan passes
□ All tests pass
□ Coverage ≥80% for new code
□ AI review — no critical findings
□ Human review — approved
□ Documentation updated
```

---

## Key Takeaways

1. **6 stages, each catches different issues** — No single stage is sufficient
2. **Start with prompt validation** — Garbage in, garbage out
3. **Automate stages 3-5** — Static analysis, testing, and AI review in CI/CD
4. **Human review is always last** — Final authority on merge decisions
5. **Use as a checklist** — Consistent process for every AI-generated change
