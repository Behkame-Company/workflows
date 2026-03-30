# Testing Strategy with AI

> A comprehensive guide to AI-assisted testing — from TDD workflows to evaluating AI-generated code quality.

---

## Why AI Changes Testing

AI coding tools can generate code faster than ever. This makes testing **more important**, not less:
- AI-generated code needs validation (models make plausible-looking mistakes)
- Tests serve as **specifications** that constrain AI output
- Tests are the fastest feedback loop for AI agents
- Coverage gaps in AI-generated code are common without explicit testing guidance

---

## Table of Contents

### [01 — Fundamentals](01-fundamentals/)
| Document | Description |
|----------|-------------|
| [Why Testing Matters More with AI](01-fundamentals/why-testing-matters-more.md) | AI amplifies both speed and risk |
| [Testing Pyramid with AI](01-fundamentals/testing-pyramid-with-ai.md) | How AI changes the traditional testing pyramid |
| [Test as Specification](01-fundamentals/test-as-specification.md) | Using tests to constrain AI behavior |
| [Choosing a Testing Framework](01-fundamentals/choosing-a-framework.md) | Framework selection for AI-assisted workflows |

### [02 — TDD with AI](02-tdd-with-ai/)
| Document | Description |
|----------|-------------|
| [TDD Workflow with AI Agents](02-tdd-with-ai/tdd-workflow.md) | Red-green-refactor with AI assistance |
| [Writing Tests First for AI](02-tdd-with-ai/writing-tests-first.md) | How to write tests that guide AI code generation |
| [Test-Driven Bug Fixing](02-tdd-with-ai/test-driven-bug-fixing.md) | Write the failing test, then let AI fix |
| [Spec-Driven Testing](02-tdd-with-ai/spec-driven-testing.md) | Generating tests from specifications |

### [03 — Test Generation](03-test-generation/)
| Document | Description |
|----------|-------------|
| [AI Test Generation Patterns](03-test-generation/generation-patterns.md) | Prompt patterns for generating tests |
| [Unit Test Generation](03-test-generation/unit-test-generation.md) | Generating unit tests with AI |
| [Integration Test Generation](03-test-generation/integration-test-generation.md) | AI-assisted integration testing |
| [Edge Case Discovery](03-test-generation/edge-case-discovery.md) | Using AI to find untested scenarios |

### [04 — Test Types](04-test-types/)
| Document | Description |
|----------|-------------|
| [Unit Testing with AI](04-test-types/unit-testing.md) | Isolated component testing |
| [Integration Testing with AI](04-test-types/integration-testing.md) | Cross-boundary testing |
| [E2E Testing with AI](04-test-types/e2e-testing.md) | Full workflow validation |
| [Property-Based Testing](04-test-types/property-based-testing.md) | Generative testing with AI |
| [Snapshot and Visual Testing](04-test-types/snapshot-testing.md) | UI regression detection |

### [05 — Coverage & Quality](05-coverage-and-quality/)
| Document | Description |
|----------|-------------|
| [Coverage Analysis](05-coverage-and-quality/coverage-analysis.md) | Measuring and improving test coverage |
| [Test Quality Metrics](05-coverage-and-quality/test-quality-metrics.md) | Beyond line coverage |
| [Mutation Testing](05-coverage-and-quality/mutation-testing.md) | Testing your tests with mutations |
| [AI-Generated Code Quality](05-coverage-and-quality/ai-code-quality.md) | Validating AI output quality |

### [06 — AI QA Workflows](06-ai-qa-workflows/)
| Document | Description |
|----------|-------------|
| [Copilot Testing Workflows](06-ai-qa-workflows/copilot-testing.md) | GitHub Copilot for testing |
| [Claude Code Testing Patterns](06-ai-qa-workflows/claude-code-testing.md) | Claude's test-first approach |
| [CI/CD Test Integration](06-ai-qa-workflows/ci-cd-integration.md) | Automated testing in pipelines |
| [Coding Agent Test Verification](06-ai-qa-workflows/agent-verification.md) | How AI agents verify their own work |

### [07 — Evaluating AI Output](07-evaluating-ai-output/)
| Document | Description |
|----------|-------------|
| [Evaluation Framework](07-evaluating-ai-output/evaluation-framework.md) | Systematic AI output evaluation |
| [LLM-Based Grading](07-evaluating-ai-output/llm-based-grading.md) | Using AI to evaluate AI |
| [Building Eval Suites](07-evaluating-ai-output/building-eval-suites.md) | Creating comprehensive evaluation sets |

### [08 — Regression & Maintenance](08-regression-and-maintenance/)
| Document | Description |
|----------|-------------|
| [Regression Testing Strategy](08-regression-and-maintenance/regression-testing.md) | Preventing regressions from AI changes |
| [Test Maintenance with AI](08-regression-and-maintenance/test-maintenance.md) | Keeping tests current |
| [Flaky Test Detection](08-regression-and-maintenance/flaky-test-detection.md) | Finding and fixing unreliable tests |

### [09 — Best Practices](09-best-practices/)
| Document | Description |
|----------|-------------|
| [Test-First AI Development](09-best-practices/test-first-development.md) | Always write tests before or alongside AI code |
| [Prompt Patterns for Testing](09-best-practices/prompt-patterns.md) | Effective prompts for test generation |
| [Structured Test Tracking](09-best-practices/structured-test-tracking.md) | JSON test lists for AI sessions |
| [Testing Across Sessions](09-best-practices/testing-across-sessions.md) | Maintaining test state in long projects |

### [10 — Anti-Patterns](10-anti-patterns/)
| Document | Description |
|----------|-------------|
| [Testing to Pass](10-anti-patterns/testing-to-pass.md) | AI writing tests that always pass |
| [Mock Everything](10-anti-patterns/mock-everything.md) | Over-mocking that hides real bugs |
| [Snapshot Abuse](10-anti-patterns/snapshot-abuse.md) | Snapshots that test nothing |
| [Skipping AI Verification](10-anti-patterns/skipping-verification.md) | Trusting AI output without testing |

### [11 — Practical Resources](11-practical/)
| Document | Description |
|----------|-------------|
| [Templates](11-practical/templates.md) | Test prompt templates |
| [Troubleshooting](11-practical/troubleshooting.md) | Common testing issues |
| [FAQ](11-practical/faq.md) | Frequently asked questions |
| [Resources](11-practical/resources.md) | Links and tools |

---

## Quick Start

1. **New to AI testing?** → [Why Testing Matters More](01-fundamentals/why-testing-matters-more.md)
2. **Want TDD with AI?** → [TDD Workflow](02-tdd-with-ai/tdd-workflow.md)
3. **Generating tests?** → [Generation Patterns](03-test-generation/generation-patterns.md)
4. **Evaluating AI code?** → [Evaluation Framework](07-evaluating-ai-output/evaluation-framework.md)
