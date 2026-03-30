# Prompt Chaining

> Decompose a task into a sequence of LLM calls where each step processes the output of the previous one, with programmatic gates between steps to ensure quality.

---

## The Pattern

Prompt chaining is the simplest workflow pattern. You break a task into a **fixed sequence of steps**, where each LLM call takes the output of the previous call as input. Between steps, you add **programmatic gates** — validation checks that ensure quality before proceeding.

```
┌──────────────────────────────────────────────────────────────────┐
│                     PROMPT CHAINING                              │
│                                                                  │
│  ┌─────────┐     ┌──────┐     ┌─────────┐     ┌──────┐        │
│  │  LLM    │────►│ Gate │────►│  LLM    │────►│ Gate │──► ...  │
│  │ Call 1  │     │  ✓/✗ │     │ Call 2  │     │  ✓/✗ │        │
│  └─────────┘     └──────┘     └─────────┘     └──────┘        │
│                     │                            │              │
│                     ▼ (fail)                     ▼ (fail)      │
│                  Retry or                     Retry or         │
│                  abort                        abort            │
└──────────────────────────────────────────────────────────────────┘
```

The key insight: by decomposing a complex task into smaller steps, each individual LLM call gets a **focused, manageable job**. This trades latency (multiple calls) for higher accuracy (each call is simpler).

## When to Use Prompt Chaining

✅ **Good fit when**:
- The task decomposes naturally into **fixed, sequential subtasks**
- You want to trade latency for **higher accuracy**
- You need **checkpoints** where you can validate or redirect

❌ **Poor fit when**:
- The number of steps isn't predictable (use [orchestrator-workers](./orchestrator-workers.md))
- Steps can run independently (use [parallelization](./parallelization.md))
- The task needs iterative refinement (use [evaluator-optimizer](./evaluator-optimizer.md))

## Coding Example 1: Generate-Validate-Implement

A three-step chain for implementing code changes:

```typescript
interface ChainResult<T> {
  success: boolean;
  data?: T;
  error?: string;
}

// Step 1: Analyze the codebase and generate an outline
async function analyzeAndOutline(task: string, codebase: string): Promise<ChainResult<string>> {
  const response = await llm.complete({
    system: "You are a senior software architect. Analyze codebases and produce implementation outlines.",
    prompt: `Task: ${task}
    
Relevant code:
${codebase}

Produce a detailed outline of the changes needed:
1. Which files to modify and why
2. What new files to create  
3. The order of changes
4. Any dependencies between changes

Output format: structured markdown outline`
  });

  return { success: true, data: response };
}

// Gate 1: Validate the outline
function validateOutline(outline: string): ChainResult<string> {
  const hasFiles = outline.includes("file") || outline.includes("File");
  const hasSteps = /\d+\.\s/.test(outline);
  const notTooLong = outline.split("\n").length < 200;

  if (!hasFiles || !hasSteps) {
    return { success: false, error: "Outline missing file references or numbered steps" };
  }
  if (!notTooLong) {
    return { success: false, error: "Outline too complex — break into smaller tasks" };
  }

  return { success: true, data: outline };
}

// Step 2: Generate code from the validated outline
async function generateCode(outline: string, existingCode: string): Promise<ChainResult<string>> {
  const response = await llm.complete({
    system: "You are an expert programmer. Implement code changes precisely following the given outline.",
    prompt: `Implementation outline:
${outline}

Existing code:
${existingCode}

Implement all changes described in the outline. For each file, show the complete updated content.`
  });

  return { success: true, data: response };
}

// Gate 2: Validate the generated code
async function validateCode(code: string): Promise<ChainResult<string>> {
  // Run programmatic checks
  const syntaxValid = await checkSyntax(code);
  if (!syntaxValid) {
    return { success: false, error: "Generated code has syntax errors" };
  }

  const hasTests = code.includes("test(") || code.includes("it(") || code.includes("describe(");
  if (!hasTests) {
    return { success: false, error: "No tests included — add test coverage" };
  }

  return { success: true, data: code };
}

// The chain: analyze → gate → generate → gate → apply
async function promptChain(task: string) {
  const codebase = await readRelevantFiles(task);

  // Step 1: Analyze and outline
  const outline = await analyzeAndOutline(task, codebase);
  if (!outline.success) return outline;

  // Gate 1: Validate outline
  const validOutline = validateOutline(outline.data!);
  if (!validOutline.success) {
    console.log(`Gate 1 failed: ${validOutline.error}. Retrying...`);
    // Could retry with feedback or abort
    return validOutline;
  }

  // Step 2: Generate code
  const code = await generateCode(validOutline.data!, codebase);
  if (!code.success) return code;

  // Gate 2: Validate code
  const validCode = await validateCode(code.data!);
  if (!validCode.success) {
    console.log(`Gate 2 failed: ${validCode.error}. Retrying with feedback...`);
    // Retry step 2 with the error as feedback
    return validCode;
  }

  return validCode;
}
```

## Coding Example 2: Document → Verify → Publish

A chain for generating and validating documentation:

```typescript
async function documentationChain(sourceFile: string) {
  const source = await readFile(sourceFile);

  // Step 1: Generate documentation
  const docs = await llm.complete(
    `Generate JSDoc comments for all exported functions in:\n${source}`
  );

  // Gate 1: Every exported function has a doc comment
  const exportedFns = source.match(/export\s+(async\s+)?function\s+\w+/g) || [];
  const docComments = docs.match(/\/\*\*[\s\S]*?\*\//g) || [];

  if (docComments.length < exportedFns.length) {
    // Retry with explicit instruction
    const retry = await llm.complete(
      `You missed some functions. These exports need docs: ${exportedFns.join(", ")}\n\n${source}`
    );
    return retry;
  }

  // Step 2: Verify accuracy of generated docs
  const verification = await llm.complete(
    `Verify these JSDoc comments are accurate. Check parameter types, return types, 
     and descriptions match the actual implementation. Flag any inaccuracies:
     
     Source: ${source}
     Generated docs: ${docs}`
  );

  // Gate 2: No inaccuracies flagged
  if (verification.toLowerCase().includes("inaccurate") || 
      verification.toLowerCase().includes("incorrect")) {
    // Fix the inaccuracies
    const fixed = await llm.complete(
      `Fix these documentation inaccuracies:\n${verification}\n\nOriginal docs:\n${docs}`
    );
    return fixed;
  }

  return docs;
}
```

## The Gate Pattern

Gates are what make prompt chaining reliable. They're programmatic checkpoints between LLM calls:

```
┌────────────────────────────────────────────────────┐
│                  GATE TYPES                         │
│                                                     │
│  Format Gate:   Does output match expected format?  │
│  Content Gate:  Does output contain required info?  │
│  Quality Gate:  Does output pass quality checks?    │
│  Safety Gate:   Does output meet safety criteria?   │
│  Test Gate:     Does generated code compile/pass?   │
└────────────────────────────────────────────────────┘
```

```typescript
// Common gate implementations
const gates = {
  // Format: output matches expected structure
  format: (output: string, schema: z.ZodSchema) => {
    const parsed = schema.safeParse(JSON.parse(output));
    return parsed.success ? { pass: true, data: parsed.data } 
                          : { pass: false, error: parsed.error };
  },

  // Content: required elements are present
  content: (output: string, required: string[]) => {
    const missing = required.filter(r => !output.includes(r));
    return missing.length === 0 ? { pass: true } 
                                : { pass: false, error: `Missing: ${missing.join(", ")}` };
  },

  // Test: code compiles and tests pass
  test: async (code: string) => {
    await writeFile("temp.ts", code);
    const result = await exec("npx tsc --noEmit temp.ts && npm test");
    return result.exitCode === 0 ? { pass: true } 
                                  : { pass: false, error: result.stderr };
  }
};
```

## Handling Gate Failures

When a gate fails, you have three options:

```typescript
async function handleGateFailure(
  step: () => Promise<string>,
  gate: (output: string) => ChainResult<string>,
  maxRetries: number = 2
) {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    const output = await step();
    const result = gate(output);

    if (result.success) return result;

    if (attempt < maxRetries) {
      console.log(`Gate failed (attempt ${attempt + 1}): ${result.error}`);
      // Option 1: Retry the step (maybe with modified prompt)
      continue;
    }
    
    // Option 2: Abort the chain
    throw new Error(`Chain aborted: ${result.error}`);
  }

  // Option 3: Fall back to a simpler approach
  return await fallbackApproach();
}
```

## Design Guidelines

1. **Each step should have a single, focused job** — don't try to do too much in one call
2. **Gates should be programmatic, not LLM-based** — use code to validate format, presence of required content, syntax
3. **Pass only relevant context forward** — don't dump the entire conversation into each step
4. **Plan for failure** — every gate needs a failure path (retry, abort, or fallback)
5. **Keep chains short** — 2-4 steps is typical; longer chains compound errors
