# Meta-Prompting

> Prompts that generate, improve, or structure other prompts.

---

## What Is Meta-Prompting?

Meta-prompting is using AI to work on prompts themselves — generating new prompts, improving existing ones, or creating prompt templates. It's "prompting about prompting."

---

## Three Forms of Meta-Prompting

### 1. Prompt Generation
Ask the model to create a prompt for a specific task:

```
I need to review TypeScript code for accessibility issues 
in React components. Generate a detailed prompt that I can 
reuse for this task. The prompt should:
- Define the reviewer's expertise
- List specific a11y checks to perform
- Specify the output format
- Include 2 examples
```

### 2. Prompt Improvement
Give the model an existing prompt and ask it to improve:

```
Here's my current prompt for code review:
"Review this code and find bugs"

Improve this prompt to be more specific, structured, 
and reliable. Keep it under 200 words.
```

### 3. Structure-Oriented Meta-Prompting
Focus on the **pattern** of problem-solving rather than specific content:

```
Given a problem of type [CATEGORY]:
1. Identify the [KEY_COMPONENTS]
2. For each component, determine [RELATIONSHIP]
3. Apply [TRANSFORMATION] to get [RESULT]
4. Verify [RESULT] satisfies [CONSTRAINT]
```

This abstract template works across domains because it captures the **structure** of reasoning, not domain-specific content.

---

## Meta-Prompting in Practice

### Creating Reusable .prompt.md Files

```
I'm building a library of reusable prompts for my team. 
Generate a .prompt.md file for [task]. The file should include:
- YAML frontmatter with description and model
- Clear role definition
- Step-by-step instructions
- Output format specification
- Variables marked with {{variable_name}}
```

### Evaluating Prompt Quality

```
Evaluate this prompt against these criteria:
1. Clarity: Would a new team member understand the task?
2. Completeness: Are all necessary constraints specified?
3. Examples: Are examples diverse enough?
4. Output format: Is the expected format unambiguous?
5. Scope: Is the task well-bounded?

Score each 1-5 and suggest specific improvements.
```

---

## Meta-Prompting vs Few-Shot

| Aspect | Meta-Prompting | Few-Shot |
|--------|---------------|----------|
| Focus | Structure and pattern | Content and examples |
| Token efficiency | Higher (abstract template) | Lower (full examples) |
| Generalization | Works across tasks | Task-specific |
| Best for | Novel or varied tasks | Repeated specific tasks |

Meta-prompting is **token-efficient** because abstract templates are smaller than full examples, while still guiding the model's reasoning structure.

---

## Key Insight from Research

> Meta-prompting can be viewed as a form of zero-shot prompting where the influence of specific examples is minimized, focusing instead on the structural patterns of problem-solving. — Zhang et al., 2024

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Infinite recursion | "Write a prompt to write a prompt to write a prompt..." | Stop at one level of meta |
| Over-abstraction | Template so abstract it's useless | Ground with one concrete example |
| Trusting generated prompts blindly | Generated prompt may have gaps | Always test generated prompts |

---

## Next Steps

- 🔗 [Self-Consistency](self-consistency.md) — Multiple reasoning paths
- 🔗 [Prompt Chaining](../07-prompt-composition/prompt-chaining.md) — Connecting prompts sequentially
