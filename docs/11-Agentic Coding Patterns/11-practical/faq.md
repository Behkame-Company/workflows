# Agentic Coding FAQ

> Answers to the most common questions about agentic coding workflows — from when to use agents to how to debug them.

---

## Getting Started

### Q: Should I use agents for everything?

**No.** Start simple and add complexity only when needed:

```
Complexity Ladder (climb only when necessary):

1. Single prompt       — "Write a function that..."
2. Augmented LLM       — Prompt + retrieval/tools
3. Prompt chain        — Multi-step with fixed sequence
4. Workflow            — Chains with routing/parallelism
5. Agent               — Autonomous tool-using loop
```

Use agents when the task requires dynamic decision-making, multi-step tool use, or adaptation to intermediate results. For well-defined, predictable tasks, a simple prompt or chain is cheaper and more reliable.

---

### Q: What's the minimum setup to start with agentic coding?

A `copilot-instructions.md` or `CLAUDE.md` file in your repo with:
- Build/test commands
- 5-10 key coding conventions
- Forbidden patterns

That's it. Add more as you discover what the agent gets wrong.

---

### Q: Do I need to learn prompt engineering first?

It helps, but you can start with templates. The key skills are:
1. Writing clear instructions (like writing for a new team member)
2. Defining success criteria explicitly
3. Knowing when to intervene vs. let the agent iterate

---

## Tool Selection

### Q: Copilot CLI vs. Claude Code — which should I use?

| Aspect | Copilot CLI | Claude Code |
|--------|------------|-------------|
| **IDE integration** | VS Code, JetBrains | Terminal-native |
| **Agent mode** | Built into editor | Standalone CLI |
| **Custom skills** | .github/skills/ | CLAUDE.md + MCP |
| **Context mgmt** | Auto-compact at 95% | Manual /compact |
| **Multi-agent** | Via Copilot coding agent | Via sub-processes |
| **Best for** | IDE-centric workflows | Terminal-centric workflows |

They're complementary. Many developers use both — Copilot for inline and editor tasks, Claude Code for complex terminal-driven work.

---

### Q: How many parallel agents should I run?

It depends on **task independence** and **rate limits**:

- **2-3 agents**: Safe starting point for most projects
- **4-6 agents**: Feasible with well-partitioned tasks and sufficient rate limits
- **7+ agents**: Only with strict file partitioning and orchestration

**Rule of thumb:** If agents might edit the same file, don't parallelize them.

---

### Q: When should I use MCP servers vs. built-in tools?

- **Built-in tools**: File editing, terminal commands, search — use these by default
- **MCP servers**: External APIs (databases, cloud services, third-party tools) — use when the agent needs access to something outside the local environment
- **Custom MCP**: When you need domain-specific tools (e.g., query your internal API, access proprietary data)

---

## Debugging & Reliability

### Q: How do I debug agent behavior?

Four approaches, from easiest to most thorough:

1. **Read the transcript** — Most agent tools show the full conversation. Look for where the agent went wrong
2. **Check tool call sequence** — Was the right tool used? Were the arguments correct?
3. **Add logging instructions** — Tell the agent to explain its reasoning before each action
4. **Step-through mode** — Use a human-in-the-loop pattern where you approve each step

```markdown
## Debug instructions to add temporarily:

Before EVERY action, state:
1. What you're about to do and why
2. What you expect the result to be
3. How you'll know if it worked
```

---

### Q: Are agents deterministic?

**No.** The same prompt can produce different outputs across runs. This is fundamental to LLMs.

**Mitigations:**
- Use `temperature: 0` where supported (reduces but doesn't eliminate variance)
- Build eval suites that test across multiple runs
- Focus on outcome verification, not exact output matching
- Design workflows that are robust to variation (test gates, not output comparison)

---

### Q: How do I prevent agents from making mistakes?

You can't fully prevent mistakes — but you can catch them quickly:

1. **Tests** — The agent must run tests before reporting completion
2. **CI gates** — Automated checks that can't be bypassed
3. **Code review** — Human reviews the diff, not the agent's summary
4. **Type checking** — Static analysis catches many errors automatically
5. **Linting** — Enforces conventions the agent might forget
6. **Incremental commits** — Small changes are easier to review and revert

> *The goal is fast detection and easy recovery, not perfection.*

---

### Q: What do I do when the agent hallucinates code?

Common hallucinations: non-existent APIs, wrong function signatures, fabricated imports.

**Fixes:**
- Include actual code examples in your instructions
- Reference specific files: "Follow the pattern in `src/routes/users.ts`"
- Use TypeScript or equivalent — type errors catch many hallucinations
- Ensure the agent reads relevant files before writing new code
- Use retrieval (search the codebase) before generation

---

## Patterns & Architecture

### Q: What's the best agent pattern for coding tasks?

It depends on the task:

| Task Type | Best Pattern | Why |
|-----------|-------------|-----|
| Bug fix | Single agent | Focused investigation, one file at a time |
| New feature | Orchestrator-workers | Decompose into backend/frontend/tests |
| Refactoring | Prompt chain | Predictable sequence: analyze → plan → execute |
| Code review | Single agent | Read-only, focused analysis |
| Migration | Parallelized workers | Same transformation across many files |

---

### Q: How do I structure a multi-agent workflow?

```
1. Define the interface contract first (shared types, APIs)
2. Partition by file/module (no overlapping edits)
3. Use an orchestrator to coordinate:
   a. Decompose task into sub-tasks
   b. Assign each to a worker with specific file ownership
   c. Run workers (parallel where possible)
   d. Verify integration after all workers complete
4. Run integration tests on the combined output
```

---

### Q: When should I intervene in an agent's work?

Intervene when:
- **Cost exceeds threshold** — Agent is burning tokens without progress
- **Stuck in a loop** — Same approach repeated 3+ times
- **Quality is dropping** — Late changes are worse than early ones
- **Wrong direction** — Agent misunderstood the task
- **Security concern** — Agent is doing something risky

Don't intervene when:
- Agent is making steady progress, even if slowly
- Agent's approach differs from what you'd do but is valid
- Agent is exploring/reading code to understand the problem

---

### Q: How do I handle agent errors gracefully?

```typescript
// Pattern: Try → Verify → Retry with new strategy → Escalate
try {
  const result = await agent.execute(task);
  const verified = await verify(result);
  
  if (!verified) {
    // Give agent feedback and a new strategy
    const retryResult = await agent.execute(task, {
      feedback: verified.errors,
      strategy: "try a different approach",
    });
  }
} catch (error) {
  // Escalate to human with full context
  await notifyHuman({
    task,
    error,
    agentAttempts: agent.history,
    suggestion: "Manual intervention needed",
  });
}
```

---

## Cost & Performance

### Q: How much do agentic workflows cost?

Rough estimates per task:

| Task Complexity | Typical Token Usage | Approximate Cost |
|----------------|--------------------| ----------------|
| Simple bug fix | 10-30K tokens | $0.05-0.15 |
| Feature addition | 50-150K tokens | $0.25-0.75 |
| Complex refactor | 100-500K tokens | $0.50-2.50 |
| Multi-agent feature | 200K-1M tokens | $1.00-5.00 |

Costs vary significantly by model, provider, and task efficiency.

---

### Q: How do I reduce agent costs?

1. **Start simple** — Use cheaper models for straightforward tasks
2. **Write better prompts** — Clear instructions = fewer iterations
3. **Set budgets** — Token/time limits per task
4. **Cache context** — Don't re-read the same files repeatedly
5. **Break up tasks** — Smaller tasks are more token-efficient
6. **Use fast models for routing** — Use a smaller model to classify, then a capable model to execute

---

### Q: How long should an agent task take?

| Task | Expected Duration | Red Flag If |
|------|-------------------|-------------|
| Bug fix | 2-10 minutes | > 15 minutes |
| Small feature | 5-20 minutes | > 30 minutes |
| Complex feature | 15-60 minutes | > 90 minutes |
| Multi-agent pipeline | 10-45 minutes | > 60 minutes |

If hitting red flags, the agent is likely stuck or the task needs decomposition.

---

## Security & Safety

### Q: Can agents introduce security vulnerabilities?

**Yes.** Agents can introduce the same vulnerabilities a human developer can — and some unique ones:

- SQL injection from generated queries
- Missing input validation
- Exposed secrets in generated code
- Insecure defaults
- Prompt injection through user inputs

**Mitigations:** Security linting (e.g., Semgrep), SAST in CI, human review for security-sensitive code, never let agents handle secrets directly.

---

### Q: Should agents have access to production?

**No.** Follow the principle of least privilege:

- **Development/test environments**: Full agent access
- **Staging**: Read-only or supervised access
- **Production**: No direct agent access — agents create PRs, humans deploy

---

### Q: How do I prevent prompt injection through agent tools?

- Sanitize all user inputs before they reach agent prompts
- Use structured tool inputs (typed schemas, not free text)
- Validate tool outputs before using them in subsequent prompts
- Run agents in sandboxed environments with limited permissions
- Never interpolate untrusted data directly into system prompts

---

## Team & Process

### Q: How do I get my team to adopt agentic workflows?

1. Start with one willing adopter on a low-risk task
2. Share the time savings (with data)
3. Create team templates that encode your conventions
4. Establish review practices for agent-generated code
5. Document what works and what doesn't for your team's codebase

---

### Q: Should agent-generated code be reviewed differently?

Review agent code the same way you'd review junior developer code:
- Verify it does what it claims
- Check edge cases and error handling
- Look for security issues
- Ensure it follows conventions
- Don't assume correctness based on confidence of the summary

The diff is what matters, not the agent's narrative about it.

---

### Q: How do I track agent effectiveness over time?

Metrics to monitor:
- **Task completion rate** — % of tasks agents finish without human intervention
- **Iteration count** — Average retries per task (lower is better)
- **Defect rate** — Bugs in agent-generated code found in review/production
- **Cost per task** — Token spend per task type
- **Time savings** — Agent time vs. estimated human time
