# Multi-Agent SDD Pipelines

> Orchestrating multiple AI agents through the spec-driven workflow.

---

## The Multi-Agent Vision

Instead of one AI agent doing everything, specialized agents handle each SDD phase:

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ Spec      │──►│ Plan     │──►│ Task     │──►│ Code     │──►│ Review   │
│ Agent     │   │ Agent    │   │ Agent    │   │ Agent    │   │ Agent    │
│           │   │          │   │          │   │          │   │          │
│ Drafts &  │   │ Creates  │   │ Decomposes│  │ Writes   │   │ Validates│
│ refines   │   │ arch.    │   │ into work│   │ code     │   │ against  │
│ specs     │   │ plans    │   │ units    │   │ & tests  │   │ spec     │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
      │              │              │              │              │
      ▼              ▼              ▼              ▼              ▼
  [Human Gate]  [Human Gate]  [Human Gate]  [Auto Tests]   [Human Final]
```

---

## Agent Specialization

### Spec Agent
**Role**: Create and refine specifications
**Strengths**: Requirements elicitation, gap identification, consistency checking
**Prompt focus**: "You are a requirements analyst..."

```
System: You are a specification expert. Your job is to write clear,
complete, testable specifications. Always:
- Ask clarifying questions before writing
- Include acceptance criteria in Given/When/Then format
- Document at least 5 edge cases
- Define explicit scope boundaries
- Reference the project constitution
```

### Plan Agent
**Role**: Create technical plans from specs
**Strengths**: Architecture design, technology selection, integration analysis
**Prompt focus**: "You are a software architect..."

```
System: You are a software architect. Given a specification,
create a technical plan that:
- Addresses every spec requirement
- Integrates with the existing codebase
- Follows the project constitution's technology choices
- Identifies technical risks with mitigations
- Proposes a testing strategy
```

### Task Agent
**Role**: Decompose plans into implementable tasks
**Strengths**: Work breakdown, dependency analysis, parallelization
**Prompt focus**: "You are a project planner..."

```
System: You are a task decomposition expert. Given a plan:
- Break into tasks completable in 1-2 hours each
- Identify dependencies between tasks
- Group parallelizable tasks
- Include test tasks alongside implementation tasks
- Define clear acceptance criteria per task
```

### Code Agent
**Role**: Implement tasks
**Strengths**: Code generation, test writing, refactoring
**Prompt focus**: "You are a senior developer..."

### Review Agent
**Role**: Validate implementation against spec
**Strengths**: Gap analysis, acceptance testing, drift detection
**Prompt focus**: "You are a QA engineer..."

```
System: You are a specification compliance reviewer. Given:
- A specification with acceptance criteria
- An implementation (code files)
Compare them and report:
- Which acceptance criteria are met (with evidence)
- Which acceptance criteria are NOT met (with specifics)
- Any behaviors implemented that aren't in the spec
- Any spec drift detected
```

---

## Orchestration Patterns

### Pattern 1: Sequential Pipeline
```
Spec → [human] → Plan → [human] → Tasks → [human] → Code → Review → [human]
```
**Best for**: Critical features, regulated environments

### Pattern 2: Parallel Generation + Human Review
```
Spec → [human] → Plan + Tasks (parallel) → [human] → Code → Review → [human]
```
**Best for**: Medium features, experienced teams

### Pattern 3: Full Automation with Human Gates
```
Spec ──[auto]──► Plan ──[auto]──► Tasks ──[auto]──► Code ──[auto]──► Review
  │                │                │                │                  │
  ▼                ▼                ▼                ▼                  ▼
[HUMAN]          [HUMAN]          [HUMAN]          [TESTS]           [HUMAN]
```
**Best for**: Well-understood feature patterns

---

## Implementation Strategies

### Using Copilot CLI
```bash
# Spec Agent
copilot "Act as a spec agent. Read the feature request in FEATURE.md
and draft a spec following .specify/templates/spec-template.md"

# Plan Agent
copilot "Act as a plan agent. Read .specify/specs/feature/spec.md
and create a technical plan"

# Task Agent
copilot "Act as a task agent. Read plan.md and create an ordered
task list with dependencies"

# Code Agent
copilot "Act as a code agent. Implement Task 1 from tasks.md.
Follow copilot-instructions.md standards"

# Review Agent
copilot "Act as a review agent. Compare the implementation in
src/features/ against spec.md. Report AC compliance"
```

### Using GSD Framework
The GSD (Get Shit Done) framework has built-in multi-agent support:
```
gsd-discuss-phase  → Spec Agent equivalent
gsd-plan-phase     → Plan Agent equivalent
gsd-execute-phase  → Code Agent with task orchestration
gsd-verify-work    → Review Agent equivalent
```

---

## Challenges and Mitigations

| Challenge | Mitigation |
|-----------|------------|
| Agent output inconsistency | Use structured templates and examples |
| Context loss between agents | Pass artifacts explicitly (spec → plan → tasks) |
| Error propagation | Quality gates between every phase |
| Over-delegation to AI | Human reviews at every gate |
| Token/cost management | Right-size context per agent (load only relevant sections) |
