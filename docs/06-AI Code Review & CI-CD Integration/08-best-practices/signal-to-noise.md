# Signal-to-Noise Ratio

> Maximizing actionable feedback and minimizing irrelevant AI review comments.

---

## The Problem

A noisy AI reviewer is worse than no AI reviewer:
- Developers start ignoring all AI comments (boy who cried wolf)
- Review fatigue increases instead of decreasing
- Merge times increase as developers dismiss false positives
- Trust in AI tools erodes

**Target**: <20% false positive rate, >50% suggestion acceptance rate.

---

## Strategies to Improve Signal

### 1. Exclude Auto-Formatted Code
```markdown
# In copilot-instructions.md
Do NOT comment on:
- Import order (handled by ESLint auto-fix)
- Code formatting (handled by Prettier)
- Trailing whitespace or line length
```

### 2. Use "Chill" Mode
Configure AI reviewers for significant issues only:
```yaml
# .coderabbit.yaml
reviews:
  profile: "chill"   # Only significant findings
```

### 3. Exclude Low-Value File Types
```yaml
# Don't review generated or config files
path_filters:
  - "!**/*.generated.ts"
  - "!**/*.lock"
  - "!**/dist/**"
  - "!**/*.min.js"
```

### 4. Severity Filtering
Only surface medium-and-above findings:
```markdown
# Instructions
Only report issues that are:
- Potential bugs or logic errors
- Security vulnerabilities  
- Performance problems
- Missing error handling

Do NOT report:
- Style preferences
- Minor naming suggestions
- Optional optimizations
```

### 5. Deduplicate with Linters
If ESLint already checks for something, don't have AI check it too:
```markdown
# Instructions
The following are already handled by our linter pipeline:
- unused variables, missing semicolons, import order
Do not comment on these.
```

---

## Measuring Signal-to-Noise

| Metric | Formula | Target |
|--------|---------|--------|
| **False positive rate** | Dismissed findings / Total findings | <20% |
| **Acceptance rate** | Applied suggestions / Total suggestions | >50% |
| **Comment density** | AI comments per PR | 3-8 (not 0, not 30) |
| **Resolution rate** | Addressed findings / Total findings | >80% |

---

## Tuning Loop

```
1. Deploy AI review
2. Measure false positive rate (weekly)
3. If >20%: Update instructions to exclude noisy patterns
4. If <5%: AI may be too conservative — loosen rules
5. Repeat weekly for first month, then monthly
```

---

## Key Takeaways

1. **Noisy reviews erode trust** — Better to have fewer, accurate comments
2. **Exclude what linters cover** — No duplication between tools
3. **Use "chill" mode** — Start conservative, add strictness as needed
4. **Measure weekly** — Track false positive and acceptance rates
5. **Tune continuously** — Instructions are living documents
