# Anatomy of Effective Context

> What the ideal context window looks like — each component optimized for maximum signal.

---

## The Components

Every context window is composed of these layers:

```
┌─────────────────────────────────────────────────┐
│  1. SYSTEM PROMPT                                │
│     Role, rules, constraints, persona            │
│     Target: 500-3,000 tokens                     │
├─────────────────────────────────────────────────┤
│  2. TOOL DEFINITIONS                             │
│     Available tools and their schemas            │
│     Target: 1,000-5,000 tokens                   │
├─────────────────────────────────────────────────┤
│  3. INJECTED CONTEXT                             │
│     Files, docs, code relevant to current task   │
│     Target: 2,000-20,000 tokens (curated)        │
├─────────────────────────────────────────────────┤
│  4. CONVERSATION HISTORY                         │
│     Previous turns (may be compacted)            │
│     Target: Managed via compaction               │
├─────────────────────────────────────────────────┤
│  5. CURRENT QUERY                                │
│     The immediate user request                   │
│     Target: As specific as possible              │
├─────────────────────────────────────────────────┤
│  6. RESPONSE BUDGET                              │
│     Space reserved for the model's output        │
│     Target: Reserve 4K-16K tokens                │
└─────────────────────────────────────────────────┘
```

---

## Signal-to-Noise Ratio

**High-signal context** (keep):
- Directly relevant code files
- Specific error messages
- Architectural decisions that affect the current task
- Active constraints and requirements

**Low-signal context** (remove or summarize):
- Old conversation turns about completed tasks
- Full file contents when a snippet would suffice
- Verbose tool outputs (full `npm install` logs)
- Repeated information already established

---

## Optimization Principles

### 1. Right Altitude for System Prompts
```
❌ Too specific (brittle):
   "If the user asks about auth, use JWT with RS256 and..."

❌ Too vague (unhelpful):
   "Write good code"

✅ Right altitude:
   "Use JWT for authentication. Follow OWASP security guidelines.
    Prefer parameterized queries for all database operations."
```

### 2. Token-Efficient Tool Results
```
❌ Full file (2,000 tokens):
   Reading entire package.json when you only need the scripts

✅ Targeted retrieval (200 tokens):
   Reading only the "scripts" section of package.json
```

### 3. Structured Over Prose
```
❌ Prose (150 tokens):
   "The project uses React 18 with TypeScript 5, and we deploy
    to AWS using Docker containers orchestrated by ECS..."

✅ Structured (80 tokens):
   Tech: React 18, TypeScript 5
   Deploy: AWS ECS (Docker)
   DB: PostgreSQL 15
```

---

## The Attention Budget

Think of the model's attention as a fixed budget:

| Context Size | Attention per Token | Quality |
|-------------|-------------------|---------|
| 5K tokens | Very high | Excellent recall, precise reasoning |
| 50K tokens | High | Good recall, reliable reasoning |
| 200K tokens | Medium | Some missed details, occasional drift |
| 500K+ tokens | Low | Significant context rot, needs curation |

**Implication**: A carefully curated 20K-token context often outperforms a 200K-token context dump.

---

## Key Takeaways

1. **Every token must earn its place** — Remove what doesn't contribute
2. **Right altitude for instructions** — Not too brittle, not too vague
3. **Targeted retrieval** — Don't read entire files when snippets suffice
4. **Structured over prose** — Tables and lists use fewer tokens
5. **Reserve output budget** — Don't fill the window completely
