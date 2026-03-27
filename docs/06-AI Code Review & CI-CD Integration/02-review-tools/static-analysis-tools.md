# Static Analysis Tools

> Deterministic code quality and security tools that complement AI reviewers.

---

## The Role of Static Analysis

Static analysis tools find issues by analyzing source code without executing it. Unlike AI reviewers (which are probabilistic), static analyzers are **deterministic** — the same code always produces the same findings.

**Use both**: Static analysis catches what AI misses (data flow, taint analysis), and AI catches what static analysis misses (logic errors, intent mismatch).

---

## Key Tools

### CodeQL (GitHub)
- **Type**: Semantic code analysis
- **Strengths**: Deep data-flow and taint-tracking analysis, custom queries
- **Languages**: JavaScript, TypeScript, Python, Java, C/C++, C#, Go, Ruby, Swift
- **Integration**: Native GitHub Actions, free for open source

```yaml
# .github/workflows/codeql.yml
name: CodeQL Analysis
on: [push, pull_request]
jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with:
          languages: javascript, python
      - uses: github/codeql-action/analyze@v3
```

### Semgrep
- **Type**: Pattern-matching static analysis
- **Strengths**: Fast, customizable rules, wide language support
- **Languages**: 30+ languages
- **Integration**: GitHub Actions, GitLab CI, any CI platform

```yaml
# .github/workflows/semgrep.yml
name: Semgrep
on: [pull_request]
jobs:
  semgrep:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: semgrep/semgrep-action@v1
        with:
          config: >-
            p/default
            p/owasp-top-ten
            p/typescript
```

### ESLint / Ruff / Biome
- **Type**: Linter / code quality
- **Strengths**: Fast, configurable, catches code smells and simple bugs
- **ESLint**: JavaScript/TypeScript ecosystem standard
- **Ruff**: Ultra-fast Python linter (replaces flake8, isort, pydocstyle)
- **Biome**: Fast JS/TS formatter + linter (Prettier + ESLint replacement)

### Dependabot / Renovate
- **Type**: Dependency scanning (SCA)
- **Strengths**: Automated dependency updates, vulnerability alerts
- **Integration**: Native on GitHub (Dependabot) or any platform (Renovate)

---

## The Layered Quality Stack

```
┌─────────────────────────────────────────────────┐
│  Layer 5: HUMAN REVIEW (architecture, logic)    │
├─────────────────────────────────────────────────┤
│  Layer 4: AI REVIEW (context-aware analysis)    │
├─────────────────────────────────────────────────┤
│  Layer 3: SAST (CodeQL, Semgrep — security)     │
├─────────────────────────────────────────────────┤
│  Layer 2: LINTERS (ESLint, Ruff — code quality) │
├─────────────────────────────────────────────────┤
│  Layer 1: FORMATTERS (Prettier, Black — style)  │
└─────────────────────────────────────────────────┘
```

**Each layer runs in CI/CD.** Lower layers are faster and cheaper; higher layers catch deeper issues.

---

## Key Takeaways

1. **Static analysis is deterministic** — Same input, same output, every time
2. **CodeQL for security** — Deep data-flow analysis for vulnerability detection
3. **Semgrep for custom rules** — Fast, flexible pattern matching
4. **Linters for hygiene** — Catch code smells before they reach reviewers
5. **Layer everything** — Formatters → linters → SAST → AI → human

---

## Next Steps

- 🔗 [Security Gates](../05-quality-gates/security-gates-sast-dast.md) — SAST/DAST in pipelines
- 🔗 [Quality Gates](../03-ci-cd-fundamentals/quality-gates.md) — Configuring gate policies
