# Agentic Prompting

> Prompts for autonomous AI agents that plan, execute, and verify their own work.

---

## What Is Agentic Prompting?

Agentic prompting designs instructions for AI agents that work autonomously — planning their approach, executing multi-step tasks, and verifying their own results. This is the frontier of AI-assisted development.

---

## The Agentic System Prompt Template

```markdown
# Your Identity
You are an autonomous coding agent working on [project].

# How to Work
1. Understand the task fully before starting
2. Plan your approach (list steps)
3. Execute one step at a time
4. Verify each step before moving on
5. Run tests after every code change
6. Commit after each logical unit of work

# When Stuck
- Re-read the relevant files
- Check git log for recent changes
- Run existing tests for diagnostic info
- Ask for clarification if truly blocked

# What NOT to Do
- Don't guess at file contents — read them
- Don't modify tests to make them pass (fix the code instead)
- Don't add unnecessary abstractions
- Don't skip verification steps
```

---

## Key Agentic Patterns

### 1. Plan Before Execute
```
Before making any changes:
1. List all files that need modification
2. Describe what changes each file needs
3. Identify potential risks or breaking changes
4. Only then start implementing

Show me the plan and wait for approval before executing.
```

### 2. Incremental Progress
```
Work incrementally:
- Make one logical change at a time
- Run tests after each change
- Commit working code before moving to the next task
- If a test fails, fix it immediately (don't accumulate failures)
- Track progress in progress.txt
```

### 3. Self-Verification
```
After completing the implementation:
1. Run the full test suite
2. Manually verify the key user flow works
3. Check for any regressions in related functionality
4. Review your own diff for obvious issues
5. Only report "done" when ALL verification passes
```

### 4. Graceful Degradation
```
If you encounter issues:
- Fixable (< 5 minutes): Fix it and continue
- Needs investigation (5-15 minutes): Investigate, document findings
- Blocking: Stop, document what you've done, what's blocking, 
  and suggested next steps

Never silently fail or skip a step.
```

---

## Agentic Prompting for Claude 4.x

Claude 4.x models have enhanced agentic capabilities. Key adjustments:

### Reduce Over-Prompting
```
❌ Old style (pre-4.6):
"CRITICAL: You MUST use grep to find files. ALWAYS verify before changing."

✅ New style (4.6+):
"Use grep to find files when needed. Verify before changing code."
```

Claude 4.6 follows normal-language instructions reliably. Aggressive emphasis causes overtriggering.

### Control Subagent Spawning
```
Use subagents when:
- The task requires deep investigation of a separate concern
- You need to explore multiple files independently

Don't use subagents for:
- Simple grep or file reads
- Tasks you can complete in a few tool calls
- Quick verification checks
```

### Manage Extended Exploration
```
Be efficient with exploration:
- Start with targeted searches, not broad exploration
- Read relevant files, not every file in a directory
- If you have enough context to proceed, stop gathering and start implementing
```

---

## Safety Guardrails

```
Before taking any of these actions, ask for confirmation:
- Deleting files or directories
- Force-pushing to any branch
- Modifying database migration files
- Running commands that modify external services (deploy, publish)
- Adding new external dependencies

For reversible actions (git commit, file edits), proceed without asking.
```

Anthropic recommends this pattern: classify actions by reversibility and only gate irreversible ones.

---

## Next Steps

- 🔗 [ReAct Pattern](react-pattern.md) — The reasoning+acting foundation
- 🔗 [Tool-Augmented Prompting](tool-augmented-prompting.md) — Effective tool use
- 🔗 [Prompt Chaining](../07-prompt-composition/prompt-chaining.md) — Sequential workflows
