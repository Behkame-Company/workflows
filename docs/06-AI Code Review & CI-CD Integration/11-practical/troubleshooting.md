# Troubleshooting

> Common problems with AI code review and CI/CD pipelines, and how to fix them.

---

## AI Review Issues

### AI review not triggering
**Symptoms**: PR opened but no AI review comments appear.
**Causes & Fixes**:
| Cause | Fix |
|-------|-----|
| Copilot review not enabled | Settings → Rules → New ruleset → Enable Copilot review |
| PR is a draft | Copilot doesn't review drafts by default |
| Repo is private (free plan) | Copilot review requires paid plan for private repos |
| PR too large | Some tools skip PRs with >500 changed files |
| Rate limited | Check API usage; wait and retry |

### Too many false positives
**Symptoms**: AI flags issues that aren't real problems.
**Fixes**:
1. Add custom instructions excluding noisy patterns
2. Switch to "chill" mode (CodeRabbit)
3. Exclude auto-formatted/generated files
4. Specify what linters already cover

### AI and linter conflict
**Symptoms**: AI suggests one style, linter enforces another.
**Fix**: Tell AI to defer to linter on formatting/style:
```markdown
# copilot-instructions.md
Do not comment on formatting, import order, or style.
These are handled by ESLint and Prettier.
```

### AI suggestions break code
**Symptoms**: Applying AI suggestion causes tests to fail.
**Fix**: Always run CI after applying suggestions. Never batch-apply without testing.

---

## CI/CD Issues

### Pipeline too slow
**Symptoms**: CI takes >15 minutes.
**Fixes**:
- Parallelize independent jobs (`needs` only where truly needed)
- Cache dependencies (`actions/cache` or built-in cache in `setup-node`)
- Use `concurrency` to cancel stale runs
- Only run expensive jobs on changed paths (`paths:` filter)

### Flaky tests
**Symptoms**: Tests pass sometimes, fail others.
**Fixes**:
- Identify flaky tests with `--retry` flag
- Quarantine known flaky tests
- Fix root causes: race conditions, time-dependent assertions, shared state
- Never blame AI-generated tests without investigating

### Security scan false positives
**Symptoms**: CodeQL/Semgrep flags code that's actually safe.
**Fixes**:
- Add inline suppression with comment explaining why
- Configure tool to exclude false positive patterns
- Use `.semgrepignore` or CodeQL query filters

### Self-healing CI creating bad fixes
**Symptoms**: AI auto-fix PR makes tests pass but introduces subtle bugs.
**Fix**: Never auto-merge AI fix PRs. Always require human review. Add mutation testing for generated fixes.

---

## Integration Issues

### Multiple AI reviewers conflicting
**Symptoms**: Copilot says one thing, CodeRabbit says another.
**Fix**: Use one primary AI reviewer. If using multiple, assign different responsibilities:
- Copilot: Code quality and patterns
- CodeRabbit: Security and compliance
- Don't overlap responsibilities

### GitHub Actions permissions errors
**Symptoms**: `Resource not accessible by integration`
**Fix**: Add required permissions to workflow:
```yaml
permissions:
  pull-requests: write    # For PR comments
  contents: read          # For checkout
  security-events: write  # For CodeQL
```

---

## Key Takeaways

1. **Most issues are configuration problems** — Not fundamental limitations
2. **Start with logs** — GitHub Actions logs show exactly what failed
3. **Isolate the problem** — Is it AI, CI, or integration?
4. **Check permissions** — Most integration failures are permission issues
5. **One AI reviewer is enough** — Multiple reviewers cause conflicts
