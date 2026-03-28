# Prompt Templates

> Ready-to-use prompt templates for common development tasks.

---

## Template 1: Code Generation

```markdown
Generate a [language] function called [name] that [description].

Interface:
  Input: [parameter types]
  Output: [return type]
  Side effects: [none / list them]

Requirements:
- [requirement 1]
- [requirement 2]
- [requirement 3]

Constraints:
- Follow the pattern in [reference file]
- [constraint 1]
- [constraint 2]

Success criteria:
- [what "done" looks like]
```

---

## Template 2: Code Review

```markdown
Review this code for [specific concern: bugs / security / performance].

ONLY flag issues that could cause [real problems / production failures].

Do NOT comment on:
- Code style or formatting
- Naming preferences
- [other things to ignore]

For each issue, provide:
- Severity: critical / high / medium
- Location: file:line
- Issue: one-sentence description
- Fix: corrected code

If no issues found, say "No issues detected."
```

---

## Template 3: Bug Fix

```markdown
Fix this bug:

Error: [exact error message]
File: [file path and line number]
Input that triggers: [specific input or steps]
Expected behavior: [what should happen]
Actual behavior: [what happens instead]

Constraints:
- Do not change the function signature
- [other constraints]
- All existing tests must still pass
```

---

## Template 4: Refactoring

```markdown
Refactor [target] to [specific change].

What to change:
- [transformation 1]
- [transformation 2]

What to preserve:
- All existing behavior
- Public API / function signatures
- All existing tests pass
- [other invariants]

Pattern to follow: [reference file or description]
```

---

## Template 5: Test Generation

```markdown
Write [framework] tests for [function/module].

Test file location: [path]
Mocking: [what to mock and how]

Test categories:
1. Happy path: [key scenarios]
2. Validation: [invalid inputs]
3. Errors: [failure scenarios]
4. Edge cases: [boundary conditions]

Each test should:
- Have a clear descriptive name
- Use Arrange → Act → Assert pattern
- Be independent (no shared state)
```

---

## Template 6: .prompt.md File

```yaml
---
description: "[What this prompt does]"
mode: "agent"
---

# Task
[Clear description of what to do]

## Context
[Project-specific information the model needs]

## Steps
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Constraints
- [Rule 1]
- [Rule 2]

## Output
[What the result should look like]
```

---

## Template 7: Architecture Decision

```markdown
I need to decide between [Option A] and [Option B] for [purpose].

Context:
- [Relevant project details]
- [Scale/performance requirements]
- [Team experience]

For each option, analyze:
1. Development effort (person-days)
2. Operational complexity
3. Scalability limits
4. Failure modes
5. Migration difficulty if we change later

Recommend one with clear reasoning.
```

---

## Template 8: System Prompt / Instructions File

```markdown
# [Project Name]

## Stack
[Framework], [Language], [Database], [Key libraries]

## Rules (10-15 max)
1. [Most important rule]
2. [Second most important rule]
...

## Patterns
- Error handling: [approach with example]
- Validation: [approach]
- Testing: [framework and patterns]

## What NOT to Do
- [Critical anti-pattern to avoid]
```

---

## Next Steps

- 🔗 [Troubleshooting](troubleshooting.md) — When prompts don't work
- 🔗 [FAQ](faq.md) — Quick answers
