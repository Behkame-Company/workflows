# Reflection and Self-Critique

> Agents that evaluate their own work — generating output, critiquing it against criteria, and refining until quality standards are met.

---

## The Pattern

Reflection adds a **self-evaluation loop** to agent output. Instead of producing a result and moving on, the agent:

1. **Generate** — produce an initial output
2. **Review** — critique the output against specific criteria
3. **Refine** — improve based on the critique
4. **Repeat** — until quality standards are met or improvement plateaus

```
┌──────────────────────────────────────────────────────────────────┐
│                  REFLECTION PATTERN                               │
│                                                                  │
│  ┌───────────────┐                                              │
│  │   GENERATE    │                                              │
│  │               │                                              │
│  │  Produce      │                                              │
│  │  initial      │                                              │
│  │  output       │                                              │
│  └───────┬───────┘                                              │
│          │                                                       │
│          ▼                                                       │
│  ┌───────────────┐     ┌───────────────┐                        │
│  │   REVIEW      │     │   REFINE      │                        │
│  │               │     │               │                        │
│  │  Critique     │────►│  Improve      │                        │
│  │  against      │     │  based on     │                        │
│  │  criteria     │     │  critique     │                        │
│  └───────┬───────┘     └───────┬───────┘                        │
│          │                     │                                  │
│          │  meets              │  not yet                        │
│          │  standards          │                                  │
│          ▼                     └──────► back to REVIEW           │
│  ┌───────────────┐                                              │
│  │   ACCEPT      │                                              │
│  │               │                                              │
│  │  Output meets │                                              │
│  │  quality bar  │                                              │
│  └───────────────┘                                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Reflection in Coding

When coding agents reflect, they review their own code for issues before declaring the task complete:

```
┌────────────────────────────────────────────────────────────────┐
│              CODE REFLECTION CYCLE                               │
│                                                                │
│  GENERATE:  Write the implementation                           │
│      │                                                         │
│      ▼                                                         │
│  REVIEW:   "Review this code for:                              │
│             □ Potential bugs                                   │
│             □ Security vulnerabilities                         │
│             □ Performance issues                               │
│             □ Missing error handling                           │
│             □ Deviation from conventions                       │
│             □ Missing edge cases"                              │
│      │                                                         │
│      ▼                                                         │
│  FINDINGS: "Found 3 issues:                                    │
│             1. SQL injection in query builder                  │
│             2. Missing null check on user.email                │
│             3. N+1 query in the loop"                          │
│      │                                                         │
│      ▼                                                         │
│  REFINE:   Fix each issue                                      │
│      │                                                         │
│      ▼                                                         │
│  RE-REVIEW: "Issues 1-3 fixed. New review:                     │
│              □ All clear"                                      │
│      │                                                         │
│      ▼                                                         │
│  ACCEPT:   Code meets quality standards                        │
└────────────────────────────────────────────────────────────────┘
```

## Implementation

### Basic Reflection Loop

```typescript
interface ReviewResult {
  issues: Issue[];
  overallQuality: "poor" | "acceptable" | "good" | "excellent";
  suggestions: string[];
}

interface Issue {
  severity: "critical" | "major" | "minor";
  description: string;
  location: string;
  fix: string;
}

async function reflectiveCodeGeneration(task: string): Promise<string> {
  const maxRefinements = 3;
  
  // Step 1: Generate initial code
  let code = await llm.complete({
    system: "You are a senior developer. Write clean, correct, well-tested code.",
    prompt: task
  });

  for (let i = 0; i < maxRefinements; i++) {
    // Step 2: Review the code
    const review: ReviewResult = await llm.complete({
      system: `You are a thorough code reviewer. Review the code for:
               - Bugs and logical errors
               - Security vulnerabilities (injection, XSS, auth bypass)
               - Performance issues (N+1 queries, memory leaks)
               - Missing error handling and edge cases
               - Deviation from best practices
               
               Be specific: cite exact lines and explain why each issue matters.
               Rate overall quality as: poor, acceptable, good, or excellent.`,
      prompt: `Review this code:\n\n${code}`
    });

    // Step 3: Check if quality is sufficient
    if (review.overallQuality === "excellent" || review.issues.length === 0) {
      console.log(`Reflection complete after ${i + 1} rounds — quality: ${review.overallQuality}`);
      return code;
    }

    if (review.overallQuality === "good" && !review.issues.some(i => i.severity === "critical")) {
      console.log(`Quality acceptable after ${i + 1} rounds`);
      return code;
    }

    // Step 4: Refine based on review
    console.log(`Round ${i + 1}: Found ${review.issues.length} issues, refining...`);
    code = await llm.complete({
      system: "Fix the identified issues while preserving correct functionality.",
      prompt: `Original code:\n${code}\n\nIssues found:\n${
        review.issues.map(issue => 
          `[${issue.severity}] ${issue.description} at ${issue.location}\nSuggested fix: ${issue.fix}`
        ).join("\n\n")
      }\n\nReturn the corrected code.`
    });
  }

  return code;
}
```

### Structured Self-Critique Prompts

The quality of reflection depends entirely on the review prompt. Here are effective patterns:

```typescript
const reviewPrompts = {
  // Bug detection
  bugReview: `Review this code for bugs:
    - Off-by-one errors
    - Null/undefined access
    - Race conditions
    - Incorrect boolean logic
    - Missing await on async operations
    - Type coercion issues`,

  // Security review
  securityReview: `Review this code for security vulnerabilities:
    - SQL injection (parameterized queries?)
    - XSS (output encoding?)
    - Authentication bypass
    - Authorization gaps (checking permissions?)
    - Sensitive data exposure (logging secrets?)
    - Path traversal (validating file paths?)`,

  // Performance review
  performanceReview: `Review this code for performance issues:
    - N+1 database queries
    - Missing indexes for common queries
    - Unbounded data fetching (no pagination?)
    - Memory leaks (unclosed connections, growing arrays?)
    - Unnecessary re-computation (memoize?)
    - Blocking the event loop`,

  // Convention review
  conventionReview: (conventions: string) => `Review this code against project conventions:
    ${conventions}
    
    Flag any deviations from these patterns.`
};
```

### Multi-Perspective Review

```typescript
async function multiPerspectiveReview(code: string): Promise<ReviewResult[]> {
  // Run multiple focused reviews in parallel
  const [bugReview, securityReview, perfReview] = await Promise.all([
    llm.complete({
      system: "You are a QA engineer focused on correctness.",
      prompt: `${reviewPrompts.bugReview}\n\nCode:\n${code}`
    }),
    llm.complete({
      system: "You are a security engineer focused on vulnerabilities.",
      prompt: `${reviewPrompts.securityReview}\n\nCode:\n${code}`
    }),
    llm.complete({
      system: "You are a performance engineer focused on efficiency.",
      prompt: `${reviewPrompts.performanceReview}\n\nCode:\n${code}`
    })
  ]);

  return [bugReview, securityReview, perfReview];
}
```

### Python Implementation

```python
from dataclasses import dataclass

@dataclass
class ReviewIssue:
    severity: str  # "critical", "major", "minor"
    description: str
    location: str
    suggested_fix: str

def reflective_code_gen(task: str, max_rounds: int = 3) -> str:
    # Generate initial code
    code = llm.complete(
        system="Write clean, correct code.",
        prompt=task
    )

    for round_num in range(max_rounds):
        # Self-critique
        review = llm.complete(
            system="""Review this code critically. Find:
            - Bugs and logic errors
            - Security issues
            - Missing error handling
            - Performance problems
            
            Return JSON: {"issues": [...], "quality": "poor|acceptable|good|excellent"}""",
            prompt=f"Review:\n{code}"
        )

        issues = parse_review(review)

        if not issues or review["quality"] in ("good", "excellent"):
            print(f"✓ Quality '{review['quality']}' after {round_num + 1} rounds")
            return code

        # Refine
        print(f"  Round {round_num + 1}: fixing {len(issues)} issues")
        code = llm.complete(
            system="Fix the identified issues. Return corrected code only.",
            prompt=f"Code:\n{code}\n\nIssues:\n{format_issues(issues)}"
        )

    return code
```

## The Evaluator-Optimizer Connection

Reflection is the **single-agent version** of the [evaluator-optimizer workflow](../02-workflow-patterns/evaluator-optimizer.md). The relationship:

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  REFLECTION (single agent):                                    │
│  ┌──────────────────────────────────────┐                     │
│  │  Same LLM generates AND reviews      │                     │
│  │  ┌──────────┐  ┌──────────┐         │                     │
│  │  │ Generate  │─►│ Review   │──► Fix  │                     │
│  │  └──────────┘  └──────────┘         │                     │
│  └──────────────────────────────────────┘                     │
│                                                                │
│  EVALUATOR-OPTIMIZER (two agents/prompts):                     │
│  ┌─────────────────┐  ┌─────────────────┐                     │
│  │  OPTIMIZER       │  │  EVALUATOR       │                     │
│  │  (generates)     │─►│  (reviews)       │                     │
│  │                  │◄─│                  │                     │
│  │  Different prompt│  │  Different prompt│                     │
│  │  or model        │  │  or model        │                     │
│  └─────────────────┘  └─────────────────┘                     │
│                                                                │
│  The evaluator-optimizer is more robust because the reviewer   │
│  has a DIFFERENT perspective than the generator.               │
└────────────────────────────────────────────────────────────────┘
```

## Limitations of Self-Critique

Models may not catch their own **systematic errors** — biases baked into the model affect both generation and review.

```
┌────────────────────────────────────────────────────────────────┐
│                SELF-CRITIQUE BLIND SPOTS                        │
│                                                                │
│  If the model doesn't understand a concept well enough:        │
│                                                                │
│  GENERATE: Writes code with a subtle concurrency bug           │
│  REVIEW:   "Code looks correct" ← same blind spot              │
│                                                                │
│  Solutions:                                                    │
│  ┌─────────────────────────────────────────────────────┐      │
│  │  1. Use a different model for evaluation             │      │
│  │  2. Use a different prompt/persona for evaluation    │      │
│  │  3. Use automated tools (linters, type checkers)     │      │
│  │  4. Use test execution as ground truth               │      │
│  │  5. Combine self-critique with external evaluation   │      │
│  └─────────────────────────────────────────────────────┘      │
└────────────────────────────────────────────────────────────────┘
```

### Mitigating Self-Critique Limitations

```typescript
async function robustReflection(code: string) {
  // Strategy 1: Different persona for review
  const selfReview = await llm.complete({
    system: `You are a DIFFERENT developer than the one who wrote this code.
             You are skeptical and looking for problems. 
             The author may have made mistakes they can't see.`,
    prompt: `Review this code thoroughly:\n${code}`
  });

  // Strategy 2: Automated tool verification
  const lintResult = await shell("npx eslint --format json src/");
  const typeCheck = await shell("npx tsc --noEmit");
  const testResult = await shell("npm test");

  // Strategy 3: Combine signals
  const allFindings = combineFindings(selfReview, lintResult, typeCheck, testResult);

  // Only automated signals (tests, types, linting) are fully reliable
  // Self-review is supplementary — catches logic issues tools miss
  return allFindings;
}
```

## When to Use Reflection

✅ **Good fit when**:
- **Quality matters more than speed** — you're willing to spend extra tokens for better output
- **Clear evaluation criteria exist** — you can articulate what "good" looks like
- **The task has common pitfalls** — security issues, performance traps, edge cases
- **No automated verification** — when you can't just run tests

❌ **Poor fit when**:
- **Speed is critical** — reflection doubles (or triples) the token cost
- **Evaluation criteria are subjective** — "make it elegant" is hard to self-evaluate
- **Automated tests exist** — running tests is a more reliable feedback signal than self-critique
- **Simple tasks** — "rename this variable" doesn't need a review cycle

## Combining Reflection with Other Patterns

Reflection works best when layered on top of other patterns:

```typescript
// ReAct + Reflection: reflect after each significant action
async function reactWithReflection(task: string) {
  // ... standard ReAct loop ...
  
  // After completing the task, add reflection
  const review = await selfCritique(completedWork);
  if (review.hasIssues) {
    // Enter another ReAct loop to fix the issues
    await reactLoop(`Fix these issues: ${review.issues}`);
  }
}

// Plan-and-Execute + Reflection: reflect on the plan AND on execution
async function planWithReflection(task: string) {
  let plan = await createPlan(task);
  
  // Reflect on the plan before executing
  const planReview = await selfCritique(plan, {
    criteria: "completeness, ordering, missing edge cases"
  });
  if (planReview.hasIssues) {
    plan = await refinePlan(plan, planReview);
  }
  
  const result = await executePlan(plan);
  
  // Reflect on the implementation
  const codeReview = await selfCritique(result, {
    criteria: "bugs, security, performance, conventions"
  });
  if (codeReview.hasIssues) {
    await fixIssues(codeReview.issues);
  }
}
```

## Next Steps

- [Evaluator-Optimizer](../02-workflow-patterns/evaluator-optimizer.md) — the formalized two-agent version
- [ReAct Pattern](./react-pattern.md) — the core loop that reflection enhances
- [Autonomous Agents](./autonomous-agents.md) — full autonomous loop with self-assessment
- [Plan-and-Execute](./plan-and-execute.md) — adding reflection to structured plans
