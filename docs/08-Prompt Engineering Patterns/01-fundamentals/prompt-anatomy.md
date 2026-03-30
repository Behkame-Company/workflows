# Anatomy of a Prompt

> The 5 components that make up every effective prompt.

---

## The 5 Components

Every prompt is composed of some combination of these elements:

```
┌─────────────────────────────────────────┐
│ 1. INSTRUCTION     What to do           │
│ 2. CONTEXT         Background info      │
│ 3. EXAMPLES        Input/output pairs   │
│ 4. CONSTRAINTS     Rules and limits     │
│ 5. OUTPUT FORMAT   How to respond       │
├─────────────────────────────────────────┤
│ → INPUT            The actual task data  │
└─────────────────────────────────────────┘
```

---

## 1. Instruction

The core task statement. What do you want the model to do?

**Weak**: "Look at this code"
**Strong**: "Review this TypeScript function for null-safety bugs"

**Guidelines**:
- Start with a verb: "Generate", "Review", "Refactor", "Explain"
- Be specific about the action
- Specify the scope (which file, which function, what aspect)

---

## 2. Context

Background information the model needs to perform the task.

```markdown
Context:
- This is a Next.js 14 App Router project
- We use Zod for all input validation
- The function is called from a server action
- Users can have null profiles if they haven't completed onboarding
```

**Guidelines**:
- Only include context relevant to the task
- Project conventions, constraints, and recent decisions
- What the model can't infer from the code alone

---

## 3. Examples

Input/output pairs that show the model what you expect.

```xml
<examples>
<example>
  <input>function add(a, b) { return a + b }</input>
  <output>function add(a: number, b: number): number { return a + b }</output>
</example>
<example>
  <input>const greet = (name) => `Hello ${name}`</input>
  <output>const greet = (name: string): string => `Hello ${name}`</output>
</example>
</examples>
```

**Guidelines**:
- 3-5 diverse examples for best results
- Cover edge cases in your examples
- Use XML tags to separate examples from instructions

---

## 4. Constraints

Rules that limit the output space.

```markdown
Constraints:
- Do NOT change function signatures
- Keep all existing comments
- Use early returns instead of nested if/else
- Maximum 30 lines per function
- No external dependencies
```

**Guidelines**:
- Tell the model what to do, not just what not to do
- "Write in plain prose paragraphs" > "Don't use markdown"
- Prioritize: conflicting constraints confuse the model

---

## 5. Output Format

How the response should be structured.

```markdown
Respond with:
1. A one-line summary of the issue
2. The corrected code in a TypeScript code block
3. A brief explanation of what changed and why
```

**Guidelines**:
- Be explicit about format (JSON, markdown, code block, prose)
- Specify what sections to include
- Show a format template if the structure is complex

---

## Putting It Together

```markdown
[INSTRUCTION]
Review this API endpoint for security vulnerabilities.

[CONTEXT]
This is a Node.js Express endpoint that handles user profile updates.
We use JWT authentication middleware. The user object is available on req.user.

[CONSTRAINTS]
- Focus only on security issues (not style or performance)
- Rate each issue as Critical, High, Medium, or Low severity
- Ignore issues already handled by our auth middleware

[OUTPUT FORMAT]
For each issue found, provide:
- Severity: [Critical/High/Medium/Low]
- Location: [line number or code section]
- Issue: [one-sentence description]
- Fix: [corrected code snippet]

If no issues found, say "No security issues detected."

[INPUT]
<code>
// ... the actual code ...
</code>
```

---

## Component Importance by Task

| Task Type | Instruction | Context | Examples | Constraints | Format |
|-----------|:-----------:|:-------:|:--------:|:-----------:|:------:|
| Code generation | ★★★ | ★★★ | ★★ | ★★★ | ★★ |
| Code review | ★★★ | ★★ | ★ | ★★★ | ★★★ |
| Refactoring | ★★ | ★★★ | ★★★ | ★★ | ★ |
| Bug fixing | ★★★ | ★★★ | ★ | ★★ | ★★ |
| Documentation | ★★ | ★★ | ★★★ | ★ | ★★★ |
