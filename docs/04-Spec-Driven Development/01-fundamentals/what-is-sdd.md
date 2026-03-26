# What Is Spec-Driven Development?

> The paradigm shift from code-first to spec-first software development with AI.

---

## The Core Idea

Spec-Driven Development (SDD) inverts the traditional software workflow. Instead of writing code and documenting later, you **write comprehensive specifications first** and let AI generate the implementation.

```
Traditional:    Idea → Code → Tests → Documentation
SDD:            Idea → Specification → AI Implementation → Validation
```

The specification becomes the **source of truth**. Code is a direct expression of the spec, not the other way around.

---

## Why SDD Emerged

SDD arose from two converging trends:

### 1. The Vibe-Coding Crisis
Early AI coding (2023–2024) was dominated by "vibe coding" — developers fed ad-hoc prompts to AI and hoped for the best. Results were:
- Inconsistent quality across features
- Architectural drift as context was lost
- "The three-month wall" — projects became unmaintainable
- Security and compliance gaps from skipped analysis

### 2. The Context Engineering Revolution
Engineers realized that **the quality of AI output is determined by the quality of its input**. Better context → better code. Specifications are the richest form of context you can provide to an AI agent.

---

## SDD in One Paragraph

> Write a detailed specification describing *what* to build, *why* it matters, *who* uses it, *what edge cases exist*, and *how success is measured*. Have the AI (or team) review and refine the spec. Then use the spec as the authoritative guide for planning, task breakdown, and implementation — with the AI validating its own output against the spec at every step.

---

## Key Vocabulary

| Term | Definition |
|------|-----------|
| **Spec (Specification)** | A structured document describing what to build, acceptance criteria, and constraints |
| **Constitution** | Project-level principles and non-negotiable constraints |
| **Plan** | Technical architecture derived from a spec |
| **Task** | A discrete, implementable unit of work from a plan |
| **Quality Gate** | A validation checkpoint between workflow phases |
| **Living Spec** | A specification that evolves alongside the code it describes |
| **Spec-Code Drift** | When code diverges from its specification over time |

---

## What SDD Is NOT

| SDD Is... | SDD Is NOT... |
|-----------|---------------|
| Writing clear requirements before coding | Writing 100-page PRDs nobody reads |
| Iterative refinement of specs | Waterfall big-design-up-front |
| AI-readable structured context | Prose novels for AI to interpret |
| Living, version-controlled artifacts | One-time documents archived after sprint |
| A complement to TDD/BDD | A replacement for testing |

---

## The SDD Promise

When practiced correctly, SDD delivers:

1. **Reduced rework** — Fewer "that's not what I meant" moments
2. **Consistent quality** — Every feature built against explicit criteria
3. **Knowledge retention** — Specs survive context window resets and team turnover
4. **Audit trails** — Every decision traceable from requirement to implementation
5. **AI effectiveness** — Agents perform dramatically better with structured specs vs. ad-hoc prompts

---

## A Brief History

| Year | Milestone |
|------|-----------|
| 2023 | "Vibe coding" era — prompt → generate → hope |
| 2024 | Teams discover that structured prompts improve AI output quality |
| Early 2025 | GitHub releases Spec Kit as open-source toolkit |
| Mid 2025 | AWS launches Kiro, a spec-driven agentic IDE |
| Late 2025 | Martin Fowler / ThoughtWorks publishes SDD analysis |
| 2026 | SDD becomes standard practice alongside meta-prompting |

---

## Who Should Use SDD?

- **Team leads** standardizing AI workflows across developers
- **Product engineers** building features with AI coding agents
- **Architects** defining system boundaries before AI implementation
- **Solo developers** who want repeatable, maintainable AI-generated code
- **Regulated industries** requiring specification-to-implementation traceability

---

*Next: [SDD vs Vibe Coding →](sdd-vs-vibe-coding.md)*
