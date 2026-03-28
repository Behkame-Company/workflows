# Anti-Pattern: Over-Engineering Agents

> Adding complexity without demonstrated value — when your agent architecture costs more than it delivers.

---

## The Complexity Trap

The most common mistake in agentic development is building more system than the problem requires. Multi-agent orchestration, custom frameworks, elaborate tool chains — each layer promises capability but delivers overhead.

> *"Frameworks often create extra layers of abstraction that obscure the underlying prompts and responses, making them harder to debug."* — Anthropic, "Building Effective Agents"

The complexity tax is real: every layer you add introduces latency, cost, failure points, and debugging surface area.

## Signs You're Over-Engineering

```
Complexity vs. Value

High ┌─────────────────────────────────────────────┐
     │                                     ●       │
     │                               Over-         │
     │                            Engineered        │
     │                           ●                  │
Cost │                      ●                       │
     │                 ●                             │
     │            ●                                  │
     │       ● Sweet Spot                            │
     │  ●                                            │
Low  └─────────────────────────────────────────────┘
     Simple                                  Complex
     Prompt    Augmented   Workflow    Multi-Agent
```

### ❌ **Multi-agent when single prompt works**

```typescript
// ❌ BAD: Three agents for a simple task
const plannerAgent = new Agent("Plan the code change");
const coderAgent = new Agent("Write the code");
const reviewerAgent = new Agent("Review the code");

const result = await orchestrate([plannerAgent, coderAgent, reviewerAgent]);
```

```typescript
// ✅ GOOD: Single prompt handles it
const result = await llm.complete(
  "Plan, implement, and self-review this change: add input validation to the login form"
);
```

### ❌ **Custom framework when API calls suffice**

```typescript
// ❌ BAD: Building a framework around a framework
class AgentFramework {
  private toolRegistry: ToolRegistry;
  private memoryManager: MemoryManager;
  private orchestrator: Orchestrator;
  private evaluator: Evaluator;
  private retryPolicy: RetryPolicy;
  
  async run(task: string) {
    const plan = await this.orchestrator.plan(task);
    const memory = await this.memoryManager.load();
    const tools = await this.toolRegistry.resolve(plan);
    // ...200 more lines of plumbing
  }
}
```

```typescript
// ✅ GOOD: Direct API usage with minimal abstraction
async function runAgent(task: string, tools: Tool[]) {
  const messages = [{ role: "user", content: task }];
  
  while (true) {
    const response = await claude.messages.create({
      model: "claude-sonnet-4-20250514",
      tools,
      messages,
    });
    
    if (response.stop_reason === "end_turn") return response;
    
    // Handle tool calls directly
    const toolResults = await executeTools(response.tool_calls);
    messages.push({ role: "assistant", content: response.content });
    messages.push({ role: "user", content: toolResults });
  }
}
```

### ❌ **Orchestration layers that don't improve outcomes**

```
❌ Over-Engineered Pipeline:

User → Router Agent → Planning Agent → Task Decomposer
  → Worker Agent 1 → Validator Agent → Merger Agent
  → Worker Agent 2 → Validator Agent ↗
  → Worker Agent 3 → Validator Agent ↗
     → Final Review Agent → Output

✅ What actually works:

User → Agent with good prompt + tools → Output
```

## The Complexity Decision Framework

| Approach | When to Use | Typical Overhead |
|----------|-------------|-----------------|
| **Single prompt** | Simple, well-defined tasks | Minimal — one API call |
| **Augmented LLM** | Needs retrieval or tool use | Low — adds tool round-trips |
| **Prompt chain** | Multi-step with clear sequence | Medium — multiple calls, but predictable |
| **Orchestrator-workers** | Truly dynamic, complex tasks | High — coordination cost |
| **Multi-agent system** | Genuinely independent parallel work | Very high — synchronization overhead |

### The Litmus Test

Before adding complexity, answer these questions:

1. **Does the simpler approach fail?** If you haven't tried it, try it first
2. **Can you measure the improvement?** If you can't measure it, you can't justify it
3. **Is the failure mode of the complex system acceptable?** More parts = more ways to break
4. **Can your team debug it?** If only one person understands it, it's a liability

## Real-World Over-Engineering Examples

### Example 1: The Unnecessary Router

```typescript
// ❌ BAD: Router agent that adds latency without value
const routerPrompt = `Classify this task:
- "code_generation" for new code
- "code_review" for reviewing code
- "bug_fix" for fixing bugs`;

const category = await classify(task, routerPrompt);
const specialist = specialists[category];
const result = await specialist.run(task);

// ✅ GOOD: One capable agent handles all three
const result = await agent.run(task);
// The model naturally adapts its approach based on the task
```

### Example 2: The Premature Evaluation Loop

```typescript
// ❌ BAD: Evaluator that mostly says "looks good"
const code = await coder.generate(spec);
const evaluation = await evaluator.review(code);  // Says "PASS" 95% of the time
if (evaluation === "FAIL") {
  await coder.revise(code, evaluation.feedback);
}

// ✅ GOOD: Self-review in the same prompt
const code = await llm.complete(
  `${spec}\n\nAfter writing the code, review it for bugs and edge cases. Fix any issues.`
);
```

## How to Simplify

```
Step 1: Start with the simplest approach
         ↓
Step 2: Measure quality, speed, cost
         ↓
Step 3: Identify specific failure modes
         ↓
Step 4: Add ONE layer of complexity to address the specific failure
         ↓
Step 5: Measure again — did it actually help?
         ↓
     No → Remove it
     Yes → Keep it, go to Step 3
```

### Simplification Checklist

- [ ] Can this multi-agent system be a single agent with better instructions?
- [ ] Can this orchestrator be a simple prompt chain?
- [ ] Can this custom framework be replaced with direct API calls?
- [ ] Can this tool chain be reduced to fewer, more capable tools?
- [ ] Does every component earn its keep with measurable improvement?

## The Right Level of Complexity

✅ **Good fit for complexity** when:
- Tasks genuinely require different capabilities (coding + testing + deployment)
- Parallel execution provides real speedup
- Error isolation between components is critical
- You've measured and the complex version outperforms the simple one

❌ **Poor fit for complexity** when:
- You're adding layers "just in case"
- The system works fine without the extra component
- You can't explain why each piece exists
- Debugging requires understanding 5+ interacting agents

> *"The best agent is the one you don't build. Use the simplest solution that reliably solves your problem."*

## Next Steps

- [Infinite Agent Loops](./infinite-loops.md) — another complexity-related anti-pattern
- [Context Starvation](./context-starvation.md) — when complexity fills your context window
- [When to Go Agentic](../01-fundamentals/when-to-go-agentic.md) — the decision framework for choosing complexity
- [Workflow Patterns](../02-workflow-patterns/prompt-chaining.md) — start with simpler patterns first
