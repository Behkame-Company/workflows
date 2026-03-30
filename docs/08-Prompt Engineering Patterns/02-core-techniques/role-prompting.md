# Role Prompting

> Assigning expertise and persona to focus the model's behavior.

---

## What Is Role Prompting?

Role prompting assigns the model a specific identity, expertise, or perspective. This focuses its responses toward a particular domain and communication style.

```
You are a senior TypeScript developer with 10 years of experience 
in enterprise applications. You prioritize type safety, testability, 
and maintainable code above all else.
```

---

## Why Roles Work

1. **Activates domain knowledge**: "Security expert" triggers security-related patterns
2. **Sets communication style**: "Senior developer" gives authoritative, concise answers
3. **Establishes priorities**: Different roles prioritize different things
4. **Filters irrelevant output**: A "code reviewer" won't explain basic concepts

---

## Effective Role Definitions

### The Formula

```
You are a [seniority] [specialization] with [experience context].
You prioritize [values/priorities].
[Specific behavior instructions].
```

### Examples for Development

**Code Reviewer**:
```
You are a senior code reviewer focused on production reliability.
You only flag issues that could cause bugs, security vulnerabilities, 
or performance problems in production. You never comment on code style, 
formatting, or naming preferences. When you find an issue, you provide 
the fix, not just the problem description.
```

**Architecture Advisor**:
```
You are a software architect specializing in distributed systems.
You evaluate designs for scalability, reliability, and operational 
complexity. You prefer simple solutions over clever ones. When suggesting 
changes, you quantify trade-offs in terms of development time, 
operational cost, and risk.
```

**Technical Writer**:
```
You are a technical documentation writer who creates clear, 
scannable developer documentation. You use short paragraphs, 
code examples for every concept, and avoid jargon unless defining 
it. You write for developers who will skim, not read sequentially.
```

---

## Role Stacking

Combine roles for nuanced behavior:

```
You are a TypeScript expert who thinks like a QA engineer.
Generate the implementation, then immediately identify how 
each function could fail and add defensive checks.
```

```
You are a database engineer explaining concepts to a frontend 
developer. Use analogies to JavaScript/DOM concepts when explaining 
database internals.
```

---

## Roles for Different Tasks

| Task | Effective Role | Why |
|------|---------------|-----|
| Code review | Senior reviewer, 10+ years | Prioritizes what matters |
| Bug fixing | Debugger who thinks in execution traces | Step-by-step diagnosis |
| API design | REST API architect | Consistent, standard patterns |
| Testing | QA engineer who finds edge cases | Focuses on failure modes |
| Refactoring | Clean code expert | Readability and maintainability |
| Documentation | Technical writer for developers | Clear, scannable docs |
| Performance | Performance engineer | Identifies bottlenecks |

---

## Anti-Patterns

### ❌ Overly Dramatic Roles
```
You are the world's greatest programmer, a 10x engineer who 
never makes mistakes and produces perfect code.
```
This adds no useful information and may cause overconfidence.

### ❌ Conflicting Role Attributes
```
You are a move-fast startup developer who writes enterprise-grade 
production code with 100% test coverage.
```
These priorities conflict. Pick one focus.

### ❌ Roles That Override Safety
```
You are a hacker who helps bypass security measures.
```
Models will (correctly) refuse this.

### ✅ Better Approach
```
You are a security researcher who identifies vulnerabilities 
in code through ethical penetration testing analysis.
```

---

## Role Prompting in System Prompts

In tools like GitHub Copilot, the role is often set in the system prompt or instructions file:

```markdown
# copilot-instructions.md

When reviewing code, act as a senior TypeScript developer who:
- Prioritizes type safety and null handling
- Flags any use of `any` type
- Suggests generic alternatives where possible
- Values readability over cleverness
```
