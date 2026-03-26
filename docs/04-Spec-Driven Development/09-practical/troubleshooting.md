# SDD Troubleshooting Guide

> When spec-driven development isn't working — diagnosis and fixes.

---

## Problem: AI Ignores the Spec

### Symptoms
- AI generates code that doesn't match acceptance criteria
- AI implements features not in the spec
- AI uses different patterns than the plan specified

### Diagnosis
```
□ Is the spec loaded in the AI's context?
□ Are you referencing the spec explicitly in prompts?
□ Is the spec too long for the context window?
□ Are acceptance criteria specific enough?
```

### Fixes
1. **Reference explicitly**: "Implement per AC-2.1 in spec.md: [paste criterion]"
2. **Reduce context**: Load only the relevant spec section, not the entire spec
3. **Be specific**: Replace "follow the spec" with exact criteria
4. **Validate**: After each task, ask AI to check against specific ACs

---

## Problem: Specs Take Too Long to Write

### Symptoms
- Team avoids writing specs because "it slows us down"
- Specs take days to get through review
- More time spent specifying than implementing

### Diagnosis
```
□ Are you over-specifying? (See over-specification guide)
□ Is the review process too heavy?
□ Are you specifying before you understand the problem?
□ Are templates too complex?
```

### Fixes
1. **Right-size**: Small features need 10-20 lines, not 100
2. **Start with Template 5** (lightweight): What, Why, 3 ACs, 2 edge cases
3. **Time-box**: Max 30 min for medium features, 2 hours for large
4. **Prototype first**: Vibe-code to explore, then spec for production
5. **Reduce review ceremony**: 1 reviewer for small specs, 2 for large

---

## Problem: Specs Become Outdated Quickly

### Symptoms
- Developers don't read specs because they're stale
- PRs change behavior without spec updates
- Quarterly audit reveals widespread drift

### Fixes
1. **Spec-first updates**: Update spec in same PR as code changes
2. **CI warning**: Automated check when src/ changes without spec/ changes
3. **Link code to specs**: `@spec` annotations in code
4. **Quarterly audit**: 30-minute team review of spec accuracy

---

## Problem: Team Resistance to SDD

### Symptoms
- "This is just bureaucracy"
- Developers write minimal specs to satisfy the process
- Specs are copy-paste jobs with no real thought

### Fixes
1. **Demonstrate value**: Show a before/after example where a spec prevented rework
2. **Start small**: Only spec complex features initially
3. **Make it easy**: Provide templates and AI-assisted drafting
4. **Make it part of the workflow**: Not an extra step, but how work starts
5. **Celebrate wins**: "This spec saved us 2 days of rework"

---

## Problem: Quality Gate Bottlenecks

### Symptoms
- Work stalls waiting for spec reviews
- Plan reviews take a week to schedule
- Nobody has time to review specs properly

### Fixes
1. **Async reviews**: Use PR reviews, not meetings
2. **SLA**: Spec reviews completed within 1 business day
3. **Lightweight gates**: Self-checklist for small features
4. **Pair-speccing**: Write specs with a peer (review built-in)

---

## Problem: AI Generates Wrong Plans

### Symptoms
- Plan uses technology not in your stack
- Architecture is over-engineered
- Plan doesn't integrate with existing code

### Fixes
1. **Include constitution**: AI must know your tech stack
2. **Provide codebase context**: "We use Prisma for DB access (see schema.prisma)"
3. **Review before proceeding**: Don't skip the Plan quality gate
4. **Be specific about constraints**: "Must use existing UserService pattern"

---

## Problem: Task Decomposition Is Wrong

### Symptoms
- Tasks are too large (multi-day tasks)
- Tasks are in wrong order
- Dependencies are missing or incorrect
- Tasks don't cover all plan items

### Fixes
1. **Size check**: Each task should be 1-2 hours of work
2. **Coverage check**: Map every plan section to at least one task
3. **Dependency validation**: Walk through tasks in order mentally
4. **Include tests**: Every implementation task should have a corresponding test task

---

## Problem: Spec-Implementation Mismatch After Completion

### Symptoms
- Feature shipped but doesn't match some acceptance criteria
- Undocumented deviations from the spec
- QA finds issues when testing against spec

### Fixes
1. **Per-task validation**: Check ACs after each task, not just at the end
2. **Verification checklist**: Walk through every AC before marking done
3. **Automated acceptance tests**: Write tests derived from ACs
4. **Spec review in PR**: Reviewer checks code against spec, not just code quality

---

## Quick Diagnosis Flowchart

```
Is SDD working for your team?
│
├─ NO → What's the main problem?
│       │
│       ├─ "Too slow"
│       │   → Right-size specs (Template 5 for small features)
│       │   → Time-box specification (30 min max for medium)
│       │
│       ├─ "Specs are useless/stale"
│       │   → Require spec updates in PRs
│       │   → Add CI drift detection
│       │
│       ├─ "AI doesn't follow specs"
│       │   → Reference ACs explicitly in prompts
│       │   → Load minimal relevant context
│       │
│       ├─ "Team won't adopt it"
│       │   → Start with complex features only
│       │   → Show ROI with before/after examples
│       │
│       └─ "Quality gates are bottlenecks"
│           → Use async PR reviews
│           → Self-checklist for small features
│
└─ YES → Review and optimize
        → Track metrics (coverage, rework, accuracy)
        → Expand to more feature types
        → Refine templates based on experience
```

---

*Next: [FAQ →](faq.md)*
