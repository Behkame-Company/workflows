# Terminology

> Key terms and definitions for AI code review and CI/CD integration.

---

## Code Review Terms

| Term | Definition |
|------|-----------|
| **Pull Request (PR)** | A request to merge code changes from one branch to another; the primary unit of code review |
| **Diff** | The set of changes between two code versions; what reviewers examine |
| **Code Review** | The process of examining code changes for quality, correctness, and adherence to standards |
| **Reviewer** | A person (or AI) that examines code changes and provides feedback |
| **Approval** | A reviewer's sign-off indicating the code is ready to merge |
| **Request Changes** | A reviewer's indication that issues must be fixed before merging |
| **Review Comment** | Feedback on a specific line or section of code |
| **Code Smell** | A surface indication of a deeper problem in the code |
| **False Positive** | An AI finding that is not actually an issue — noise in the review |
| **Suggestion** | A proposed code change that can be applied directly from the review |

## AI Review Terms

| Term | Definition |
|------|-----------|
| **AI Code Review** | Automated analysis of code changes by a large language model |
| **Review Bot** | An automated agent that posts review comments on PRs |
| **Signal-to-Noise Ratio** | The proportion of useful findings vs. irrelevant comments |
| **Confidence Score** | How certain the AI is about a finding (used for escalation) |
| **Review Sensitivity** | Configuration that controls how strict/permissive the AI reviewer is |
| **Custom Instructions** | Rules provided to AI reviewers via configuration files to guide behavior |
| **Automation Bias** | The tendency to trust AI output without sufficient verification |

## CI/CD Terms

| Term | Definition |
|------|-----------|
| **CI (Continuous Integration)** | Automatically building and testing code on every push |
| **CD (Continuous Delivery)** | Automatically preparing code for release after CI passes |
| **Pipeline** | A sequence of automated stages (build → test → review → deploy) |
| **Quality Gate** | An automated checkpoint that blocks progression if standards aren't met |
| **GitHub Actions** | GitHub's CI/CD platform for defining automated workflows |
| **Workflow** | A YAML-defined automation process triggered by events |
| **Job** | A set of steps that run on the same runner within a workflow |
| **Runner** | The machine (VM or container) that executes workflow jobs |
| **Artifact** | A file produced by a CI/CD job (test results, builds, reports) |

## Security Terms

| Term | Definition |
|------|-----------|
| **SAST** | Static Application Security Testing — analyzes source code for vulnerabilities |
| **DAST** | Dynamic Application Security Testing — tests running applications for vulnerabilities |
| **SCA** | Software Composition Analysis — checks dependencies for known vulnerabilities |
| **CodeQL** | GitHub's semantic code analysis engine for security scanning |
| **Semgrep** | A fast, customizable static analysis tool for finding patterns |
| **Secrets Scanning** | Detection of accidentally committed credentials (API keys, tokens) |

## Agentic CI/CD Terms

| Term | Definition |
|------|-----------|
| **Agentic Workflow** | A GitHub Actions workflow driven by AI agent instructions in Markdown |
| **Safe Outputs** | Controlled actions an AI agent is allowed to perform (labels, comments) |
| **Continuous AI** | The paradigm of AI agents continuously monitoring and improving repositories |
| **Self-Healing CI** | AI that diagnoses CI failures and proposes fixes automatically |
| **Human-in-the-Loop (HITL)** | Requiring human approval for AI-proposed changes before execution |

---

## Next Steps

- 🔗 [GitHub Copilot Code Review](../02-review-tools/github-copilot-code-review.md) — The primary review tool
- 🔗 [CI/CD Overview](../03-ci-cd-fundamentals/ci-cd-overview.md) — Pipeline fundamentals
