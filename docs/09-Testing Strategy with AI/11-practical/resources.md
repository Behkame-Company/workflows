# Resources & Further Reading

> Curated tools, documentation, articles, and community resources for AI-assisted testing — organized by category with brief descriptions.

---

## Official Documentation

### AI Coding Tools

| Resource | Description |
|----------|-------------|
| [GitHub Copilot Docs](https://docs.github.com/en/copilot) | Official Copilot documentation — setup, custom instructions, workspace agents |
| [Copilot Custom Instructions](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions) | How to configure `.github/copilot-instructions.md` for testing rules |
| [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code) | Claude Code setup, CLAUDE.md configuration, agent workflows |
| [Anthropic Evaluations Guide](https://docs.anthropic.com/en/docs/build-with-claude/develop-tests) | Official guide to building evaluation suites for AI output |
| [OpenAI Best Practices](https://platform.openai.com/docs/guides/prompt-engineering) | Prompt engineering guide applicable to test generation prompts |

### Testing Best Practices

| Resource | Description |
|----------|-------------|
| [Google Testing Blog](https://testing.googleblog.com/) | Industry testing practices from Google's engineering teams |
| [Martin Fowler — Testing](https://martinfowler.com/testing/) | Testing patterns, test pyramid, and testing philosophy |
| [Testing Trophy](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) | Kent C. Dodds' alternative to the testing pyramid |

---

## Testing Frameworks

### JavaScript / TypeScript

| Framework | Type | Description |
|-----------|------|-------------|
| [Jest](https://jestjs.io/) | Unit / Integration | Most popular JS test framework — great AI familiarity |
| [Vitest](https://vitest.dev/) | Unit / Integration | Vite-native, faster alternative to Jest |
| [Playwright](https://playwright.dev/) | E2E | Cross-browser E2E testing with excellent API |
| [Cypress](https://www.cypress.io/) | E2E | Developer-friendly E2E testing with time-travel debugging |
| [React Testing Library](https://testing-library.com/react) | Component | Test React components by behavior, not implementation |
| [MSW (Mock Service Worker)](https://mswjs.io/) | Mocking | API mocking at the network level — works with any framework |
| [fast-check](https://fast-check.dev/) | Property-based | Property-based testing for JavaScript |

### Python

| Framework | Type | Description |
|-----------|------|-------------|
| [pytest](https://docs.pytest.org/) | Unit / Integration | Feature-rich Python testing framework |
| [Hypothesis](https://hypothesis.readthedocs.io/) | Property-based | Property-based testing for Python |
| [pytest-cov](https://pytest-cov.readthedocs.io/) | Coverage | Coverage reporting plugin for pytest |
| [Factory Boy](https://factoryboy.readthedocs.io/) | Fixtures | Test data factories for Python |

### Other Languages

| Framework | Language | Type |
|-----------|----------|------|
| [JUnit 5](https://junit.org/junit5/) | Java | Unit / Integration |
| [xUnit](https://xunit.net/) | C# | Unit / Integration |
| [RSpec](https://rspec.info/) | Ruby | BDD-style testing |
| [Go Testing](https://pkg.go.dev/testing) | Go | Built-in testing package |

---

## Test Quality Tools

### Mutation Testing

| Tool | Language | Description |
|------|----------|-------------|
| [Stryker Mutator](https://stryker-mutator.io/) | JS/TS, C#, Scala | Industry-standard mutation testing — tests your tests |
| [mutmut](https://mutmut.readthedocs.io/) | Python | Mutation testing for Python projects |
| [PIT](https://pitest.org/) | Java | Mutation testing for Java/JVM |
| [cargo-mutants](https://github.com/sourcefrog/cargo-mutants) | Rust | Mutation testing for Rust projects |

### Coverage Analysis

| Tool | Description |
|------|-------------|
| [Istanbul/nyc](https://istanbul.js.org/) | JavaScript code coverage — line, branch, function |
| [c8](https://github.com/bcoe/c8) | Native V8 coverage for Node.js (faster than Istanbul) |
| [Codecov](https://about.codecov.io/) | Coverage tracking and reporting across PRs |
| [Coveralls](https://coveralls.io/) | Test coverage history and badge service |

### Property-Based Testing

| Tool | Language | Description |
|------|----------|-------------|
| [fast-check](https://fast-check.dev/) | JavaScript | Generate random inputs, find edge cases automatically |
| [Hypothesis](https://hypothesis.readthedocs.io/) | Python | The gold standard for property-based testing |
| [jqwik](https://jqwik.net/) | Java | Property-based testing on JUnit 5 |
| [QuickCheck](https://hackage.haskell.org/package/QuickCheck) | Haskell | Original property-based testing library |

---

## Articles & Blog Posts

### AI-Assisted Testing

| Article | Key Takeaway |
|---------|-------------|
| [Anthropic: Building Effective Evals](https://docs.anthropic.com/en/docs/build-with-claude/develop-tests) | How to evaluate AI output systematically using test suites |
| [Write Tests, Not Too Many, Mostly Integration](https://kentcdodds.com/blog/write-tests) | Testing strategy that applies well to AI-generated code |
| [The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) | Foundation for understanding test types and trade-offs |
| [TDD Is Dead, Long Live Testing](https://martinfowler.com/articles/is-tdd-dead/) | Nuanced discussion of TDD applicability |

### Test Quality

| Article | Key Takeaway |
|---------|-------------|
| [Mutation Testing in Practice](https://stryker-mutator.io/docs/General/introduction/) | Why coverage alone is insufficient for test quality |
| [Mocking Is a Code Smell](https://medium.com/javascript-scene/mocking-is-a-code-smell-944a70c90a6a) | When mocking indicates design problems |
| [Flaky Tests at Google](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html) | How Google handles flaky tests at scale |

---

## CI/CD Integration

| Tool | Description |
|------|-------------|
| [GitHub Actions](https://docs.github.com/en/actions) | CI/CD platform — test automation, coverage gates |
| [GitHub Branch Protection](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-a-branch-protection-rule) | Require status checks (tests) before merging |

---

## Community & Discussions

| Resource | Description |
|----------|-------------|
| [GitHub Community Discussions](https://github.com/orgs/community/discussions) | GitHub Copilot discussions and best practices |
| [Anthropic Discord](https://discord.gg/anthropic) | Claude Code community and AI development discussions |
| [r/ExperiencedDevs](https://reddit.com/r/ExperiencedDevs) | Professional developer discussions on testing practices |
| [Testing JavaScript (course)](https://testingjavascript.com/) | Kent C. Dodds' comprehensive testing course |

---

## Cross-References: This Documentation Series

### Fundamentals

| Document | When to Read |
|----------|-------------|
| [Why Testing Matters More with AI](../01-fundamentals/why-testing-matters-more.md) | Starting point — understand why AI changes testing |
| [Testing Pyramid with AI](../01-fundamentals/testing-pyramid-with-ai.md) | How AI affects test type distribution |
| [Test as Specification](../01-fundamentals/test-as-specification.md) | Using tests to constrain AI behavior |

### Workflows

| Document | When to Read |
|----------|-------------|
| [TDD Workflow](../02-tdd-with-ai/tdd-workflow.md) | Implementing TDD with AI agents |
| [Test-First Development](../09-best-practices/test-first-development.md) | Making test-first the default |
| [Test Prompt Patterns](../09-best-practices/prompt-patterns.md) | Writing effective test generation prompts |

### Quality & Maintenance

| Document | When to Read |
|----------|-------------|
| [Coverage Analysis](../05-coverage-and-quality/coverage-analysis.md) | Measuring and improving coverage |
| [Mutation Testing](../05-coverage-and-quality/mutation-testing.md) | Testing your tests |
| [Regression Testing](../08-regression-and-maintenance/regression-testing.md) | Preventing regressions |
| [Test Maintenance](../08-regression-and-maintenance/test-maintenance.md) | Keeping tests healthy |

### Anti-Patterns

| Document | When to Read |
|----------|-------------|
| [Testing to Pass](../10-anti-patterns/testing-to-pass.md) | When AI writes meaningless tests |
| [Over-Mocking](../10-anti-patterns/mock-everything.md) | When mocks hide real bugs |
| [Snapshot Abuse](../10-anti-patterns/snapshot-abuse.md) | When snapshots test nothing |
| [Skipping Verification](../10-anti-patterns/skipping-verification.md) | When AI output goes untested |

### Practical

| Document | When to Read |
|----------|-------------|
| [Templates](templates.md) | Ready-to-use testing templates |
| [Troubleshooting](troubleshooting.md) | Fixing common AI testing problems |
| [FAQ](faq.md) | Quick answers to common questions |

---

## Recommended Reading Order

**For beginners:**
1. [Why Testing Matters More](../01-fundamentals/why-testing-matters-more.md)
2. [TDD Workflow](../02-tdd-with-ai/tdd-workflow.md)
3. [Test-First Development](../09-best-practices/test-first-development.md)
4. [Anti-Patterns Overview](../10-anti-patterns/testing-to-pass.md)

**For experienced developers adopting AI:**
1. [Test as Specification](../01-fundamentals/test-as-specification.md)
2. [Test Prompt Patterns](../09-best-practices/prompt-patterns.md)
3. [Mutation Testing](../05-coverage-and-quality/mutation-testing.md)
4. [Structured Test Tracking](../09-best-practices/structured-test-tracking.md)

**For team leads:**
1. [CI/CD Test Integration](../06-ai-qa-workflows/ci-cd-integration.md)
2. [Regression Testing Strategy](../08-regression-and-maintenance/regression-testing.md)
3. [Templates](templates.md)
4. All four [Anti-Patterns](../10-anti-patterns/testing-to-pass.md)
