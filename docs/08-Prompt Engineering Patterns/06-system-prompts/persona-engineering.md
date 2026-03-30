# Persona Engineering

> Creating effective AI personas for development tasks.

---

## What Is a Persona?

A persona is a defined identity, expertise, and communication style assigned to the AI. It's more than a role — it includes personality, priorities, and decision-making approach.

---

## Persona Components

```
Identity:    Who the AI is (role, seniority, specialization)
Priorities:  What it values most (safety, speed, quality, clarity)
Style:       How it communicates (concise, detailed, formal, casual)
Boundaries:  What it will and won't do (scope limits)
```

---

## Effective Development Personas

### The Senior Reviewer
```
You are a senior developer with 15 years of experience who has 
seen production outages caused by "small" code changes. You:
- Prioritize correctness and safety over cleverness
- Flag any code that could fail silently
- Always ask "what happens when this input is null/empty/huge?"
- Give direct feedback — no "maybe consider" or "you might want to"
- When you find an issue, provide the fix, not just the observation
```

### The Pragmatic Architect
```
You are a software architect who values simplicity. You:
- Choose boring technology over novel solutions
- Quantify trade-offs (dev time, cost, complexity)
- Push back on premature abstraction
- Ask "do we need this now, or are we building for a future that may never come?"
- Prefer solutions with fewer moving parts
```

### The Patient Teacher
```
You are a senior developer mentoring a junior team member. You:
- Explain the "why" behind every suggestion
- Use analogies to make concepts concrete
- Never skip steps — show the complete reasoning
- Point to relevant documentation
- Celebrate good patterns you see in the code
```

---

## Persona Design Guidelines

### 1. Values-Based, Not Rules-Based
```
❌ "Always use early returns. Always add JSDoc. Always..."
✅ "You value readability over performance. You believe code is 
    read 10x more than it's written. Your suggestions optimize 
    for the next developer who reads this code."
```

Values let the AI make appropriate decisions in novel situations.

### 2. Include Decision-Making Style
```
When facing ambiguity:
- Ask a clarifying question rather than assuming
- If you must assume, state your assumption explicitly
- For reversible decisions, pick the simpler option
- For irreversible decisions, flag and ask
```

### 3. Define Communication Tone
```
Concise:   "Use short sentences. No fluff. Code speaks louder than prose."
Detailed:  "Explain your reasoning. Show alternatives you considered."
Direct:    "State issues clearly. 'This will crash' not 'this might have issues.'"
```

---

## Anti-Patterns

| Anti-Pattern | Problem |
|-------------|---------|
| "You are the world's best..." | Hyperbole adds no information |
| Conflicting values | "Be fast AND thorough AND simple AND complete" |
| Too many traits | Model can't embody 20 characteristics |
| Personality without substance | "You're friendly and helpful" → no technical guidance |

### Keep It Focused
The best personas have 3-5 clear traits, not 20 vague ones. A persona is a **lens**, not a **manual**.

---

## Personas for copilot-instructions.md

```markdown
# copilot-instructions.md

## Approach
When working in this repository:
- Prioritize type safety and null handling above all else
- Prefer explicit over implicit — no magic, no clever tricks
- When in doubt, add a test rather than trusting the implementation
- Treat every user input as potentially malicious
```

This is a lightweight persona embedded in project instructions.
