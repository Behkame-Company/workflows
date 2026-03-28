# Tool-Specific Prompting

> Tips and patterns for GitHub Copilot CLI, Claude Code, Cursor, and ChatGPT.

---

## GitHub Copilot CLI

### What Works Well
- **Direct, task-focused prompts**: "Add error handling to the login function in src/auth/service.ts"
- **Referencing project files**: Copilot reads copilot-instructions.md and .instructions.md automatically
- **Using .prompt.md files**: Reusable prompts with variables (`{{endpoint}}`, `{{model}}`)
- **Agent mode**: Let Copilot use tools (file reads, terminal commands, grep)

### Tips
```
- Keep copilot-instructions.md under 500 lines
- Use path-specific .instructions.md for directory-level rules
- Create .prompt.md files for repeated tasks
- In agent mode, let it explore — don't micromanage tool use
- Use # to reference files in chat: "Refactor #src/auth/service.ts"
```

### Avoid
- Overly long system instructions (Copilot has limited context)
- Asking it to maintain state across sessions (use files instead)
- Expecting it to run long-running processes

---

## Claude Code

### What Works Well
- **CLAUDE.md** for persistent project instructions
- **Extended thinking** for complex reasoning tasks
- **Agentic workflows**: Claude Code is designed for multi-step tasks
- **Subagent delegation**: Claude naturally delegates to subagents
- **Memory tool**: Persistent cross-session notes

### Tips
```
- Use CLAUDE.md (root) + CLAUDE.md in subdirectories
- Let Claude use bash for verification (tests, builds)
- Use "/compact" to manage long sessions
- Claude 4.6 follows normal-language instructions — no need for "CRITICAL" emphasis
- Keep CLAUDE.md focused: 10-15 rules, not 50
```

### Avoid
- Over-prompting tool use ("ALWAYS use grep before...")
- Leaving aggressive anti-laziness prompts (4.6 models are proactive)
- Forgetting to use memory tool for cross-session state

---

## Cursor

### What Works Well
- **.cursorrules** for project-level instructions
- **@ references**: @file, @codebase, @docs for context injection
- **Composer mode** for multi-file changes
- **Tab completion**: Context-aware inline suggestions

### Tips
```
- Keep .cursorrules concise (same principles as copilot-instructions.md)
- Use @file references to inject specific context
- Use Composer for multi-file changes, Chat for single-file
- Be explicit about which files to modify in prompts
```

### Avoid
- Relying on Cursor to find files (reference them explicitly)
- Mixing generation and review in one prompt
- Expecting long-context memory without explicit state files

---

## ChatGPT / OpenAI API

### What Works Well
- **System message**: Strong adherence to system prompt instructions
- **JSON mode**: `response_format: { type: "json_object" }`
- **Structured outputs**: Pydantic schema enforcement
- **Function calling**: Tool definitions with schemas

### Tips
```
- Use the system message for all persistent instructions
- Enable JSON mode for structured output tasks
- Use function calling for tool-augmented workflows
- Temperature 0 for code generation, 0.3-0.5 for general tasks
```

### Avoid
- Long system messages without structure (use sections/headers)
- Expecting state across separate API calls (stateless)
- Relying on training data for project-specific knowledge

---

## Cross-Tool Best Practices

| Practice | Works Everywhere |
|----------|:----------------:|
| Be specific about what you want | ✅ |
| Use examples for format control | ✅ |
| Reference project files by path | ✅ |
| Keep instructions under 15 rules | ✅ |
| Provide success criteria | ✅ |
| Test and iterate on prompts | ✅ |

---

## Next Steps

- 🔗 [System Prompt Design](../06-system-prompts/system-prompt-design.md) — Platform-agnostic design
- 🔗 [Iterative Refinement](iterative-refinement.md) — Improving prompts for any tool
