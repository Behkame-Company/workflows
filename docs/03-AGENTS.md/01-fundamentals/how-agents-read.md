# How AI Agents Read AGENTS.md

> Resolution rules, directory hierarchy, and what happens under the hood.

---

## Resolution: Which AGENTS.md Gets Loaded?

### Single Repository (Simple Case)

```
project/
├── AGENTS.md          ← Agent reads this
├── src/
│   └── app.ts
└── tests/
    └── app.test.ts
```

Every file in the project uses the root AGENTS.md.

### Monorepo (Hierarchical Resolution)

**Rule: Nearest AGENTS.md in the directory tree wins.** The agent walks up from the file being edited and uses the first AGENTS.md it finds. Root AGENTS.md may also be merged depending on the tool.

Merging behavior, inheritance patterns, and tool-specific resolution rules vary significantly across tools.

For detailed monorepo resolution rules, inheritance patterns, and tool-specific behaviors, see [Directory Hierarchy](../02-deep-dives/directory-hierarchy.md).

---

## What Agents Do With the Content

### Step 1: Discovery

Agent identifies AGENTS.md in the repository. Most agents scan:
1. Current working directory
2. Repository root
3. Parent directories upward (for nested AGENTS.md)

### Step 2: Loading

The full content of AGENTS.md is loaded into the agent's **context window** alongside:
- The task description (issue, prompt, or command)
- File contents being worked on
- Tool-specific system prompts
- Previous conversation or action history

### Step 3: Interpretation

The agent treats AGENTS.md content as **high-priority instructions** — stronger than inferred patterns from the codebase, but weaker than the user's direct prompt.

```
Priority (highest → lowest):
1. User's direct prompt / issue description
2. AGENTS.md instructions
3. Tool-specific instruction files (copilot-instructions.md, etc.)
4. Patterns inferred from existing code
5. General training knowledge
```

### Step 4: Application

The agent applies instructions throughout its work:
- **Commands**: Runs specified build/test commands
- **Conventions**: Follows stated coding rules
- **Boundaries**: Avoids forbidden actions
- **Patterns**: Matches described architectural patterns

---

## The Attention Problem

Like all LLMs, coding agents have **attention patterns** that affect how well they follow instructions:

### Primacy Effect (Beginning Gets More Attention)

```
[Top of AGENTS.md]     ← HIGH attention
[Middle of AGENTS.md]  ← LOWER attention
[End of AGENTS.md]     ← HIGH attention
```

**Strategy**: Put critical commands and safety rules at the top and bottom. Less critical conventions go in the middle.

### Token Competition

AGENTS.md competes for context window space with:
- File contents the agent is reading/editing
- Task description
- Intermediate reasoning and tool output
- Conversation history

**Strategy**: Keep AGENTS.md concise. Every unnecessary word pushes out space for actual code.

### Relevance Filtering

Agents don't mechanically follow every instruction. They **weigh relevance** to the current task:

```
Task: "Fix the login form validation"

HIGH relevance: "Validate input with Zod"
HIGH relevance: "Use TypeScript strict mode"
LOW relevance:  "Database migrations must use nullable columns"
```

**Strategy**: Rules that are always relevant (conventions, boundaries) work better than highly specific rules that rarely apply.

---

## Compliance Expectations

| Instruction Type | Expected Compliance |
|-----------------|-------------------|
| Explicit commands (`pnpm test`) | ~95% — agents follow these reliably |
| Clear boundaries ("NEVER modify X") | ~90% — strong negative constraints |
| Specific conventions ("Named exports only") | ~85% — mostly followed |
| Soft preferences ("Prefer X when possible") | ~50% — often ignored |
| Vague guidance ("Write clean code") | ~5% — effectively meaningless |

**Takeaway**: Specificity correlates directly with compliance.

---

## Debugging: Is My AGENTS.md Being Read?

### Test 1: Direct Question

Ask the agent: "What are the project conventions according to AGENTS.md?"

If it quotes your content, it's reading the file.

### Test 2: Behavioral Test

Include a distinctive instruction and test compliance:
```markdown
## Conventions
- All functions must return explicit types (never inferred)
```

Ask the agent to create a function. Does it include an explicit return type?

### Test 3: Command Test

Include a specific test command:
```markdown
## Setup
- Test: `pnpm test:unit --reporter=verbose`
```

Assign a task that requires testing. Does the agent run this exact command?

### Test 4: Boundary Test

Include a boundary and try to violate it:
```markdown
## Boundaries
- NEVER use `console.log`
```

Ask the agent to add logging. Does it use a logger instead of console.log?
