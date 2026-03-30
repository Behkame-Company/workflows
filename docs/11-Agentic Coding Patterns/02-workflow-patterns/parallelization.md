# Parallelization

> Run multiple LLM calls simultaneously — either independent subtasks (sectioning) or the same task from multiple perspectives (voting) — to reduce latency and improve coverage.

---

## The Pattern

Parallelization has two distinct variations:

1. **Sectioning**: Split a task into independent subtasks and process them in parallel
2. **Voting**: Run the same task multiple times (possibly with different prompts) and aggregate the results

```
┌──────────────────────────────────────────────────────────────────┐
│                     PARALLELIZATION                              │
│                                                                  │
│  SECTIONING                         VOTING                      │
│  (independent subtasks)             (multiple perspectives)      │
│                                                                  │
│       ┌──────────┐                      ┌──────────┐            │
│       │ Subtask  │                      │ Attempt  │            │
│    ┌──│    A     │──┐               ┌──│    1     │──┐         │
│    │  └──────────┘  │               │  └──────────┘  │         │
│    │  ┌──────────┐  │               │  ┌──────────┐  │         │
│  Input│ Subtask  │  ├─► Combine   Input│ Attempt  │  ├─► Agg.  │
│    │  │    B     │  │               │  │    2     │  │         │
│    │  └──────────┘  │               │  └──────────┘  │         │
│    │  ┌──────────┐  │               │  ┌──────────┐  │         │
│    └──│ Subtask  │──┘               └──│ Attempt  │──┘         │
│       │    C     │                      │    3     │            │
│       └──────────┘                      └──────────┘            │
│                                                                  │
│  Each parallel call gets                Multiple perspectives    │
│  focused attention on                   on the same problem      │
│  one specific aspect                    catch different issues   │
└──────────────────────────────────────────────────────────────────┘
```

The key insight: each parallel LLM call gets **focused attention** on one aspect, rather than trying to juggle everything in a single call.

## When to Use Parallelization

✅ **Good fit when**:
- **Sectioning**: Subtasks are genuinely **independent** — no dependencies between them
- **Voting**: You need **diverse perspectives** or **higher reliability** on a critical judgment
- Latency matters and tasks can run concurrently

❌ **Poor fit when**:
- Subtasks depend on each other's output (use [prompt chaining](./prompt-chaining.md))
- The number/nature of subtasks isn't known upfront (use [orchestrator-workers](./orchestrator-workers.md))
- A single focused call is sufficient

## Sectioning: Independent Subtasks

### Example 1: Parallel Feature Implementation

When building a feature that spans multiple independent components:

```typescript
interface SubtaskResult {
  component: string;
  code: string;
  tests: string;
}

async function parallelImplementation(featureSpec: string) {
  // Analyze the feature to identify independent components
  const components = await llm.complete({
    prompt: `Break this feature into independent implementation components:
             ${featureSpec}
             
             Return a JSON array of components, each with:
             - name: component identifier
             - description: what to implement
             - files: which files to create/modify`
  });

  const componentList = JSON.parse(components);

  // Implement all components in parallel
  const results = await Promise.all(
    componentList.map(async (component: any): Promise<SubtaskResult> => {
      const implementation = await llm.complete({
        system: `You are implementing a single component of a larger feature.
                 Focus ONLY on your component. Do not modify files outside your scope.`,
        prompt: `Implement this component:
                 Name: ${component.name}
                 Description: ${component.description}
                 Files: ${component.files.join(", ")}
                 
                 Feature context: ${featureSpec}`,
        tools: [readFile, writeFile, editFile]
      });

      return {
        component: component.name,
        code: implementation,
        tests: "" // Tests added in next step
      };
    })
  );

  return results;
}
```

### Example 2: Guardrails Alongside Generation

Run safety checks in parallel with content generation:

```typescript
async function generateWithGuardrails(task: string) {
  // Run generation and safety checks in parallel
  const [generatedCode, securityCheck, licenseCheck] = await Promise.all([
    // Main generation
    llm.complete({
      system: "You are an expert programmer. Implement the requested feature.",
      prompt: task,
      tools: [readFile, writeFile, search]
    }),

    // Security guardrail (runs simultaneously)
    llm.complete({
      system: `You are a security auditor. Analyze the task for potential security risks.
               Flag: SQL injection, XSS, path traversal, hardcoded secrets, 
               insecure dependencies, missing input validation.
               Respond with JSON: { "risks": [...], "blocked": boolean }`,
      prompt: `Analyze security risks in this task:\n${task}`
    }),

    // License guardrail (runs simultaneously)
    llm.complete({
      system: `You are a compliance checker. Check if the task might involve
               copying copyrighted code or using restricted licenses.
               Respond with JSON: { "concerns": [...], "blocked": boolean }`,
      prompt: `Check compliance concerns:\n${task}`
    })
  ]);

  // Evaluate guardrail results before returning
  const security = JSON.parse(securityCheck);
  const license = JSON.parse(licenseCheck);

  if (security.blocked) {
    return { blocked: true, reason: "Security risks detected", details: security.risks };
  }
  if (license.blocked) {
    return { blocked: true, reason: "License concerns", details: license.concerns };
  }

  return {
    blocked: false,
    code: generatedCode,
    warnings: [...security.risks, ...license.concerns]
  };
}
```

### Example 3: Parallel Code Analysis

Analyze different aspects of code simultaneously:

```typescript
async function comprehensiveCodeReview(filePath: string) {
  const code = await readFile(filePath);

  // Run all review dimensions in parallel
  const [
    correctness,
    performance,
    security,
    readability,
    testCoverage
  ] = await Promise.all([
    // Each reviewer focuses on ONE dimension
    llm.complete({
      system: "You are reviewing code ONLY for correctness. Look for bugs, logic errors, off-by-one errors, null pointer issues, race conditions. Ignore style.",
      prompt: `Review for correctness:\n${code}`
    }),
    llm.complete({
      system: "You are reviewing code ONLY for performance. Look for N+1 queries, unnecessary allocations, missing caching opportunities, algorithmic complexity issues. Ignore style.",
      prompt: `Review for performance:\n${code}`
    }),
    llm.complete({
      system: "You are reviewing code ONLY for security. Look for injection, XSS, CSRF, auth bypass, data exposure, insecure crypto. Ignore style.",
      prompt: `Review for security:\n${code}`
    }),
    llm.complete({
      system: "You are reviewing code ONLY for readability. Look for unclear naming, missing documentation on complex logic, overly clever code, inconsistent patterns. Ignore performance.",
      prompt: `Review for readability:\n${code}`
    }),
    llm.complete({
      system: "You are assessing test coverage. Identify untested code paths, missing edge cases, and critical functionality without tests.",
      prompt: `Assess test needs:\n${code}`
    })
  ]);

  return { correctness, performance, security, readability, testCoverage };
}
```

## Voting: Multiple Perspectives

### Example 1: Vulnerability Scanning

Multiple scanning perspectives catch different issues:

```typescript
async function multiAngleSecurityScan(code: string) {
  const scanPrompts = [
    {
      angle: "attacker",
      system: "You are a penetration tester. Think like an attacker — how would you exploit this code? What inputs would break it?"
    },
    {
      angle: "auditor",
      system: "You are a security auditor following OWASP guidelines. Systematically check each OWASP Top 10 category."
    },
    {
      angle: "defender",
      system: "You are a security engineer reviewing code for production deployment. What security controls are missing? What needs hardening?"
    }
  ];

  // Run all three perspectives in parallel
  const results = await Promise.all(
    scanPrompts.map(async ({ angle, system }) => {
      const findings = await llm.complete({
        system,
        prompt: `Analyze this code for security issues:\n${code}\n\nReturn JSON: { "findings": [{ "severity": "high|medium|low", "issue": "...", "location": "..." }] }`
      });
      return { angle, findings: JSON.parse(findings).findings };
    })
  );

  // Aggregate: deduplicate and rank by consensus
  return aggregateFindings(results);
}

function aggregateFindings(results: any[]) {
  const allFindings = results.flatMap(r => 
    r.findings.map((f: any) => ({ ...f, source: r.angle }))
  );

  // Issues flagged by multiple angles are higher priority
  const grouped = groupBySimilarity(allFindings);
  return grouped.sort((a, b) => b.sources.length - a.sources.length);
}
```

### Example 2: Code Review Consensus

Multiple review passes with different prompts provide more thorough coverage:

```typescript
async function consensusCodeReview(diff: string) {
  const reviews = await Promise.all([
    // Reviewer 1: Focus on bugs
    llm.complete({
      system: "Review ONLY for bugs. Would this code work correctly in all cases?",
      prompt: diff
    }),
    // Reviewer 2: Focus on design
    llm.complete({
      system: "Review ONLY for design. Is this the right approach? Are there better patterns?",
      prompt: diff
    }),
    // Reviewer 3: Focus on maintainability
    llm.complete({
      system: "Review ONLY for maintainability. Will this be easy to modify in 6 months?",
      prompt: diff
    })
  ]);

  // Synthesize reviews into a single coherent review
  const synthesis = await llm.complete({
    system: "Synthesize these three code reviews into one cohesive review. Remove duplicates, prioritize by severity, keep it actionable.",
    prompt: reviews.map((r, i) => `Review ${i + 1}:\n${r}`).join("\n\n---\n\n")
  });

  return synthesis;
}
```

## Implementation Patterns

### Managing Parallel Execution

```typescript
// Pattern: Parallel with concurrency limit
async function parallelWithLimit<T>(
  tasks: (() => Promise<T>)[],
  concurrency: number = 5
): Promise<T[]> {
  const results: T[] = [];
  const executing: Promise<void>[] = [];

  for (const task of tasks) {
    const p = task().then(result => { results.push(result); });
    executing.push(p);

    if (executing.length >= concurrency) {
      await Promise.race(executing);
      executing.splice(executing.findIndex(e => e === p), 1);
    }
  }

  await Promise.all(executing);
  return results;
}

// Usage: process 20 files, max 5 concurrent
await parallelWithLimit(
  files.map(f => () => reviewFile(f)),
  5
);
```

### Combining Parallel Results

```typescript
// Pattern: Combine parallel results with conflict resolution
async function combineResults(results: SubtaskResult[]) {
  // Check for conflicts (e.g., two subtasks modifying the same file)
  const fileConflicts = findFileConflicts(results);

  if (fileConflicts.length > 0) {
    // Resolve conflicts with a dedicated LLM call
    const resolved = await llm.complete({
      system: "Merge these conflicting changes into a single coherent result.",
      prompt: `Conflicts:\n${JSON.stringify(fileConflicts)}`
    });
    return resolved;
  }

  // No conflicts — simple merge
  return results.flatMap(r => r.changes);
}
```

## Design Guidelines

1. **Verify independence** — parallel subtasks must not depend on each other's output
2. **Set concurrency limits** — don't fire 100 parallel LLM calls; rate limits and costs add up
3. **Plan for partial failure** — if one parallel branch fails, decide whether to retry, skip, or abort all
4. **Aggregate thoughtfully** — combining results is as important as generating them
5. **Use focused prompts** — the whole point of parallelization is that each call focuses on one thing
