# GitHub Agentic Workflows

> Intent-driven CI/CD automation where AI agents interpret Markdown instructions and execute tasks through GitHub Actions.

---

## What Are Agentic Workflows?

Agentic Workflows let you describe automation in **natural language (Markdown)** instead of complex YAML. An AI agent (Copilot, Claude, or Codex) interprets your instructions and executes them through GitHub Actions.

```
Traditional CI/CD:     YAML rules → Deterministic execution
Agentic Workflows:     Markdown intent → AI interprets → Guided execution
```

---

## How It Works

### 1. Write a Workflow in Markdown

```markdown
---
on:
  issues:
    types: [opened]
permissions:
  issues: read
  contents: read
safe-outputs:
  add-labels:
    allowed: [bug, enhancement, documentation, question]
  add-comment: {}
tools:
  github:
    toolsets: [issues, labels]
---

# Issue Triage Agent

When a new issue is opened:
1. Read the issue title and body
2. Determine the most appropriate label based on content
3. Apply the label
4. Post a comment explaining your classification decision
```

### 2. Compile to GitHub Actions
```bash
gh aw compile issue-triage.md
# Generates issue-triage.lock.yml (executable Actions workflow)
```

### 3. Deploy and Monitor
The compiled workflow runs as a standard GitHub Action, but with an AI agent doing the reasoning.

---

## Key Concepts

### Safe Outputs
Control what the AI agent can do:
```yaml
safe-outputs:
  add-labels:
    allowed: [bug, enhancement, docs]     # Only these labels
  add-comment: {}                          # Can comment freely
  # No delete, no merge, no deploy        # Explicitly NOT allowed
```

### Tools Allowlist
Restrict which tools the agent can access:
```yaml
tools:
  github:
    toolsets: [issues, labels, pull_requests]
  # No file system, no external APIs
```

### Sandboxing
- Agents run with minimum permissions
- Can only perform actions defined in safe-outputs
- All actions are auditable in the Actions log

---

## Use Cases

| Use Case | Trigger | Agent Does |
|----------|---------|-----------|
| **Issue Triage** | Issue opened | Labels, assigns, comments |
| **PR Review** | PR opened | Analyzes code, posts feedback |
| **Docs Sync** | Code changed | Updates documentation |
| **Self-Healing CI** | CI failure | Diagnoses, proposes fix PR |
| **Release Notes** | Release created | Generates changelog |
| **Stale Cleanup** | Schedule (weekly) | Closes stale issues/PRs |

---

## Security Model

| Control | Implementation |
|---------|---------------|
| **Least privilege** | Explicit permission grants per workflow |
| **Safe outputs** | Agent can only perform listed actions |
| **Audit trail** | All agent actions visible in Actions log |
| **Human oversight** | Changes proposed as PRs, not auto-merged |
| **Network isolation** | Agent sandboxed from external network |

---

## Key Takeaways

1. **Markdown-driven** — Describe intent in natural language, not YAML
2. **Safe by design** — Explicit allowlists for what the agent can do
3. **Compile and deploy** — `gh aw compile` generates standard Actions YAML
4. **Audit everything** — Full transparency in Actions logs
5. **Start small** — Begin with read-only tasks (triage, docs) before write tasks

---

## Next Steps

- 🔗 [Self-Healing CI](self-healing-ci.md) — AI that fixes CI failures
- 🔗 [Continuous AI](continuous-ai.md) — The always-on AI paradigm
