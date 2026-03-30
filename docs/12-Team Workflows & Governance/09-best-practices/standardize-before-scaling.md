# Standardize Before Scaling

> The urge to scale AI tools across the organization as fast as possible is understandable — but premature scaling without standards creates fragmentation, inconsistency, and technical debt that's harder to fix than it was to prevent. Get one team right first.

---

## The Premature Scaling Trap

Here's what happens when an organization rolls out AI tools to all teams simultaneously without established standards:

```
Quarter 1: "Let's give everyone Copilot!"
  → Team A creates instruction files in .github/
  → Team B creates instruction files in docs/
  → Team C doesn't create instruction files at all
  → Team D copies Team A's file without understanding it

Quarter 2: "Why is everyone doing this differently?"
  → 4 different review processes for AI-generated code
  → 3 different prompt libraries in 3 different formats
  → No shared metrics — can't compare across teams
  → Security team finds 2 teams sharing code with unapproved tools

Quarter 3: "We need to standardize."
  → Teams resist changing their established practices
  → Political battles over whose approach becomes the standard
  → Significant rework to align existing practices
  → Morale drops as teams feel their work is being undone

Quarter 4: "This is harder than it needed to be."
```

✅ **Better approach:** One pilot team spends 8 weeks developing and refining practices, then those proven practices become the standard for the next teams.

❌ **Premature scaling:** All 20 teams start simultaneously, each developing independent practices that later need painful harmonization.

---

## The Pilot-First Framework

### Phase 1: Pilot Team (Weeks 1–8)

Select one team to serve as the pilot. This team will develop, test, and refine the practices that become organizational standards.

**Selecting the right pilot team:**

| Criteria | Why It Matters |
|---|---|
| **Willing, not forced** | Enthusiasm drives experimentation and honest feedback |
| **Representative tech stack** | Standards should apply to the majority of teams |
| **Moderate complexity** | Too simple won't test limits; too complex slows learning |
| **Strong team lead** | Someone who can champion and document practices |
| **Good existing practices** | Teams with solid fundamentals adopt AI tools more effectively |
| **Reasonable sprint cadence** | Room to experiment without constant deadline pressure |

**What the pilot team does:**
1. Adopt AI tools using the [gradual adoption strategy](./gradual-adoption.md)
2. Document everything — what works, what doesn't, what surprised them
3. Create the initial instruction files, review checklists, and prompt library
4. Measure impact using agreed-upon metrics
5. Present findings to the broader organization

✅ **Good pilot selection:** A mid-sized product team working on a mainstream feature with a team lead who has expressed interest in AI tools.

❌ **Bad pilot selection:** The CTO's pet project team, a team in crisis mode, or a team using a niche tech stack that doesn't represent the organization.

### Phase 2: Document What Works (Weeks 6–10)

While the pilot is still running, begin codifying their practices into reusable standards.

**Documentation deliverables:**

```
Standards Package:
├── instruction-file-template.md      # Starter template for new teams
├── review-checklist.md               # AI-aware code review checklist
├── onboarding-guide.md               # Step-by-step for new AI users
├── prompt-library/                   # Proven prompts organized by use case
│   ├── testing-prompts.md
│   ├── refactoring-prompts.md
│   ├── documentation-prompts.md
│   └── debugging-prompts.md
├── quality-gates.md                  # What AI code must pass before merge
├── tool-configuration.md             # Recommended settings and extensions
├── faq.md                            # Common questions and answers
└── case-studies/                     # Real examples from the pilot
    ├── case-study-testing.md
    ├── case-study-refactoring.md
    └── case-study-onboarding.md
```

✅ **Good documentation:** Includes specific examples from the pilot team's actual codebase — "Here's how we used AI to migrate our test suite, including the prompts we used and the review process we followed."

❌ **Bad documentation:** Generic advice copied from vendor marketing materials — "AI makes developers 55% more productive."

### Phase 3: Create Standards (Weeks 8–12)

Transform pilot learnings into formal standards that other teams can follow.

**Key principle:** Standards should be opinionated enough to prevent fragmentation but flexible enough to accommodate different tech stacks and team sizes.

```
Standard vs. Guideline:

Standard (must follow):
  "All repositories must have a .github/copilot-instructions.md file."
  "AI-generated code must pass the same review process as human code."
  "Approved AI tools list: [specific tools]."

Guideline (recommended):
  "Consider using AI for test generation before manual test writing."
  "Prompt library contributions are encouraged but not required."
  "Agent mode is recommended for multi-file refactoring."
```

### Phase 4: Expand With Standards (Weeks 10–16+)

Roll out to additional teams with the standards package in hand.

**Expansion sequence:**
```
Wave 1 (Weeks 10-12): 2-3 teams
  → Teams with champions who observed the pilot
  → Close support from pilot team members
  → Feedback loop to refine standards

Wave 2 (Weeks 12-14): 3-5 teams
  → Broader expansion with proven materials
  → Pilot team members as mentors (1 per new team)
  → Lighter support, more self-service

Wave 3 (Weeks 14-16+): Remaining teams
  → Organization-wide rollout
  → Standards are mature and well-documented
  → Self-service onboarding with office hours support
  → Ongoing community of practice
```

---

## What to Standardize

### 1. Instruction Files

The most critical standard. Without consistent instruction files, AI tools behave differently across teams.

**Standard to set:**

```markdown
# Instruction File Standard

## Location
All repositories MUST have: .github/copilot-instructions.md

## Required Sections
Every instruction file MUST include:
1. Project overview (what this repo does)
2. Tech stack (languages, frameworks, versions)
3. Code conventions (naming, structure, patterns)
4. Testing requirements (framework, coverage, patterns)
5. Patterns to avoid (anti-patterns specific to this project)

## Optional Sections
Teams MAY include:
- Architecture overview
- API conventions
- Database patterns
- Deployment notes

## Maintenance
- Instruction files MUST be reviewed quarterly
- Changes to instruction files MUST be reviewed by the team
- Instruction file changes SHOULD be discussed in retros
```

✅ **Standardized instruction file:** Every team's instruction file follows the same structure, making it easy for developers moving between teams to understand AI behavior.

❌ **Unstandardized instruction files:** Team A has a 3-line file, Team B has a 500-line file, Team C has no file, and Team D has a file in a non-standard location.

### 2. Review Process

AI-generated code requires review attention in different areas than human-written code. Standardize what reviewers look for.

**Standard review checklist for AI-generated code:**

```markdown
## AI Code Review Checklist

### Always Check
- [ ] Does the code actually solve the stated problem?
- [ ] Are there hallucinated APIs, methods, or imports?
- [ ] Does it handle edge cases (null, empty, boundary values)?
- [ ] Are error messages meaningful (not generic AI boilerplate)?
- [ ] Does it follow the project's established patterns?

### Security Check
- [ ] No hardcoded credentials or secrets
- [ ] Input validation is present and correct
- [ ] No SQL injection, XSS, or other injection vectors
- [ ] Authentication and authorization checks are in place
- [ ] Dependencies are from trusted sources

### Test Check
- [ ] Tests cover the happy path
- [ ] Tests cover error cases and edge cases
- [ ] Tests don't just test that the AI code works — they test behavior
- [ ] No tests that always pass (tautological assertions)
- [ ] Test data is realistic, not placeholder
```

### 3. Quality Gates

Define automated checks that AI-generated code must pass before it can be merged.

**Standard quality gate pipeline:**

```yaml
# .github/workflows/ai-quality-gates.yml
name: AI Quality Gates

on: [pull_request]

jobs:
  quality-check:
    runs-on: ubuntu-latest
    steps:
      - name: Lint check
        run: npm run lint
        # AI-generated code must pass all lint rules

      - name: Type check
        run: npm run typecheck
        # No 'any' types, all interfaces properly typed

      - name: Unit tests
        run: npm run test
        # All existing tests must still pass

      - name: Coverage check
        run: npm run test:coverage
        # Coverage must not decrease

      - name: Security scan
        run: npm audit && npm run security:scan
        # No new vulnerabilities introduced
```

✅ **Automated quality gates:** Every PR triggers the same checks regardless of whether code is AI-generated or human-written. No exceptions.

❌ **Manual quality gates:** Reviewers are expected to remember the checklist. Some do, some don't. Quality varies by reviewer.

### 4. Onboarding Guide

Every new team member (or team adopting AI tools) needs a clear path from zero to productive.

**Standard onboarding structure:**

```markdown
# AI Tools Onboarding Guide

## Day 1: Setup
- [ ] Install approved AI tools (list with links)
- [ ] Configure IDE settings (link to standard config)
- [ ] Verify instruction file is loaded
- [ ] Complete "Hello World" exercise with AI assist
- [ ] Join #ai-tools Slack channel

## Week 1: Foundations
- [ ] Complete autocomplete training module
- [ ] Pair with an experienced AI user for 2 hours
- [ ] Read team instruction file and ask questions
- [ ] Use AI assist for at least 3 tasks in daily work
- [ ] Share one "win" in the Slack channel

## Week 2: Expanding
- [ ] Complete chat-based coding training module
- [ ] Contribute one improvement to the instruction file
- [ ] Use AI for test generation on a real task
- [ ] Attend AI tools office hours
- [ ] Review prompt library and try 3 prompts

## Week 3-4: Proficiency
- [ ] Complete agent mode training module (if team is at this stage)
- [ ] Submit one prompt to the shared prompt library
- [ ] Pair with a new team member to teach what you've learned
- [ ] Complete self-assessment survey
```

### 5. Prompt Library

Shared, proven prompts prevent every developer from reinventing the wheel.

**Standard prompt library structure:**

```markdown
# Team Prompt Library

## Categories
- [Testing](#testing)
- [Refactoring](#refactoring)
- [Documentation](#documentation)
- [Debugging](#debugging)
- [Code Review](#code-review)

## Testing

### Generate Unit Tests
**When to use:** After implementing a new function or module
**Prompt:**
> Write unit tests for [function/module]. Cover: happy path,
> edge cases (null, empty, boundary values), error cases.
> Use [test framework]. Follow the Arrange-Act-Assert pattern.
> Each test should have a descriptive name explaining the scenario.

**Example from our codebase:**
> Write unit tests for the UserService.createUser method.
> Cover: successful creation, duplicate email, invalid email format,
> missing required fields. Use Jest with our test utilities from
> src/test/helpers.ts. Follow AAA pattern.

### Generate Integration Tests
**When to use:** After implementing an API endpoint
**Prompt:**
> Write integration tests for the [endpoint] endpoint.
> Test request validation, authentication, authorization,
> happy path response, and error responses.
> Use supertest with our test setup from src/test/setup.ts.
```

---

## When You're Ready to Scale

Don't scale until you can answer "yes" to all of these:

### Readiness Assessment

```markdown
# Scaling Readiness Checklist

## Pilot Results
- [ ] Pilot team has used AI tools for at least 6 weeks
- [ ] Measurable improvement in at least 2 key metrics
- [ ] No AI-related production incidents during pilot
- [ ] Pilot team recommends expansion (genuine enthusiasm, not obligation)

## Documentation Complete
- [ ] Instruction file template is tested and refined
- [ ] Onboarding guide has been used by at least 2 new adopters
- [ ] Review checklist is validated by code reviewers
- [ ] Quality gates are automated and running
- [ ] Prompt library has at least 10 proven entries
- [ ] FAQ addresses the top 20 questions from the pilot

## Training Materials Ready
- [ ] Self-paced training modules exist for each stage
- [ ] Hands-on exercises use real-world scenarios
- [ ] Workshop facilitator guide is documented
- [ ] Pairing protocol is defined

## Champions Identified
- [ ] At least 1 champion per target team in the next wave
- [ ] Champions have observed or participated in the pilot
- [ ] Champions have time allocated for the role (not just "also do this")
- [ ] Champion communication channel exists (Slack, Teams, etc.)

## Organizational Support
- [ ] Leadership has approved expansion plan and budget
- [ ] IT/Security has approved tool access for next wave
- [ ] Support capacity is planned for expansion
- [ ] Success metrics for expansion are defined
```

✅ **Ready to scale:** Pilot team shows 25% reduction in test-writing time, documentation is complete, 3 champions are identified and trained, and 4 teams have expressed interest.

❌ **Not ready to scale:** Pilot shows promising results after 3 weeks, documentation is a rough draft, no champions identified, but management wants all teams on board by end of quarter.

---

## The Standards Lifecycle

Standards aren't static. They evolve as you learn more and as tools improve.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   PROPOSE    │────▶│    PILOT     │────▶│   PUBLISH    │
│              │     │              │     │              │
│ Suggest a    │     │ Test with    │     │ Make it      │
│ standard     │     │ willing team │     │ official     │
└──────────────┘     └──────────────┘     └──────────────┘
       ▲                                        │
       │                                        ▼
┌──────────────┐                         ┌──────────────┐
│   EVOLVE     │◀────────────────────────│   REVIEW     │
│              │                         │              │
│ Update based │                         │ Quarterly    │
│ on feedback  │                         │ assessment   │
└──────────────┘                         └──────────────┘
```

### Quarterly Standards Review

Every quarter, assess each standard:

1. **Is it followed?** — If teams are routinely ignoring a standard, it's either wrong or poorly communicated
2. **Is it still relevant?** — Tools and practices evolve quickly; remove outdated standards
3. **Is it effective?** — Does following the standard actually improve outcomes?
4. **Can it be simplified?** — The best standards require minimum effort for maximum benefit

---

## Common Standardization Mistakes

### Mistake 1: Over-Standardizing

✅ **Right level:** "All instruction files must include tech stack, conventions, and testing requirements."

❌ **Over-standardized:** "All instruction files must be exactly 150 lines, use H2 headers only, include exactly 5 code examples, and be reviewed by the AI Standards Board every 2 weeks."

### Mistake 2: Standardizing Too Early

✅ **Right timing:** After 6–8 weeks of pilot experience with multiple iterations of the instruction file.

❌ **Too early:** After 1 week of pilot, before the team has encountered real challenges and refined their approach.

### Mistake 3: Ignoring Team Differences

✅ **Flexible standard:** "Each team must have an instruction file covering [required sections]. Additional sections are at team discretion."

❌ **Rigid standard:** "All teams must use this exact instruction file template, word for word, regardless of tech stack or project type."

### Mistake 4: No Feedback Mechanism

✅ **Open standard:** "Standards are reviewed quarterly. Submit change proposals via [process]. All teams can contribute."

❌ **Closed standard:** "Standards are set by the AI Governance Board. Teams follow them."

### Mistake 5: Documentation Without Examples

✅ **Practical standard:** "Here's the review checklist, and here are 3 examples of reviews that used it effectively — note how the reviewer caught a hallucinated API in example 2."

❌ **Abstract standard:** "Review AI-generated code thoroughly using best practices."

---

## Scaling Anti-Patterns to Avoid

### The Copy-Paste Expansion

```
Scenario: Team B copies Team A's instruction file verbatim.
Problem:  Team B uses Python/Django, Team A uses TypeScript/React.
Result:   AI gives TypeScript-style suggestions for Python code.
Fix:      Template with required sections, not copy-paste of content.
```

### The Standards Police

```
Scenario: A central team enforces standards through audits and tickets.
Problem:  Teams feel policed, not supported. Compliance is grudging.
Result:   Standards are followed minimally, spirit is ignored.
Fix:      Standards are supported by champions, not enforced by police.
```

### The Abandoned Pilot

```
Scenario: Pilot team succeeds, but no one captures their practices.
Problem:  Knowledge stays in the pilot team's heads.
Result:   Every subsequent team rediscovers the same lessons.
Fix:      Documentation is an explicit pilot deliverable, not an afterthought.
```

---

## Template: Standardization Roadmap

```markdown
# AI Standardization Roadmap: [Organization Name]

## Pilot Phase (Weeks 1-8)
- Pilot team: ___
- Champion: ___
- Focus areas: ___
- Success metrics: ___
- Documentation owner: ___

## Standardization Phase (Weeks 6-12)
- Standards author: ___
- Review committee: ___
- Standard deliverables:
  - [ ] Instruction file template
  - [ ] Review checklist
  - [ ] Quality gate configuration
  - [ ] Onboarding guide
  - [ ] Prompt library (minimum 10 entries)
  - [ ] FAQ (minimum 20 questions)
  - [ ] Training materials (3 modules)

## Wave 1 Expansion (Weeks 10-12)
- Teams: ___
- Champions: ___
- Mentor assignments: ___
- Feedback collection method: ___
- Standards revision target: ___

## Wave 2 Expansion (Weeks 12-14)
- Teams: ___
- Champions: ___
- Self-service onboarding: Ready? ___
- Support model: ___

## Wave 3 / Full Rollout (Weeks 14-16+)
- Remaining teams: ___
- Community of practice established: ___
- Ongoing support model: ___
- Quarterly review cadence: ___

## Long-Term Governance
- Standards review: Quarterly
- Standards owner: ___
- Feedback channel: ___
- Evolution process: ___
```
