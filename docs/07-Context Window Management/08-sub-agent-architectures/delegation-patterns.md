# Delegation Patterns

> When to delegate to sub-agents and how to structure the handoff.

---

## Delegation Decision Framework

```
Task received
    │
    ├── Can I do this with 1-2 tool calls?
    │   YES → Do it directly (no sub-agent)
    │
    ├── Will this generate >5K tokens of intermediate context?
    │   YES → Delegate to sub-agent
    │
    ├── Is this an independent task (no dependency on my current state)?
    │   YES → Delegate to sub-agent (can run in parallel)
    │
    ├── Does this require deep exploration I won't need later?
    │   YES → Delegate to explore agent
    │
    └── Is this a well-defined, bounded task?
        YES → Delegate to task agent
        NO  → Keep it (requires iterative judgment)
```

---

## Delegation Types

### 1. Explore (Read-Only Investigation)
```
Purpose: Answer a question about the codebase
Returns: Summary of findings
Tools: grep, glob, view (no modifications)
Example: "How is authentication implemented in this project?"
```

### 2. Task (Execute and Report)
```
Purpose: Complete a bounded task
Returns: Success/failure + summary
Tools: All tools including file modifications
Example: "Run the test suite and report results"
```

### 3. Code Review (Analyze Changes)
```
Purpose: Review code for issues
Returns: List of findings
Tools: Read-only + analysis
Example: "Review the changes in this PR for bugs"
```

### 4. General-Purpose (Complex Work)
```
Purpose: Multi-step task requiring judgment
Returns: Detailed results
Tools: Full toolset
Example: "Refactor the auth module to use middleware pattern"
```

---

## Effective Delegation Prompts

### What to Include
```markdown
Task: [Specific, bounded task description]
Context: [What the sub-agent needs to know]
Files: [Relevant file paths]
Constraints: [What to do and not do]
Output: [What to return]
```

### Example: Good Delegation
```markdown
Task: Investigate why the login endpoint returns 401 
      for valid credentials.
      
Context: The auth module is in src/auth/. We recently 
         changed the JWT signing algorithm from HS256 to RS256.
         
Files: src/auth/jwt.ts, src/routes/auth/login.ts, 
       tests/auth.test.ts
       
Constraints: Do NOT modify any files. Only investigate 
             and report findings.
             
Output: Return a summary with:
  1. Root cause of the 401 error
  2. Which file(s) need to be changed
  3. Suggested fix (description, not code)
```

---

## Parallel vs. Sequential Delegation

### Parallel (Independent Tasks)
```
Launch simultaneously:
  Agent A: "Explore the auth module"
  Agent B: "Explore the payment module"
  Agent C: "Review the test coverage"

Wait for all → Synthesize results
```

### Sequential (Dependent Tasks)
```
Step 1: Agent A explores → Returns findings
Step 2: Use findings to delegate to Agent B → Returns code
Step 3: Use code to delegate to Agent C → Returns test results
```

---

## Key Takeaways

1. **Delegate when context cost > overhead cost** — >5K intermediate tokens
2. **Specific prompts get specific results** — Include task, context, constraints
3. **Parallelize independent work** — Explore agents are safe to parallelize
4. **Keep iterative judgment tasks** — Don't delegate what needs ongoing decisions
5. **Return summaries, not dumps** — Sub-agents compress, lead agent decides

---

## Next Steps

- 🔗 [Multi-Agent Context Coordination](multi-agent-context-coordination.md) — Orchestrating agents
- 🔗 [Minimal Viable Context](../09-best-practices/minimal-viable-context.md) — The guiding principle
