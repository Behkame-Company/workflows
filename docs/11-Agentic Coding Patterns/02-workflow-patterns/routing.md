# Routing

> Classify an input and direct it to a specialized follow-up task — achieving separation of concerns where each route has optimized prompts, tools, and models for its category.

---

## The Pattern

Routing uses an LLM (or simple heuristic) to **classify** an input, then directs it to a **specialized handler**. Each handler has its own prompt, tools, and potentially even a different model — optimized for that specific category of work.

```
┌──────────────────────────────────────────────────────────────────┐
│                         ROUTING                                  │
│                                                                  │
│                    ┌──────────────┐                              │
│                    │  Classifier  │                              │
│                    │  (LLM or     │                              │
│                    │   heuristic) │                              │
│                    └──────┬───────┘                              │
│                           │                                      │
│              ┌────────────┼────────────┐                        │
│              ▼            ▼            ▼                        │
│        ┌──────────┐ ┌──────────┐ ┌──────────┐                  │
│        │ Handler  │ │ Handler  │ │ Handler  │                  │
│        │    A     │ │    B     │ │    C     │                  │
│        │(bug fix) │ │(feature) │ │(refactor)│                  │
│        └──────────┘ └──────────┘ └──────────┘                  │
│                                                                  │
│  Each handler has specialized:                                   │
│  • System prompts    • Tool sets    • Model selection           │
└──────────────────────────────────────────────────────────────────┘
```

The key insight: **separation of concerns**. Instead of one massive prompt that tries to handle every type of input, routing lets you build focused, specialized handlers that excel at their specific category.

## When to Use Routing

✅ **Good fit when**:
- Inputs fall into **distinct categories** that benefit from different handling
- **Classification is accurate** — the router can reliably determine the category
- Each category needs different **prompts, tools, or models**

❌ **Poor fit when**:
- Categories overlap heavily (classification will be unreliable)
- All categories need the same handling (routing adds complexity with no benefit)
- The number of categories is very large or constantly changing

## Coding Example 1: Task Type Router

Route coding tasks to specialized handlers:

```typescript
type TaskCategory = "bug_fix" | "feature" | "refactor" | "documentation" | "test";

// Step 1: Classify the task
async function classifyTask(input: string): Promise<TaskCategory> {
  const response = await llm.complete({
    model: "claude-haiku-4-20250514", // Fast, cheap model for classification
    system: `Classify the coding task into exactly one category.
             Respond with ONLY the category name, nothing else.
             
             Categories:
             - bug_fix: Fixing broken behavior, errors, crashes
             - feature: Adding new functionality
             - refactor: Improving code structure without changing behavior
             - documentation: Adding or updating docs, comments, READMEs
             - test: Adding or fixing tests`,
    prompt: input
  });

  return response.trim() as TaskCategory;
}

// Step 2: Route to specialized handler
const handlers: Record<TaskCategory, (input: string) => Promise<string>> = {
  bug_fix: async (input) => {
    return await llm.complete({
      system: `You are a debugging expert. Approach:
               1. Reproduce the issue (identify the exact failing behavior)
               2. Trace to root cause (don't just fix symptoms)
               3. Fix with minimal changes
               4. Add a regression test
               Always explain what caused the bug.`,
      tools: [readFile, search, runTests, editFile],
      prompt: input
    });
  },

  feature: async (input) => {
    return await llm.complete({
      system: `You are a feature developer. Approach:
               1. Understand the requirement fully
               2. Design the interface/API first
               3. Implement with clean, tested code
               4. Update documentation
               Consider edge cases and error handling.`,
      tools: [readFile, writeFile, search, runTests, editFile],
      prompt: input
    });
  },

  refactor: async (input) => {
    return await llm.complete({
      system: `You are a refactoring specialist. Rules:
               1. NO behavior changes — refactoring only
               2. Run tests before AND after every change
               3. Make small, incremental changes
               4. Improve readability, reduce duplication, simplify logic
               If tests fail after a change, revert it.`,
      tools: [readFile, editFile, search, runTests, gitDiff],
      prompt: input
    });
  },

  documentation: async (input) => {
    return await llm.complete({
      model: "claude-haiku-4-20250514", // Docs don't need the biggest model
      system: `You are a technical writer. Write clear, accurate documentation.
               Match the existing documentation style in the project.
               Include code examples where helpful.`,
      tools: [readFile, writeFile, search],
      prompt: input
    });
  },

  test: async (input) => {
    return await llm.complete({
      system: `You are a testing expert. Approach:
               1. Understand what the code does by reading it
               2. Identify edge cases and failure modes
               3. Write tests that cover happy path + edge cases
               4. Use the project's existing test framework and patterns
               Prefer testing behavior over implementation details.`,
      tools: [readFile, writeFile, search, runTests],
      prompt: input
    });
  }
};

// The complete routing workflow
async function routeTask(input: string) {
  const category = await classifyTask(input);
  console.log(`Routed to: ${category}`);
  return await handlers[category](input);
}
```

## Coding Example 2: Model Complexity Router

Route based on task complexity to optimize cost:

```typescript
type ComplexityLevel = "simple" | "moderate" | "complex";

async function routeByComplexity(task: string, context: string) {
  // Classify complexity
  const complexity = await llm.complete({
    model: "claude-haiku-4-20250514",
    system: `Assess the complexity of this coding task. Respond with ONE word:
             - simple: Single file, small change, obvious solution (typo, rename, add comment)
             - moderate: Multiple related changes, clear approach (add validation, update API)
             - complex: Multi-file, architectural decisions, needs exploration (new feature, refactor)`,
    prompt: `Task: ${task}\nContext: ${context}`
  });

  // Route to appropriate model
  const modelMap: Record<ComplexityLevel, string> = {
    simple:   "claude-haiku-4-20250514",      // Fast, cheap — good enough
    moderate: "claude-sonnet-4-20250514",     // Balanced
    complex:  "claude-sonnet-4-20250514"      // Full capability
  };

  const toolMap: Record<ComplexityLevel, Tool[]> = {
    simple:   [readFile, editFile],                          // Minimal tools
    moderate: [readFile, editFile, search, runTests],        // Standard tools
    complex:  [readFile, editFile, search, runTests,         // Full toolkit
               writeFile, gitDiff, gitLog, listDirectory]
  };

  const level = complexity.trim() as ComplexityLevel;

  return await llm.complete({
    model: modelMap[level],
    tools: toolMap[level],
    prompt: task
  });
}
```

## Coding Example 3: Domain Router

Route frontend vs. backend vs. infrastructure tasks to different agent configurations:

```typescript
type Domain = "frontend" | "backend" | "infrastructure" | "database";

const domainConfigs: Record<Domain, AgentConfig> = {
  frontend: {
    systemPrompt: `You are a frontend specialist. You know React, TypeScript, CSS, 
                   and web APIs deeply. Follow component patterns in the codebase.`,
    tools: [readFile, editFile, search, runTests, browserPreview],
    contextFiles: ["src/components/**", "src/styles/**", "package.json"],
    model: "claude-sonnet-4-20250514"
  },

  backend: {
    systemPrompt: `You are a backend specialist. You know Node.js, Express, 
                   API design, and authentication patterns. Follow REST conventions.`,
    tools: [readFile, editFile, search, runTests, curlAPI, readLogs],
    contextFiles: ["src/api/**", "src/middleware/**", "src/models/**"],
    model: "claude-sonnet-4-20250514"
  },

  infrastructure: {
    systemPrompt: `You are a DevOps specialist. You know Docker, CI/CD, 
                   cloud services, and deployment. Be cautious with production configs.`,
    tools: [readFile, editFile, search, dockerCompose, checkCI],
    contextFiles: ["Dockerfile", "docker-compose.yml", ".github/workflows/**"],
    model: "claude-sonnet-4-20250514"
  },

  database: {
    systemPrompt: `You are a database specialist. You know SQL, migrations, 
                   schema design, and query optimization. Always create reversible migrations.`,
    tools: [readFile, editFile, search, runMigration, queryDB],
    contextFiles: ["src/db/**", "migrations/**", "prisma/schema.prisma"],
    model: "claude-sonnet-4-20250514"
  }
};
```

## Classification Strategies

The router's classification step can use different approaches:

### LLM-Based Classification

Best for nuanced categorization:

```typescript
const category = await llm.complete({
  model: "claude-haiku-4-20250514", // Classification doesn't need a large model
  system: "Classify this task. Respond with only the category name.",
  prompt: taskDescription
});
```

### Heuristic Classification

Best for simple, deterministic routing:

```typescript
function classifyByHeuristic(input: string): TaskCategory {
  const lower = input.toLowerCase();
  
  if (lower.match(/\b(bug|fix|broken|crash|error|fail)\b/)) return "bug_fix";
  if (lower.match(/\b(add|create|implement|new|feature)\b/)) return "feature";
  if (lower.match(/\b(refactor|clean|reorganize|simplify)\b/)) return "refactor";
  if (lower.match(/\b(test|spec|coverage|assert)\b/)) return "test";
  if (lower.match(/\b(doc|readme|comment|jsdoc)\b/)) return "documentation";
  
  return "feature"; // Default fallback
}
```

### Hybrid Classification

Use heuristics first, fall back to LLM for ambiguous cases:

```typescript
async function classifyHybrid(input: string): Promise<TaskCategory> {
  const heuristic = classifyByHeuristic(input);
  const confidence = getHeuristicConfidence(input, heuristic);
  
  if (confidence > 0.8) return heuristic;       // High confidence — use heuristic
  return await classifyWithLLM(input);            // Low confidence — ask LLM
}
```

## Design Guidelines

1. **Keep classification fast and cheap** — use small models or heuristics for the router
2. **Ensure categories are mutually exclusive** — overlapping categories cause routing errors
3. **Always have a default/fallback route** — handle unclassifiable inputs gracefully
4. **Log routing decisions** — you need visibility into what's being routed where
5. **Test classification accuracy** — measure how often the router gets it right

```typescript
// Logging routing decisions for observability
async function routeWithLogging(input: string) {
  const category = await classifyTask(input);
  
  console.log(JSON.stringify({
    event: "task_routed",
    input: input.substring(0, 100),
    category,
    timestamp: new Date().toISOString()
  }));
  
  return await handlers[category](input);
}
```

## Next Steps

- [Parallelization](./parallelization.md) — run multiple routes simultaneously
- [Orchestrator-Workers](./orchestrator-workers.md) — when routing needs to be dynamic
- [Prompt Chaining](./prompt-chaining.md) — chain steps within a single route
