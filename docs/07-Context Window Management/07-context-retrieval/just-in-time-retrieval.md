# Just-in-Time Retrieval

> Loading context on demand instead of up front — the key to efficient context management.

---

## The Concept

Instead of loading everything into context at the start, maintain **lightweight references** (file paths, queries, URLs) and load data when actually needed.

```
Traditional (Eager Loading):
  Start → Load all 50 files → Work → Run out of context

Just-in-Time (Lazy Loading):
  Start → Store file paths → Load file X when needed →
  Work on X → Clear X → Load file Y when needed → ...
```

---

## How Claude Code Does It

Claude Code uses this hybrid approach:
1. **CLAUDE.md** files are loaded upfront (small, high-signal context)
2. **`glob` and `grep`** discover files at runtime
3. **File reads** are targeted (specific lines, not entire files)
4. **`head` and `tail`** analyze large files without full loading
5. **Bash commands** query databases without loading full datasets

```
Agent thought process:
"I need to understand the auth module"
  → glob src/auth/**/*.ts     (find files, costs ~100 tokens)
  → grep "export" src/auth/   (find public API, costs ~200 tokens)
  → read src/auth/jwt.ts:1-30 (read just the exports, costs ~300 tokens)
  
Total: ~600 tokens instead of reading all auth files (~5,000 tokens)
```

---

## Implementing Just-in-Time

### 1. Store References, Not Content
```markdown
# In copilot-instructions.md (loaded upfront)
## Project Structure
- Authentication: src/auth/ (JWT-based, RS256)
- API routes: src/routes/
- Database models: src/models/
- Tests: tests/

When you need details about any module, read the specific files.
Do NOT ask me to paste file contents — use your tools.
```

### 2. Targeted Reads
```
❌ read_file("src/auth/jwt.ts")           → 1,500 tokens
✅ read_file("src/auth/jwt.ts", lines=1-20) → 200 tokens
```

### 3. Search Before Read
```
Step 1: grep "validateToken" src/auth/  → Find the right file + line
Step 2: Read only that section           → Minimal tokens
```

### 4. Clear After Use
```
Turn 5: Read file content (1,500 tokens)
Turn 6: AI analyzes and responds
Turn 7: Clear the raw file content (save 1,500 tokens)
```

---

## When to Use Eager vs. Just-in-Time

| Scenario | Strategy |
|----------|---------|
| Small project (<20 files) | Eager — fits in context |
| Large project (100+ files) | Just-in-time — load per task |
| Quick question about one file | Eager — read the file |
| Multi-file refactoring | JIT — read each file as needed |
| Configuration review | Eager — configs are small |
| Codebase analysis | JIT + search-first |

---

## Key Takeaways

1. **References are cheap, content is expensive** — Store paths, load on demand
2. **Search before read** — Find the right file/line before loading
3. **Targeted reads** — Specific lines, not entire files
4. **Clear after use** — Free up context once information is absorbed
5. **Hybrid works best** — Small context upfront + JIT for details
