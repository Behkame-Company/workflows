# SDD Frequently Asked Questions

> 20 frequently asked questions about Spec-Driven Development.

---

## Getting Started

### Q1: Do I need Spec Kit to practice SDD?
**No.** Spec Kit is helpful but optional. You can practice SDD with plain Markdown files and any AI tool. The methodology matters more than the tooling.

### Q2: How do I start SDD if my team has never written specs?
Start with one complex feature. Write a lightweight spec (Template 5: What, Why, 3 ACs, 2 edge cases). Use it to guide AI implementation. Show the team the results. Expand from there.

### Q3: What's the minimum viable spec?
```markdown
**What**: [1 sentence]
**Success**: [3 testable criteria]
**Edge cases**: [2 likely failure scenarios]
```
This takes 5 minutes and prevents the most common AI implementation mistakes.

### Q4: Can AI write specs for me?
AI can **draft** specs. Humans must **review and refine**. AI specs are fluent but may hallucinate requirements, miss business context, or apply generic patterns. Always review.

---

## Process

### Q5: When should I NOT write a spec?
- Trivial bug fixes with obvious cause and solution
- Dependency updates
- Refactoring with no behavior change
- Style/formatting changes
- One-line configuration changes

### Q6: How long should writing a spec take?
| Feature Size | Max Spec Time |
|-------------|---------------|
| Small | 10 minutes |
| Medium | 30 minutes |
| Large | 2 hours |
| Epic | Decompose first |

If you're spending more time than this, you're probably over-specifying.

### Q7: Should specs be reviewed?
**Yes.** At minimum, one peer should review medium+ specs. For small features, a self-review checklist is sufficient. The review catches gaps that the author can't see.

### Q8: What if requirements change after spec approval?
Update the spec. That's expected and normal. SDD specs are living documents. Change the spec, assess impact on plan/tasks, and continue. Document the change with a version note.

### Q9: Who should write specs?
The person closest to the feature requirements. Usually the product engineer or tech lead. The author doesn't need to be the implementer.

---

## Technical

### Q10: Where should specs live?
In the repository, version-controlled alongside code. Default: `.specify/specs/<feature>/spec.md`. Alternative: `specs/` directory. Never in Google Docs, Confluence, or Slack.

### Q11: How do I handle specs for microservices?
Each service has its own `.specify/` directory. Cross-service features have a "coordination spec" in the primary service that references specs in other services.

### Q12: Can I use SDD with an existing codebase?
Yes. Write specs for new features going forward. For existing features, create retroactive specs when they need changes. You don't need to spec everything at once.

### Q13: How does SDD work with agile sprints?
Specs are written just-in-time for the next sprint's features. Don't spec features that are months away. A spec should describe work completable within 1-2 sprints.

### Q14: How do I handle SDD in a monorepo?
Use path-scoped specs:
```
packages/
  auth/
    .specify/specs/      # Auth-specific specs
  dashboard/
    .specify/specs/      # Dashboard-specific specs
.specify/
  constitution.md        # Shared constitution
```

---

## AI Integration

### Q15: How do I get AI to follow specs better?
1. Paste the specific acceptance criteria (not just "follow the spec")
2. Ask AI to validate each criterion after implementation
3. Load only relevant spec sections (not the entire spec)
4. Use structured formats (tables, Given/When/Then) that AI processes well

### Q16: What if the AI's plan contradicts the spec?
The spec wins. AI plans are proposals for human review. If the plan doesn't address a spec requirement, either:
- Fix the plan to address the requirement, or
- Update the spec if the requirement is genuinely wrong

### Q17: Can I use SDD with non-AI (manual) development?
Absolutely. SDD is valuable even without AI. Human developers benefit from clear specs just as much as AI agents do. The methodology predates AI coding tools.

---

## Common Concerns

### Q18: Isn't SDD just waterfall?
No — SDD specs are per-feature, living documents with days-long cycles and backward feedback loops. See [SDD vs Vibe Coding](../01-fundamentals/sdd-vs-vibe-coding.md) and [Waterfall in Disguise](../05-anti-patterns/waterfall-in-disguise.md) for detailed comparisons.

### Q19: What metrics should I track?
Start with these four:
1. **Spec coverage**: % of features with specs (target: >80%)
2. **Rework rate**: % of tasks requiring rework (target: <20%)
3. **Spec accuracy**: % of specs still matching code at review (target: >90%)
4. **Time-to-first-spec**: How long new hires take to write their first spec

### Q20: What's the future of SDD?
SDD is becoming the standard for AI-assisted development. Trends:
- Every major AI coding tool adding spec support
- Automated spec-to-test generation
- Real-time drift detection
- AI agents that can write, review, and maintain specs
- Convergence of SDD with TDD/BDD in unified workflows
