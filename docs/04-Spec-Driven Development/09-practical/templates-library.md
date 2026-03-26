# Templates Library

> Ready-to-use templates for specifications, plans, tasks, and constitutions.

---

## Template 1: Feature Specification

```markdown
# Feature: [Feature Name]

| Field | Value |
|-------|-------|
| Version | 1.0 |
| Status | Draft |
| Author | @[author] |
| Created | [YYYY-MM-DD] |

## Summary
[1 paragraph: What is this feature and why does it matter?]

## User Stories

### US-1: [Story Title]
As a [role], I want [goal], so that [benefit].

### US-2: [Story Title]
As a [role], I want [goal], so that [benefit].

## Acceptance Criteria

### US-1: [Story Title]
- AC-1.1: Given [precondition]
  When [action]
  Then [expected result]
- AC-1.2: Given [precondition]
  When [action]
  Then [expected result]

### US-2: [Story Title]
- AC-2.1: Given [precondition]
  When [action]
  Then [expected result]

## Edge Cases

| Scenario | Expected Behavior |
|----------|------------------|
| [Edge case 1] | [Expected behavior] |
| [Edge case 2] | [Expected behavior] |
| [Edge case 3] | [Expected behavior] |

## Non-Functional Requirements

### Performance
- [Metric]: [Target value]

### Security
- [Requirement]

### Accessibility
- [Standard]

## Out of Scope
- [Item 1] — [reason/timeline]
- [Item 2] — [reason/timeline]

## Open Questions
- [ ] [Question 1]
- [ ] [Question 2]

## Dependencies
- [Dependency 1]
```

---

## Template 2: Technical Plan

```markdown
# Plan: [Feature Name]

| Field | Value |
|-------|-------|
| Spec | .specify/specs/[feature]/spec.md |
| Author | @[author] |
| Created | [YYYY-MM-DD] |

## Overview
[1 paragraph: Technical approach summary]

## Architecture

### Components
- [Component A] — [responsibility]
- [Component B] — [responsibility]

### Component Interaction
[Diagram or description of how components connect]

## Data Model

### New Tables/Collections
| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| id | UUID | PK | Auto-generated |
| [field] | [type] | [constraints] | [notes] |

### Modified Tables
- [Table]: Add [column] ([type]) — [reason]

## API Design

### [METHOD] [Path]
- **Purpose**: [Description]
- **Request**: [Schema or example]
- **Response**: [Schema or example]
- **Errors**: [Error codes and meanings]

## Integration Points
- [System A]: [How this feature interacts with it]
- [System B]: [How this feature interacts with it]

## Testing Strategy
- **Unit**: [What to test at unit level]
- **Integration**: [What to test at integration level]
- **E2E**: [What to test end-to-end]

## Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| [Risk 1] | [H/M/L] | [H/M/L] | [Mitigation] |

## Architecture Decisions

### [Decision Title]
- **Options**: [Option A], [Option B]
- **Chosen**: [Option X]
- **Reason**: [Why this option]
```

---

## Template 3: Task Breakdown

```markdown
# Tasks: [Feature Name]

| Field | Value |
|-------|-------|
| Spec | .specify/specs/[feature]/spec.md |
| Plan | .specify/specs/[feature]/plan.md |
| Total Tasks | [N] |

## Dependency Graph
[Task 1] → [Task 2] → [Task 3]
                     → [Task 4] (parallel with 3)

## Task 1: [Title]
- **Description**: [What to do, 2-3 sentences]
- **Files**: [New or modified files]
- **Acceptance**: [How to verify completion]
- **Dependencies**: None
- **Spec Reference**: [AC-X.X]

## Task 2: [Title]
- **Description**: [What to do]
- **Files**: [Files]
- **Acceptance**: [Verification]
- **Dependencies**: Task 1
- **Spec Reference**: [AC-X.X]
```

---

## Template 4: Project Constitution

```markdown
# [Project Name] — Constitution

## Mission
[1 sentence: What this project does and for whom]

## Design Principles
1. **[Principle]**: [Why it matters]
2. **[Principle]**: [Why it matters]
3. **[Principle]**: [Why it matters]

## Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | [Lang] | [Version] |
| Framework | [Framework] | [Version] |
| Database | [DB] | [Version] |
| Testing | [Framework] | [Version] |

## Coding Standards
- [Standard 1]
- [Standard 2]
- [Standard 3]

## Quality Requirements
- Test coverage: >[N]% for new code
- [Quality requirement 2]
- [Quality requirement 3]

## Security Requirements
- [Security requirement 1]
- [Security requirement 2]

## API Conventions
- [Convention 1]
- [Convention 2]
```

---

## Template 5: Lightweight Spec (5-Minute Version)

```markdown
# [Feature Name]

**What**: [1 sentence description]
**Why**: [1 sentence justification]
**Who**: [Target user]

## Success Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

## Edge Cases
- [Edge case 1]: [Expected behavior]
- [Edge case 2]: [Expected behavior]

## Not Included
- [Out of scope item]
```

---

## Template Usage Tips

1. **Don't fill every field** — Templates show maximum structure. Skip sections that don't apply.
2. **Adapt to your project** — Add sections for your domain (compliance, i18n, etc.)
3. **Start with Template 5** — If new to SDD, use the lightweight template until comfortable.
4. **Version your templates** — When you improve a template, update it in `.specify/templates/`.

---

*Next: [Examples Gallery →](examples-gallery.md)*
