# Evaluator-Optimizer

> One LLM generates a response, another evaluates it against clear criteria, and the generator refines based on feedback — looping until quality standards are met.

---

## The Pattern

The evaluator-optimizer pattern creates an **iterative refinement loop** between two roles:

1. **Generator**: Produces an output (code, text, plan, etc.)
2. **Evaluator**: Assesses the output against defined criteria and provides feedback
3. The generator **refines** its output based on the evaluator's feedback
4. The loop continues until the evaluator passes the output (or a max iteration limit is hit)

```
┌──────────────────────────────────────────────────────────────────┐
│                   EVALUATOR-OPTIMIZER                             │
│                                                                  │
│  ┌─────────────┐         ┌─────────────┐                        │
│  │  GENERATOR  │────────►│  EVALUATOR  │                        │
│  │             │         │             │                        │
│  │ Produces    │◄────────│ Assesses    │                        │
│  │ output      │ feedback│ against     │                        │
│  │             │         │ criteria    │                        │
│  └─────────────┘         └──────┬──────┘                        │
│        ▲                        │                                │
│        │     ┌──────────────────┤                                │
│        │     │                  │                                │
│        │     ▼                  ▼                                │
│        │  ┌──────┐         ┌──────┐                              │
│        └──│ FAIL │         │ PASS │──────► Final Output         │
│  (refine) │      │         │      │                              │
│           └──────┘         └──────┘                              │
│                                                                  │
│  Loop continues until:                                           │
│  • Evaluator passes ✓                                            │
│  • Max iterations reached                                        │
│  • No meaningful improvement between iterations                  │
└──────────────────────────────────────────────────────────────────┘
```

This is the **"self-critique" pattern** — and it works because LLMs are often better at evaluating output than generating perfect output on the first try.

## When to Use Evaluator-Optimizer

Two signs that this pattern is a good fit (from Anthropic):

1. **LLM responses can be demonstrably improved when a human articulates feedback** — if pointing out specific issues leads to better output, an LLM evaluator can provide similar feedback
2. **The LLM can provide such feedback** — the evaluation criteria are clear enough that an LLM can assess quality

✅ **Good fit when**:
- Clear, articulable **quality criteria** exist
- Iterative refinement **measurably improves** results
- The task has a definition of "good enough" (a stopping condition)

❌ **Poor fit when**:
- Quality is subjective and can't be programmatically assessed
- First-pass output is almost always good enough
- The cost of iteration outweighs the quality improvement

## Coding Example 1: Generate → Review → Refine

The most common application — code generation with quality review:

```typescript
interface EvalResult {
  passed: boolean;
  score: number;        // 0-10
  issues: string[];     // Specific problems found
  suggestions: string[];// How to fix them
}

async function generateWithReview(spec: string, maxIterations: number = 3) {
  let code = "";
  let feedback = "";

  for (let i = 0; i < maxIterations; i++) {
    // GENERATE (or refine based on feedback)
    code = await llm.complete({
      system: `You are an expert programmer. Write clean, tested, production-quality code.
               ${feedback ? "IMPORTANT: Address all feedback from the previous review." : ""}`,
      prompt: i === 0
        ? `Implement the following specification:\n${spec}`
        : `Improve this code based on review feedback:
           
           Current code:
           ${code}
           
           Review feedback:
           ${feedback}
           
           Fix all issues and return the complete improved code.`
    });

    // EVALUATE
    const evaluation: EvalResult = await evaluate(code, spec);

    console.log(`Iteration ${i + 1}: Score ${evaluation.score}/10, ${evaluation.issues.length} issues`);

    if (evaluation.passed) {
      console.log(`✅ Passed after ${i + 1} iteration(s)`);
      return { code, iterations: i + 1, finalScore: evaluation.score };
    }

    // Prepare feedback for next iteration
    feedback = [
      "Issues found:",
      ...evaluation.issues.map((issue, idx) => `${idx + 1}. ${issue}`),
      "",
      "Suggestions:",
      ...evaluation.suggestions.map((s, idx) => `${idx + 1}. ${s}`)
    ].join("\n");
  }

  console.log(`⚠️ Max iterations reached. Returning best effort.`);
  return { code, iterations: maxIterations, finalScore: -1 };
}

async function evaluate(code: string, spec: string): Promise<EvalResult> {
  const response = await llm.complete({
    system: `You are a rigorous code reviewer. Evaluate code against the specification.
             
             Evaluation criteria:
             1. Correctness: Does it implement the spec completely?
             2. Error handling: Are edge cases and errors handled?
             3. Type safety: Are types correct and complete?
             4. Test coverage: Are critical paths tested?
             5. Code quality: Is it readable, maintainable, well-structured?
             
             Score 0-10. Pass threshold: 8.
             Be specific about issues — vague feedback is useless.`,
    prompt: `Specification:
${spec}

Code to evaluate:
${code}

Return JSON: { "passed": boolean, "score": number, "issues": [...], "suggestions": [...] }`
  });

  return JSON.parse(response);
}
```

## Coding Example 2: Test Coverage Loop

Write tests, check coverage, add missing tests:

```typescript
async function achieveTestCoverage(sourceFile: string, targetCoverage: number = 80) {
  const sourceCode = await readFile(sourceFile);
  let testCode = "";
  
  for (let i = 0; i < 4; i++) {
    // GENERATE: Write or improve tests
    testCode = await llm.complete({
      system: "You are a testing expert. Write comprehensive tests using the project's test framework.",
      prompt: i === 0
        ? `Write tests for this source file:\n${sourceCode}`
        : `Improve test coverage. Current tests:\n${testCode}\n\nSource:\n${sourceCode}\n\nMissing coverage:\n${await getCoverageReport(sourceFile)}`
    });

    // Write tests and run coverage
    await writeFile(sourceFile.replace(".ts", ".test.ts"), testCode);
    const coverage = await runCoverageCheck(sourceFile);

    console.log(`Iteration ${i + 1}: ${coverage.percent}% coverage`);

    // EVALUATE: Check if coverage target is met
    if (coverage.percent >= targetCoverage) {
      console.log(`✅ Coverage target met: ${coverage.percent}%`);
      return { testCode, coverage: coverage.percent, iterations: i + 1 };
    }

    // Feedback: which lines are uncovered
    console.log(`Missing coverage on lines: ${coverage.uncoveredLines.join(", ")}`);
  }

  return { testCode, coverage: -1, iterations: 4 };
}
```

## Coding Example 3: Implement → Test → Fix Loop

Generate code that must pass a specific test suite:

```typescript
async function implementUntilTestsPass(spec: string, testFile: string) {
  const tests = await readFile(testFile);
  let implementation = "";

  for (let i = 0; i < 5; i++) {
    // GENERATE: Implement (or fix based on test failures)
    implementation = await llm.complete({
      system: `You are an expert developer doing test-driven development.
               The tests are already written — implement code that makes them pass.`,
      prompt: i === 0
        ? `Implement code to pass these tests:\n${tests}\n\nSpec: ${spec}`
        : `Fix the implementation to pass failing tests.
           
           Current implementation:
           ${implementation}
           
           Test results:
           ${await getTestOutput(testFile)}
           
           Fix the issues and return the complete corrected implementation.`
    });

    await writeFile(spec, implementation);

    // EVALUATE: Run the actual tests
    const testResult = await runTests(testFile);

    if (testResult.allPassed) {
      console.log(`✅ All ${testResult.total} tests pass after ${i + 1} iteration(s)`);
      return { implementation, iterations: i + 1 };
    }

    console.log(`Iteration ${i + 1}: ${testResult.passed}/${testResult.total} tests pass`);
    // Loop continues with test failure output as feedback
  }

  throw new Error("Could not pass all tests within iteration limit");
}
```

## Evaluation Strategies

The evaluator can use different approaches depending on what you're checking:

### LLM-as-Judge

Use when evaluation requires understanding and judgment:

```typescript
async function llmEvaluator(code: string, criteria: string[]): Promise<EvalResult> {
  const response = await llm.complete({
    system: "You are a strict code evaluator. Assess against each criterion independently.",
    prompt: `Code:\n${code}\n\nCriteria:\n${criteria.map((c, i) => `${i + 1}. ${c}`).join("\n")}`
  });
  return JSON.parse(response);
}
```

### Programmatic Evaluation

Use when criteria are objective and measurable:

```typescript
async function programmaticEvaluator(code: string): Promise<EvalResult> {
  const issues: string[] = [];

  // Type check
  const typeCheck = await exec("npx tsc --noEmit");
  if (typeCheck.exitCode !== 0) issues.push(`Type errors: ${typeCheck.stderr}`);

  // Lint
  const lint = await exec("npx eslint --format json");
  if (lint.exitCode !== 0) issues.push(`Lint errors: ${lint.stdout}`);

  // Tests
  const tests = await exec("npm test -- --reporter json");
  if (tests.exitCode !== 0) issues.push(`Test failures: ${tests.stderr}`);

  return {
    passed: issues.length === 0,
    score: Math.max(0, 10 - issues.length * 2),
    issues,
    suggestions: issues.map(i => `Fix: ${i}`)
  };
}
```

### Hybrid Evaluation

Combine programmatic checks with LLM judgment:

```typescript
async function hybridEvaluator(code: string, spec: string): Promise<EvalResult> {
  // Programmatic checks first (fast, deterministic)
  const progResult = await programmaticEvaluator(code);
  if (!progResult.passed) return progResult;  // Fail fast on objective issues

  // LLM evaluation for subjective quality (only if programmatic checks pass)
  const llmResult = await llmEvaluator(code, [
    "Does the code fully implement the specification?",
    "Are edge cases handled appropriately?",
    "Is the code maintainable and well-structured?"
  ]);

  return llmResult;
}
```

## Preventing Infinite Loops

Always include safeguards: **max iterations**, **stagnation detection** (stop if score doesn't improve), and **timeouts**. For comprehensive loop prevention strategies, detection patterns, and escape mechanisms, see [Infinite Agent Loops](../10-anti-patterns/infinite-loops.md).

```typescript
// Minimal safeguard example
async function safeEvalOptLoop(generate, evaluate, maxIter = 5) {
  let output = await generate("");
  let lastScore = 0;

  for (let i = 0; i < maxIter; i++) {
    const result = await evaluate(output);
    if (result.passed) return output;
    if (result.score <= lastScore + 0.5) return output; // Stagnation
    lastScore = result.score;
    output = await generate(result.issues.join("\n"));
  }
  return output;
}
```
```

## Design Guidelines

1. **Make evaluation criteria explicit and measurable** — vague criteria lead to vague feedback
2. **Use programmatic evaluation where possible** — it's faster, cheaper, and deterministic
3. **Always cap iterations** — without limits, loops can run indefinitely
4. **Track improvement across iterations** — stop if quality plateaus
5. **Make feedback specific and actionable** — "improve the code" is useless; "line 42 doesn't handle null input" is actionable
6. **Consider cost** — each iteration is another LLM call; ensure the improvement justifies the cost
