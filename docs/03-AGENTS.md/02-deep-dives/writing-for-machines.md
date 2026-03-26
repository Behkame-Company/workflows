# Writing for Machines, Not Humans

> Why AGENTS.md needs fundamentally different writing than README.md, wikis, or onboarding docs.

---

## The Core Insight

README.md teaches humans who can ask follow-up questions, interpret context clues, and use judgment.

AGENTS.md instructs LLMs that interpret text literally, have limited context windows, and cannot ask "what did you mean by that?"

**Writing for agents requires a different mindset.**

---

## Side-by-Side Comparison

### ❌ Written for Humans (Conversational)

```markdown
We generally prefer TypeScript for most things, but sometimes
we use plain JavaScript for quick scripts. Tests are pretty
important here — try to add them when you can. We've been
moving towards Vitest but some older tests still use Jest.
Oh, and please don't touch anything in the old/ directory,
there's some legacy code there we're working on migrating.
```

### ✅ Written for Agents (Actionable)

```markdown
## Conventions
- Language: TypeScript only (except scripts/ — JavaScript OK)
- Tests required for all new functions in src/
- Test framework: Vitest (never Jest for new tests)

## Boundaries
- NEVER modify files in old/
- NEVER convert Jest tests to Vitest (migration tracked separately)
```

---

## Seven Principles for Agent-Oriented Writing

### 1. Be Imperative, Not Descriptive

```markdown
❌ "We tend to use named exports in this project."
✅ "Use named exports. Never use default exports."
```

Agents don't process "we tend to" — they need directives.

### 2. Be Specific, Not Abstract

```markdown
❌ "Write clean, well-structured code."
✅ "Maximum 30 lines per function. Extract helpers for any repeated logic (3+ occurrences)."
```

"Clean code" means nothing to an agent. Concrete rules are followable.

### 3. Use Exact Commands, Not Descriptions

```markdown
❌ "Run the tests to make sure everything works."
✅ "Run: `pnpm test --run --reporter=verbose`"
```

Agents need the exact command string, not a description of what to do.

### 4. State Both Positive and Negative Rules

```markdown
✅ "Use `async/await`. Never use `.then()` chains."
✅ "Import from `@/lib/logger`. Never use `console.log`."
✅ "Use Zod for validation. Never validate manually."
```

The "never" side prevents the most common agent errors.

### 5. Prefer Bullets Over Paragraphs

```markdown
❌ "When writing tests, we use Vitest with React Testing Library.
    The naming convention follows a 'should...when' pattern, and
    we generally aim for 80% coverage in core modules."

✅ ## Testing
   - Framework: Vitest + React Testing Library
   - Naming: `it('should [behavior] when [condition]')`
   - Coverage: 80% minimum in src/lib/ and src/server/
```

Each bullet becomes a discrete, parseable instruction.

### 6. Show, Don't Tell (Code Examples > Prose)

```markdown
❌ "API routes should follow a consistent pattern with proper
    error handling and input validation."

✅ ## API Route Pattern
   ```typescript
   export async function POST(req: Request) {
     const body = await req.json();
     const input = schema.parse(body);
     const result = await service.create(input);
     return Response.json(result, { status: 201 });
   }
   ```
```

A single code example communicates more than a paragraph of description.

### 7. Front-Load Critical Information

```markdown
❌ After explaining the full project context, its history, the
   team structure, and design philosophy... here are the commands:
   `pnpm install && pnpm dev`

✅ ## Setup
   - Install: `pnpm install`
   - Dev: `pnpm dev`
   [context and history can go later...]
```

Agents process top content with highest attention. Commands and rules go first.

---

## What NOT to Include

Content that wastes tokens and dilutes instructions:

| Don't Include | Why |
|--------------|-----|
| Project history | Agents don't care about why you migrated from Webpack |
| Team bios | Irrelevant to code generation |
| Meeting notes | Not actionable |
| Aspirational goals | "We want to eventually..." is not followable |
| Duplicate README content | Agents already read README.md |
| Lengthy design docs | Link to them instead; include only actionable rules |
| Obvious language features | "Use functions to organize code" adds nothing |

---

## Token Efficiency

Every word in AGENTS.md has a cost — it reduces available context for actual code.

### ❌ Verbose (62 tokens)

```markdown
When working on this project, please make sure that you always
use TypeScript instead of JavaScript, and ensure that strict
mode is enabled. We want to catch type errors at compile time.
```

### ✅ Efficient (14 tokens)

```markdown
- TypeScript strict mode. Never use JavaScript.
```

Same information, 4.4× fewer tokens.

### Budget Rule of Thumb

| AGENTS.md Size | Tokens | Context Budget Impact |
|---------------|--------|----------------------|
| 50 lines | ~200 tokens | < 1% of context — excellent |
| 100 lines | ~400 tokens | ~2% of context — good |
| 200 lines | ~800 tokens | ~4% of context — acceptable |
| 300 lines | ~1,200 tokens | ~6% of context — approaching limit |
| 500+ lines | ~2,000+ tokens | ~10%+ — agent performance degrades |

---

## The README.md vs AGENTS.md Contract

| README.md | AGENTS.md |
|-----------|-----------|
| Audience: Humans | Audience: AI Agents |
| Purpose: Understand the project | Purpose: Follow the rules |
| Style: Conversational, detailed | Style: Imperative, concise |
| Length: As long as needed | Length: Under 300 lines |
| Includes: History, motivation, screenshots | Includes: Commands, rules, boundaries |
| Links to: External docs, tutorials | Links to: Nothing (agents can't browse) |

⚠️ **Critical**: Agents cannot follow links. Never write "See our wiki for style guide." Include the rules inline.

---

*Next: [Section-by-Section Guide](section-by-section.md) →*
