# Quality Gates

> Validation checkpoints between phases that prevent errors from cascading.

---

## What Are Quality Gates?

Quality gates are **mandatory review checkpoints** between SDD phases. No phase advances until its gate passes.

```
Specify ──[Gate 1]──► Plan ──[Gate 2]──► Tasks ──[Gate 3]──► Implement ──[Gate 4]──► Merge
```

---

## Why Quality Gates Matter

Without gates, errors cascade and amplify:

```
❌  Without Gates:
  Vague spec → Wrong architecture → Bad tasks → Broken code → Expensive rework

✅  With Gates:
  Vague spec → Gate catches it → Spec refined → Right architecture → ...
```

**The cost of catching errors increases 10x at each phase**:
- Fixing a spec: 10 minutes
- Fixing a plan (wrong architecture): 2 hours
- Fixing tasks (wrong decomposition): 4 hours
- Fixing implementation (wrong code): 1–2 days

---

## Gate 1: Spec Review

**Between**: Specify → Plan

| Check | Question | Pass Criteria |
|-------|----------|---------------|
| Completeness | Are all required sections filled? | No empty sections |
| Clarity | Would a new team member understand? | No jargon without definition |
| Testability | Can each criterion be tested? | Every AC is verifiable |
| Scope | Is "out of scope" defined? | Clear boundaries stated |
| Edge Cases | Are error/boundary cases listed? | At least 3 edge cases |
| Stakeholder Sign-off | Have relevant people approved? | Explicit approval |

### Automated Checks (if tooling supports)
```bash
speckit validate spec.md
# ✅ All required sections present
# ✅ Acceptance criteria use Given/When/Then format
# ⚠️ No edge cases for error scenarios
# ❌ "Out of scope" section is missing
```

---

## Gate 2: Plan Review

**Between**: Plan → Tasks

| Check | Question | Pass Criteria |
|-------|----------|---------------|
| Spec Coverage | Does plan address every spec requirement? | 1:1 mapping verified |
| Architecture Fit | Is the architecture appropriate for scale? | Justified for project size |
| Existing Code | Does plan integrate with current codebase? | References existing patterns |
| Technical Risks | Are risks identified with mitigations? | Risk table present |
| Testing Strategy | Is there a test plan? | Strategy covers unit/integration/E2E |

### Coverage Verification
```
Spec Requirements:         Plan Sections:
├── User Stories (5)       ├── Architecture     → covers stories 1,2,3 ✅
├── Acceptance (8)         ├── Data Model       → covers criteria 4,5 ✅
├── Edge Cases (4)         ├── API Design       → covers criteria 6,7,8 ✅
├── Non-Functional (3)     ├── Testing Strategy → covers edge cases ✅
└── Dependencies (2)       └── Integration      → covers deps ✅
                           Total coverage: 100% ✅
```

---

## Gate 3: Task Review

**Between**: Tasks → Implement

| Check | Question | Pass Criteria |
|-------|----------|---------------|
| Plan Coverage | Does every plan item have tasks? | No plan sections without tasks |
| Task Size | Is each task completable in one session? | No task > ~1 hour |
| Dependencies | Are task dependencies correct? | Valid DAG, no cycles |
| Acceptance | Does each task have success criteria? | Every task has AC |
| Testing | Are test tasks included? | Tests exist alongside implementation |
| Ordering | Is the sequence logical? | Dependencies respected |

---

## Gate 4: Merge Review

**Between**: Implement → Merge/Deploy

| Check | Question | Pass Criteria |
|-------|----------|---------------|
| All Tasks Done | Are all tasks marked complete? | No incomplete tasks |
| AC Passed | Do all acceptance criteria pass? | Manual or automated check |
| Edge Cases | Are edge cases handled? | Test coverage for each |
| Tests Green | Do all tests pass? | CI pipeline green |
| Code Review | Has a human reviewed the code? | Approved by reviewer |
| Spec Updated | Is the spec current with any deviations? | No spec-code drift |
| Documentation | Is documentation updated? | API docs, README, etc. |

---

## Lightweight vs Heavyweight Gates

Not every project needs full ceremony. Scale the gates:

| Project Type | Gate Depth |
|-------------|------------|
| Personal project | Self-review checklist (2 min) |
| Small team | Peer review (10 min) |
| Team project | Formal review meeting (30 min) |
| Enterprise / Regulated | Sign-off with audit trail |

### Lightweight Gate Example
```
Before moving from Spec to Plan:
  □ I re-read the spec with fresh eyes
  □ I can explain it to someone who doesn't know the project
  □ Acceptance criteria are specific enough to test
  → Move to Plan ✅
```

---

## Automating Quality Gates

### CI/CD Integration
```yaml
# .github/workflows/spec-check.yml
on:
  pull_request:
    paths: ['.specify/**']

jobs:
  validate-spec:
    steps:
      - name: Check spec structure
        run: speckit validate
      - name: Check spec-plan coverage
        run: speckit coverage --spec spec.md --plan plan.md
      - name: Check task decomposition
        run: speckit validate-tasks
```

### AI-Assisted Validation
```
Prompt: "Review this spec for completeness. Check:
1. Are all acceptance criteria testable?
2. Are edge cases documented?
3. Is scope clearly bounded?
4. Would a developer know exactly what to build?"
```
