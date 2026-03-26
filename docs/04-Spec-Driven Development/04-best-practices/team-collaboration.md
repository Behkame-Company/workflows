# Team Collaboration in SDD

> Roles, ownership, and review workflows for spec-driven teams.

---

## Roles in SDD

### Spec Author
Writes the initial specification. Usually the product engineer or team lead closest to the feature.

**Responsibilities**:
- Draft the spec from requirements or product input
- Facilitate clarification rounds
- Incorporate review feedback
- Keep spec updated during implementation

### Spec Reviewer
Reviews specifications for quality, completeness, and feasibility.

**Best reviewers**:
- Someone who will implement the feature (catches ambiguity)
- Someone from a different team (catches assumptions)
- QA engineer (catches missing test scenarios)

### Plan Owner
Creates the technical plan from the approved spec. Usually a senior engineer.

### Implementer
Executes the tasks, either manually or by directing AI agents.

---

## Team Workflows

### Small Team (2-5 devs)

```
1. Author writes spec (30 min)
2. Peer review in PR (15 min)
3. Author creates plan + tasks (30 min)
4. Peer reviews plan (15 min)
5. Implementer executes
6. Code review as normal
```

**Total ceremony**: ~90 minutes per medium feature

### Medium Team (6-15 devs)

```
1. Product + Engineering write spec together (1 hour)
2. Spec review by 2 reviewers (30 min async)
3. Architecture review meeting for plan (30 min)
4. Tasks reviewed by implementer (15 min)
5. Implementation with daily check-ins
6. Code review + spec validation
```

### Large Team / Enterprise

```
1. Product manager writes feature requirements
2. Engineering lead converts to SDD spec
3. Architecture review board approves plan
4. Tech lead creates task breakdown
5. Multiple implementers execute in parallel
6. Integration testing against spec
7. Spec archived with compliance trail
```

---

## Ownership Model

### Feature Ownership

```
Feature: Notifications
├── Spec Owner: @jane (product engineer)
├── Plan Owner: @bob (senior engineer)
├── Implementation: @alice, @charlie (engineers)
└── Review: @dave (QA lead)
```

### Spec Ownership Rules
1. **Every spec has exactly one owner** — the person responsible for keeping it current
2. **Ownership transfers with feature ownership** — when a feature changes hands
3. **Owner doesn't mean solo author** — collaboration is encouraged
4. **Owner = final decision maker** — when reviewers disagree

---

## Communication Patterns

### Async-First Spec Review
Use PR reviews for spec iterations:
```
Author opens PR with spec.md → Reviewers comment → Author addresses → Merge
```

### Spec Discussion Thread
For complex features, use a dedicated discussion:
```
GitHub Discussion: "Spec: Notification System"
├── @jane: Here's the draft spec, questions below
├── @bob: What about rate limiting?
├── @jane: Good point — added to v1.1
├── @alice: WebSocket or SSE?
├── @jane: Good question — added to Open Questions
└── Resolution: WebSocket chosen for bidirectional support
```

### Spec Presentation (complex features)
5-minute walkthrough of the spec for the team:
```
1. What we're building and why (Summary)
2. Key user stories (2-3 most important)
3. Important edge cases
4. Discussion / questions
```

---

## Handling Disagreements

### Spec-Level Disagreements

| Type | Resolution |
|------|-----------|
| Scope disagreement | Product owner decides what's in/out |
| Technical approach | Add to spec as "Open Question," resolve in Plan phase |
| Acceptance criteria | Make both versions testable, data-decide |
| Priority | Product roadmap determines priority |

### During Implementation

```
Developer: "The spec says X but I think Y is better"
Process:
  1. Propose the change with rationale
  2. Update spec if approved
  3. THEN implement the change
  4. Never silently deviate from spec
```

---

## Spec Templates for Teams

### Standardize with Templates

Provide templates so everyone writes specs the same way:

```
.specify/templates/
├── feature-spec.md       # New feature template
├── enhancement-spec.md   # Change to existing feature
├── api-spec.md          # New API endpoint/service
├── migration-spec.md    # Data or system migration
└── bugfix-spec.md       # Complex bug fix (simple bugs skip spec)
```

### Template Adoption
- Start with one template for the first month
- Add templates as new spec types are needed
- Review and refine templates quarterly

---

## Measuring Team SDD Effectiveness

| Metric | Target | How to Measure |
|--------|--------|---------------|
| Spec coverage | >80% of features have specs | Count specs vs shipped features |
| Spec review time | <1 day turnaround | Track PR review time |
| Rework rate | <20% of tasks need rework | Track task reopens |
| Spec accuracy | >90% at completion | Audit spec vs final implementation |

---

*Next: [Common Mistakes →](../05-anti-patterns/common-mistakes.md)*
