# Over-Specification

> When too much detail becomes the enemy of progress.

---

## What Is Over-Specification?

Over-specification happens when specs try to eliminate all uncertainty upfront — describing every possible scenario, prescribing exact implementations, and documenting trivial decisions.

```
Normal Spec:     50-100 lines  → Clear enough to implement
Over-Specified:  500+ lines    → Nobody reads it, nobody follows it
```

---

## The Symptoms

### Symptom 1: The Spec Is Longer Than the Code
If your spec for a user registration form is 400 lines and the implementation is 200 lines, you're over-specifying.

### Symptom 2: Specifying the Obvious
```markdown
❌  "The submit button shall be clickable"
❌  "The form shall accept keyboard input"
❌  "The page shall load when the user navigates to the URL"
```

### Symptom 3: Implementation Masquerading as Spec
```markdown
❌  "Use a 12-column CSS grid with 16px gutters.
     The submit button shall be a 40px tall rounded rectangle
     with background-color #1a73e8 and text-color #ffffff,
     positioned 24px below the last form field."
```

### Symptom 4: Edge Case Exhaustion
Listing every conceivable edge case, including ones with near-zero probability:
```markdown
❌  "What if the user enters exactly 2,147,483,647 characters?"
❌  "What if the database server is hit by a meteorite?"
❌  "What if the user submits the form while their computer is on fire?"
```

### Symptom 5: Analysis Paralysis
The team spends more time debating the spec than it would take to build the feature.

---

## Why Over-Specification Happens

| Cause | Psychology |
|-------|-----------|
| Fear of ambiguity | "If we don't specify it, AI will get it wrong" |
| Mistrust of implementer | "Developers can't be trusted to make good decisions" |
| Previous failures | "Last time we under-specified and it went badly" |
| Perfectionism | "This spec should be complete and perfect" |
| Compliance anxiety | "What if auditors ask why this wasn't specified?" |

---

## The Real Costs

### 1. Context Window Waste
AI agents have limited context. An over-specified spec fills the context window with trivial details, leaving less room for reasoning:
```
16K context window:
  Over-specified: 8K spec + 4K code + 4K reasoning  (constrained)
  Right-sized:    2K spec + 4K code + 10K reasoning  (plenty of room)
```

### 2. Throughput Reduction
Research shows exhaustive specs can slow throughput 5-10x compared to right-sized iterative approaches:
```
Right-sized spec:   30 min spec → 2 hours implementation = 2.5 hours total
Over-specified:     4 hours spec → 2 hours implementation = 6 hours total
                    (and the extra 3.5 hours of spec detail didn't improve quality)
```

### 3. Maintenance Burden
Every line of spec is a line that must be maintained. More lines = more drift risk.

### 4. Review Fatigue
Nobody carefully reviews a 400-line spec. Reviewers skim, miss issues, and rubber-stamp.

---

## How to Right-Size Specs

### The "Would a Senior Dev Need This?" Test
For each spec statement, ask: "Would a competent developer need this information?"
```
"Sessions expire after 30 minutes" → Yes, this is a business rule ✅
"The login form has an email field" → Obviously needed for login, skip ❌
"The API returns JSON" → Obvious from the project stack, skip ❌
"Rate limit: 5 login attempts per 15 minutes" → Not obvious, include ✅
```

### The Priority Filter
Categorize requirements by importance:

```
Must Specify:  Business rules, acceptance criteria, security constraints
Should Specify: Edge cases for likely failures, performance SLAs
Can Skip:       Implementation details, obvious behavior, UI details
                (unless it's a UI-specific spec)
```

### The "One More Line" Test
For each line you're about to add, ask: "Does this line prevent a bug or misunderstanding?"
- If yes → keep it
- If no → remove it
- If maybe → keep it but mark as "nice to have"

---

## The Spec Size Budget

| Feature Size | Max Spec Lines | Spend Time On |
|-------------|----------------|---------------|
| Small | 20 lines | Acceptance criteria only |
| Medium | 80 lines | + Edge cases + NFRs |
| Large | 150 lines | + Data model + API contracts |
| Epic | Split into multiple specs | Each sub-spec ≤ 150 lines |

### The 80/20 Rule for Specs
80% of spec value comes from 20% of the content:
- Summary (what and why)
- Acceptance criteria (what success looks like)
- Edge cases (what can go wrong)
- Scope boundaries (what's not included)

Everything else is supporting detail.

---

## Recovering From Over-Specification

If you've already over-specified:

### 1. Extract the Essential Spec
Create a condensed version with only the critical information. Keep the full spec as an appendix.

### 2. Use the "Executive Summary" Pattern
```markdown
## Quick Spec (read this first)
[20-line summary with key ACs and scope]

## Full Details (reference as needed)
[Remaining details organized by section]
```

### 3. Establish Size Guidelines
Set team-wide limits on spec length. Make it a review criterion.
