# CodeRabbit

> An AI-powered PR review tool that provides comprehensive, automated code analysis across GitHub, GitLab, Bitbucket, and Azure DevOps.

---

## What Is CodeRabbit?

CodeRabbit is a dedicated AI code review platform that automatically reviews every pull request, providing multi-layered feedback with line-by-line analysis, severity tagging, and one-click fixes.

Key differentiators:
- **46% bug detection rate** (reported by CodeRabbit; independent verification pending) — among the highest in AI review tools
- **Full codebase context** — analyzes the whole repo, not just the diff
- **40+ built-in linters/scanners** — combines AI with deterministic analysis
- **Incremental reviews** — re-reviews on each new commit

---

## Setup

### 1. Connect Repository
- Sign up at [coderabbit.ai](https://coderabbit.ai)
- Connect via OAuth to GitHub, GitLab, Bitbucket, or Azure DevOps
- Select repositories to enable

### 2. Configure with `.coderabbit.yaml`
```yaml
# .coderabbit.yaml in repository root
reviews:
  profile: "chill"           # "chill" (significant issues) or "assertive" (strict)
  auto_review:
    enabled: true
    drafts: false            # Skip draft PRs
  path_instructions:
    - path: "src/database/**"
      instructions: "Check for N+1 queries and missing indexes"
    - path: "src/api/**"
      instructions: "Verify input validation on all endpoints"
  path_filters:
    - "!**/*.test.ts"        # Skip test files
    - "!docs/**"             # Skip documentation
```

### 3. Review Mode Options

| Mode | Behavior |
|------|----------|
| **Chill** | Only flags significant issues — fewer comments, higher signal |
| **Assertive** | Flags everything including style and minor improvements |

---

## Review Features

### PR Summary
CodeRabbit auto-generates a concise PR summary including:
- What changed and why
- Files affected with change categories
- Potential impact assessment

### Line-by-Line Analysis
Every changed line is analyzed for:
- Logic errors and bugs
- Security vulnerabilities
- Performance issues
- Code smell and anti-patterns

### Severity Tagging
Issues are categorized:
- 🔴 **Critical** — Must fix before merge
- 🟠 **Major** — Should fix, significant quality impact
- 🟡 **Minor** — Nice to fix, minor improvement
- ⚪ **Trivial** — Nitpick, optional

### One-Click Fixes
Many suggestions include apply-ready code changes directly in the PR.

---

## CI/CD Integration

CodeRabbit integrates into your merge workflow:
- **Set commit status**: Block merge until review passes
- **Request changes**: Keep PR in "changes requested" state
- **Auto-approve**: After all issues are resolved

---

## Key Takeaways

1. **High signal** — "Chill" mode reduces noise significantly
2. **Full context** — Analyzes entire codebase, not just the diff
3. **Customizable** — `.coderabbit.yaml` controls behavior per path
4. **Multi-platform** — GitHub, GitLab, Bitbucket, Azure DevOps
5. **Complements Copilot** — Use alongside Copilot for defense-in-depth
