# Human-AI Review Balance

> What humans should review vs. what AI can check. The collaboration model: AI handles the first pass (catching 70-80% of mechanical issues), humans do the deep review (catching subtle issues AI misses).

---

## The Division of Labor

Not all review tasks are equal. Some are mechanical (consistent, pattern-based, automatable) and some require judgment (context-dependent, nuanced, creative). The key is assigning each to the right reviewer.

```
┌────────────────────────────────────┐
│         AI First Pass              │
│  Style, patterns, known vulns,     │
│  coverage gaps, documentation      │
│  ──────────────────────────────    │
│  Catches ~70-80% of mechanical     │
│  issues quickly and consistently   │
└──────────────┬─────────────────────┘
               │
               ▼
┌────────────────────────────────────┐
│         Human Deep Review          │
│  Business logic, architecture,     │
│  security implications, edge       │
│  cases, UX impact                  │
│  ──────────────────────────────    │
│  Catches subtle issues that        │
│  require context and judgment      │
└────────────────────────────────────┘
```

---

## What AI Is Good At Reviewing

### Style Consistency
AI excels at checking if code follows established patterns:
- Naming conventions match
- Import order is correct
- File structure follows template
- Comment format is consistent

✅ AI catches: `getUserData` vs team standard `findUser`
✅ AI catches: Missing JSDoc on a public function
✅ AI catches: Default export instead of named export

### Common Patterns
AI recognizes and flags deviations from standard patterns:
- Missing error handling on async operations
- Unvalidated user input
- Missing null checks
- Inconsistent response formats

### Known Vulnerability Patterns
AI can check for well-documented security issues:
- SQL injection (string concatenation in queries)
- XSS (unsanitized output)
- Hardcoded credentials
- Insecure random number generation
- Missing authentication checks on routes

### Test Coverage Gaps
AI can identify untested code paths:
- Functions without corresponding tests
- Missing edge case tests
- Error paths not tested
- Boundary conditions not covered

### Documentation Completeness
AI can verify documentation exists and is structured:
- Public functions have JSDoc/docstrings
- README sections are complete
- API documentation matches implementation
- Changelog is updated

---

## Where Humans Are Essential

### Business Logic Correctness
Only humans understand the business context:

```typescript
// AI sees: valid code that calculates a discount
function calculateDiscount(order: Order): number {
  if (order.total > 100) return order.total * 0.1;
  return 0;
}

// Human knows: "We changed the discount policy last week. 
// It should be 15% for orders over $100, and 5% for 
// loyalty members regardless of order size."
```

✅ Human catches: Outdated business rules
✅ Human catches: Missing loyalty member discount path
❌ AI misses: It doesn't know the current business policy

### Architectural Decisions
Code placement and design choices require system-level understanding:

```
AI review: "This code is correct and follows patterns."
Human review: "This logic belongs in the service layer, 
not the route handler. It also duplicates what's in 
OrderService.calculateTotal()."
```

✅ Human catches: Wrong layer, duplication, design violations
❌ AI misses: It evaluates code in isolation, not in system context

### Security Implications
Beyond known patterns, security requires threat modeling:

```typescript
// AI sees: valid code with input validation
app.get('/api/documents/:id', async (req, res) => {
  const doc = await documentRepo.findById(req.params.id);
  if (!doc) return res.status(404).json({ error: 'Not found' });
  return res.json(doc);
});

// Human sees: "No authorization check. Any authenticated 
// user can access any document by ID. This is an 
// Insecure Direct Object Reference (IDOR) vulnerability."
```

✅ Human catches: Authorization logic, IDOR, privilege escalation
❌ AI misses: Context-dependent security issues

### Edge Case Reasoning
Humans understand real-world scenarios:

```typescript
// AI sees: function handles null input
function processPayment(amount: number): Result<Payment, Error> {
  if (amount <= 0) return err(new Error('Invalid amount'));
  // process payment...
}

// Human asks: "What about:
// - Currency conversion rounding ($0.001)?
// - Maximum transaction limits?
// - Duplicate payment detection?
// - Partial refund scenarios?
// - Network timeout during processing?"
```

✅ Human catches: Real-world edge cases from domain experience
❌ AI misses: Domain-specific scenarios not in the code

### UX Impact
Code changes affect user experience in ways that require human judgment:

```
AI review: "Loading state implemented correctly."
Human review: "This shows a spinner for 3 seconds on every 
page navigation. We should use skeleton loading for a 
better perceived performance."
```

---

## The Responsibility Matrix

| Review Area | AI First Pass | Human Deep Review |
|---|---|---|
| **Style & formatting** | ✅ Primary | 🔲 Skip |
| **Naming conventions** | ✅ Primary | 🔲 Skip |
| **Known vulnerability patterns** | ✅ Primary | ✅ Verify critical areas |
| **Test existence** | ✅ Primary | 🔲 Skip |
| **Test quality** | ⚠️ Partial | ✅ Primary |
| **Documentation completeness** | ✅ Primary | 🔲 Skip |
| **Business logic correctness** | ❌ Cannot | ✅ Primary |
| **Architectural fit** | ❌ Cannot | ✅ Primary |
| **Security implications** | ⚠️ Partial | ✅ Primary |
| **Edge case coverage** | ⚠️ Partial | ✅ Primary |
| **Performance implications** | ⚠️ Partial | ✅ Primary |
| **UX impact** | ❌ Cannot | ✅ Primary |
| **Cross-system integration** | ❌ Cannot | ✅ Primary |

**Legend:** ✅ Primary reviewer | ⚠️ Can partially help | ❌ Cannot assess | 🔲 Not needed

---

## The Collaboration Model in Practice

### Step 1: AI Pre-Review

Set up automated AI review on PR creation:

```markdown
AI Review Prompt:
"Review this PR for:
1. Style consistency with project conventions
2. Missing error handling
3. Known security vulnerability patterns
4. Test coverage gaps
5. Documentation completeness

Do NOT comment on:
- Business logic decisions
- Architectural choices
- Performance optimization suggestions

Only flag issues you are confident about."
```

### Step 2: Developer Addresses AI Findings

Developer fixes mechanical issues flagged by AI before human review starts. This saves human reviewer time.

### Step 3: Human Focused Review

Human reviewer now focuses exclusively on high-judgment areas:

```markdown
Human Review Focus Areas:
□ Does this code implement the correct business logic?
□ Does it fit the system architecture?
□ Are there security implications beyond common patterns?
□ What edge cases are missing?
□ What's the performance impact?
□ How does this affect the user experience?
```

### Step 4: Decision

Human reviewer has the final say. AI findings are advisory.

---

## What Should Never Be Fully Automated

Even with excellent AI review, these decisions require human judgment:

| Decision | Why Human Required |
|---|---|
| **Merging to main** | Humans must own production readiness |
| **Deploying to production** | Business risk assessment needed |
| **Architecture decisions** | System-level understanding required |
| **Dependency additions** | License, security, maintenance implications |
| **API contract changes** | Consumer impact assessment needed |
| **Data model changes** | Migration risk, backward compatibility |
| **Security-sensitive changes** | Threat modeling requires context |

---

## Measuring the Balance

Track these metrics to ensure the human-AI balance is working:

| Metric | Healthy Range | Action if Unhealthy |
|---|---|---|
| AI pre-review finding rate | 3-10 findings/PR | Too low: AI checks too lenient; Too high: standards unclear |
| Human-only findings | 1-3 per PR | Zero: review too shallow; 5+: AI checks insufficient |
| Review cycle time | Decreasing | If increasing: AI creating more work than saving |
| Post-merge bugs | Decreasing | If increasing: review depth insufficient |
| Reviewer satisfaction | Stable/improving | If declining: review burden too high |

## Next Steps

- [Create PR templates for transparency →](pr-templates.md)
- [Use detailed review checklists →](review-checklists.md)
- [Return to the full review workflow →](ai-code-review-workflow.md)
- [Set quality standards for AI output →](../03-standards-and-conventions/code-quality-standards.md)
