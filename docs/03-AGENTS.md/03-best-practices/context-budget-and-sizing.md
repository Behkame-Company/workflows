# Context Budget & File Sizing

> Understanding token limits, measuring AGENTS.md cost, and optimizing for maximum agent performance.

---

## How AGENTS.md Uses Your Context Budget

AI coding agents have finite context windows. Everything loaded into context competes for space:

```
┌──────────────────────────────────┐
│         Context Window            │
│ (128K-200K tokens, depending)     │
├──────────────────────────────────┤
│ System Prompt (tool-specific)     │  ~2,000 tokens
│ AGENTS.md content                 │  ~200-1,200 tokens ← YOU CONTROL THIS
│ Task description (issue/prompt)   │  ~200-1,000 tokens
│ Files being read/edited           │  ~10,000-50,000 tokens
│ Tool output (test results, etc.)  │  ~5,000-20,000 tokens
│ Agent reasoning + history         │  ~10,000-50,000 tokens
│ Remaining capacity                │  Available for generation
└──────────────────────────────────┘
```

AGENTS.md typically uses **0.1% to 1%** of the context window. This is small, but every token counts when the agent is working with large files.

---

## Sizing Guidelines

### The Token Math

| Metric | Approximate Value |
|--------|-------------------|
| 1 word | ~1.3 tokens |
| 1 line of Markdown | ~10-15 tokens |
| 50-line AGENTS.md | ~200 tokens |
| 100-line AGENTS.md | ~400 tokens |
| 200-line AGENTS.md | ~800 tokens |
| 300-line AGENTS.md | ~1,200 tokens |
| 500-line AGENTS.md | ~2,000 tokens |

### Recommended Sizing

| Project Type | Lines | Tokens | Rating |
|-------------|-------|--------|--------|
| Small project / library | 20-50 | 80-200 | ✅ Ideal |
| Standard application | 50-100 | 200-400 | ✅ Good |
| Large application | 100-150 | 400-600 | ⚠️ Acceptable |
| Monorepo (root file) | 30-50 | 120-200 | ✅ Root should be lean |
| Monorepo (per-package) | 30-80 | 120-320 | ✅ Package-specific |
| Enterprise (root + packages) | 50 + 50 per package | 200 + 200 each | ✅ Distributed |

### The Performance Cliff

Research shows a clear relationship between file size and agent compliance:

```
Compliance
  100% ┤
   90% ┤ ████████
   80% ┤ ████████████████
   70% ┤ ██████████████████████
   60% ┤ ████████████████████████████
   50% ┤ ██████████████████████████████████
   40% ┤ ██████████████████████████████████████████
       ├──────────┬──────────┬──────────┬──────────
       50        100        200        300+ lines
```

**Key inflection point**: Compliance drops sharply above ~150 lines.

---

## Optimization Strategies

### Strategy 1: The 80/20 Audit

Most AGENTS.md files follow the 80/20 rule: 20% of the content drives 80% of the value.

**Audit steps:**
1. Read every line of your AGENTS.md
2. Ask: "Has an agent ever needed this specific rule?"
3. If the answer is "no" or "maybe" — remove it
4. Keep only rules that directly prevent errors you've seen

### Strategy 2: Replace Prose with Bullets

```markdown
❌ BEFORE (45 tokens):
When you're working on API routes in this project, please ensure that
all input is validated using Zod schemas and that responses follow our
standard format with proper HTTP status codes.

✅ AFTER (18 tokens):
## API Rules
- Validate input: Zod
- Response: `Response.json(data, { status })` format
```

**Saving: 60% fewer tokens, same information.**

### Strategy 3: Replace Descriptions with Code Examples

```markdown
❌ BEFORE (35 tokens):
Components should be functional, use TypeScript, have an explicit
Props interface, and use named exports.

✅ AFTER (25 tokens + more information):
## Component Pattern
```typescript
interface ButtonProps {
  label: string;
  onClick: () => void;
}
export function Button({ label, onClick }: ButtonProps) {
  return <button onClick={onClick}>{label}</button>;
}
```
```

The code example communicates even more than the prose while using fewer tokens.

### Strategy 4: Split for Monorepos

Instead of one 300-line file, use:
- Root AGENTS.md: 40 lines (shared rules)
- Per-package AGENTS.md: 40-60 lines each

Total content may be more, but each agent invocation only loads the relevant ~80-100 lines.

### Strategy 5: Move Documentation to Comments

Don't use AGENTS.md for information that belongs in code:

```markdown
❌ In AGENTS.md (wastes tokens globally):
"The formatCurrency helper takes cents as integer and returns
formatted string with $ prefix and 2 decimal places."

✅ In the source file (discovered contextually):
/**
 * @param cents - Amount in cents (integer)
 * @returns Formatted currency string (e.g., "$12.34")
 */
export function formatCurrency(cents: number): string { ... }
```

---

## Measuring Your Budget

### Quick Estimate

```bash
# Count lines
wc -l AGENTS.md

# Estimate tokens (rough: words × 1.3)
wc -w AGENTS.md
```

On Windows:
```powershell
(Get-Content AGENTS.md | Measure-Object -Line).Lines
(Get-Content AGENTS.md | Measure-Object -Word).Words
```

### With Tiktoken (Exact)

```python
import tiktoken
enc = tiktoken.encoding_for_model("gpt-4")
with open("AGENTS.md") as f:
    tokens = enc.encode(f.read())
print(f"Tokens: {len(tokens)}")
```

---

## Tool-Specific Budget Notes

| Tool | Context Window | AGENTS.md Budget |
|------|---------------|-----------------|
| GitHub Copilot (chat) | ~128K tokens | Keep under 400 tokens (~100 lines) |
| Copilot Coding Agent | ~200K tokens | Up to 600 tokens (~150 lines) OK |
| Claude Code (Sonnet) | ~200K tokens | Up to 800 tokens (~200 lines) OK |
| Claude Code (Opus) | ~200K tokens | Up to 800 tokens (~200 lines) OK |
| Codex CLI | ~128K tokens | Keep under 400 tokens (~100 lines) |
| Cursor | ~128K tokens | Keep under 400 tokens (~100 lines) |

**General rule**: Keep AGENTS.md under 0.5% of the tool's context window.

---

*Next: [Maintaining Over Time](maintaining-over-time.md) →*
