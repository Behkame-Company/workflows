# Review Instructions

> How to write effective custom instructions that guide AI reviewers to your team's standards.

---

## Where Instructions Go

| Tool | File | Scope |
|------|------|-------|
| **Copilot** | `.github/copilot-instructions.md` | Repository-wide |
| **Copilot (path)** | `src/api/.instructions.md` | Directory-specific |
| **CodeRabbit** | `.coderabbit.yaml` | Repository-wide |
| **PR-Agent** | `.pr_agent.toml` | Repository-wide |

---

## Writing Effective Review Instructions

### Structure
```markdown
## Code Review Rules

### Always Check
- [Critical rules that should always be enforced]

### Never Allow
- [Things that must be blocked]

### Ignore
- [Things the AI should not comment on]

### Per-Area Rules
- [Directory or file-type specific rules]
```

### Example: copilot-instructions.md

```markdown
## Code Review

When reviewing pull requests for this project:

### Security (Always Block)
- Flag any SQL query not using parameterized statements
- Flag hardcoded API keys, tokens, or passwords
- Flag disabled CORS or permissive CORS origins
- Flag missing input validation on API endpoints

### Quality (Require Fix)
- All public functions must have JSDoc/docstring documentation
- Maximum function length: 50 lines
- Maximum cyclomatic complexity: 10
- No console.log in production code (use our logger)

### Testing (Require Fix)  
- New features must include unit tests
- Bug fixes must include regression tests
- Minimum 80% coverage for new files

### Ignore (Don't Comment)
- Formatting issues (Prettier handles this)
- Import order (auto-sorted by ESLint)
- CSS class naming (follows project convention)
- Test file structure (follow existing patterns)
```

---

## Instruction Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| **Too vague** | "Write good code" | Specific rules with examples |
| **Too strict** | "100% coverage required" | Realistic, achievable thresholds |
| **Too many rules** | AI can't prioritize | Focus on 10-15 critical rules |
| **No examples** | AI interprets differently | Include code examples |
| **Never updated** | Rules become stale | Review quarterly |

---

## Key Takeaways

1. **Be specific** — "Flag SQL without parameterization" not "check security"
2. **Include what to ignore** — Prevents noise from auto-formatted code
3. **Separate by severity** — Block, fix, warn, inform
4. **Keep it focused** — 10-15 critical rules, not 100 nice-to-haves
5. **Update quarterly** — Instructions should evolve with your codebase

---

## Next Steps

- 🔗 [Signal-to-Noise](signal-to-noise.md) — Optimizing feedback quality
- 🔗 [Incremental Adoption](incremental-adoption.md) — Phased rollout
