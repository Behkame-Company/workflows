# Dynamic Prompt Assembly

> Building prompts from modular components at runtime.

---

## What Is Dynamic Assembly?

Instead of writing complete prompts from scratch, you compose them from reusable building blocks:

```
┌─────────────────────────────────────────────┐
│ Final Prompt                                 │
│                                             │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐     │
│ │  Role     │ │ Project  │ │ Task     │     │
│ │  Block    │ │ Context  │ │ Specific │     │
│ └──────────┘ └──────────┘ └──────────┘     │
│ ┌──────────┐ ┌──────────┐                   │
│ │ Format   │ │ Examples │                   │
│ │ Rules    │ │ Block    │                   │
│ └──────────┘ └──────────┘                   │
└─────────────────────────────────────────────┘
```

---

## Building Blocks

### 1. Role Blocks
```markdown
# roles/senior-reviewer.md
You are a senior code reviewer with 10+ years of production experience.
You prioritize bugs and security over style.
```

```markdown
# roles/architect.md
You are a software architect focused on scalability and simplicity.
You quantify trade-offs and prefer boring technology.
```

### 2. Context Blocks
```markdown
# context/project.md
Stack: Next.js 14 (App Router), TypeScript strict, Prisma, PostgreSQL
Testing: Jest + React Testing Library
Validation: Zod on all API boundaries
Error format: { error: string, code: string }
```

### 3. Format Blocks
```markdown
# formats/review-output.md
For each issue, provide:
- Severity: critical | high | medium | low
- Location: file:line
- Issue: one sentence
- Fix: corrected code snippet
```

### 4. Constraint Blocks
```markdown
# constraints/security.md
- Treat all user input as untrusted
- Use parameterized queries (never string interpolation for SQL)
- Never log sensitive data (passwords, tokens, PII)
- Validate and sanitize all input before processing
```

---

## Assembly in .prompt.md Files

GitHub Copilot's prompt files can reference other files:

```yaml
---
description: "Security review for API endpoint"
---

# Context
Read the project configuration in copilot-instructions.md for conventions.

# Role
You are a senior security engineer reviewing this API endpoint.

# Task
Review the endpoint at {{file_path}} for OWASP Top 10 vulnerabilities.

# Output Format
For each finding:
- **Severity**: Critical/High/Medium/Low
- **Category**: OWASP category
- **Issue**: Description
- **Fix**: Code showing the correction
```

---

## Programmatic Assembly

```typescript
function assemblePrompt(task: {
  type: 'review' | 'generate' | 'test';
  role: 'reviewer' | 'architect' | 'tester';
  includeExamples: boolean;
}): string {
  const parts: string[] = [];
  
  // Role
  parts.push(loadBlock(`roles/${task.role}.md`));
  
  // Always include project context
  parts.push(loadBlock('context/project.md'));
  
  // Task-specific instructions
  parts.push(loadBlock(`tasks/${task.type}.md`));
  
  // Optional examples
  if (task.includeExamples) {
    parts.push(loadBlock(`examples/${task.type}.md`));
  }
  
  // Format rules
  parts.push(loadBlock(`formats/${task.type}-output.md`));
  
  return parts.join('\n\n---\n\n');
}
```

---

## Benefits

| Benefit | Why |
|---------|-----|
| Reusability | Write role/context blocks once, use everywhere |
| Consistency | Same context block = same project conventions |
| Testability | Test individual blocks in isolation |
| Maintainability | Update project context in one place |
| Composability | Mix and match for new use cases |

---

## Design Principles

1. **Keep blocks small** — Each block should be under 200 tokens
2. **One responsibility per block** — A role block doesn't include format rules
3. **Make blocks combinable** — No block should assume the presence of another
4. **Version control blocks** — They're code artifacts, treat them like code
5. **Test combinations** — Ensure common combinations produce good results

---

## Next Steps

- 🔗 [Prompt Routing](prompt-routing.md) — Selecting which components to assemble
- 🔗 [Prompt Chaining](prompt-chaining.md) — Connecting assembled prompts in sequence
