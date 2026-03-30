# FAQ

> Frequently asked questions about prompt engineering.

---

## Basics

### Q: What's the single most important prompting tip?
**Be specific.** The more precisely you describe what you want, the better the result. "Fix the null reference on line 42" beats "fix this code" every time.

### Q: How long should a prompt be?
As long as it needs to be, no longer. Simple tasks: 1-2 sentences. Complex tasks: a structured prompt with context, constraints, and examples. System prompts: 500-1,100 tokens (10-15 rules).

### Q: Should I use markdown/XML in my prompts?
**XML tags** work great with Claude for separating sections. **Markdown** works well across all models. Both are better than unstructured prose for complex prompts.

### Q: Does capitalization (CAPS) help?
With older models, yes slightly. With modern models (Claude 4.6, GPT-4o), normal language works. CAPS can cause overtriggering.

---

## Techniques

### Q: When should I use Chain-of-Thought?
For tasks requiring reasoning: debugging, algorithm analysis, architecture decisions, complex logic. Not needed for simple generation, formatting, or renaming.

### Q: How many few-shot examples do I need?
3-5 is the sweet spot. Make them diverse (different cases, edge cases). One good example beats zero; going past 5 has diminishing returns.

### Q: Is zero-shot or few-shot better?
Try zero-shot first. If the output format or logic is wrong, add examples. Few-shot is more reliable but uses more tokens.

### Q: What about "Let's think step by step"?
It works for reasoning tasks (math, logic, debugging). It's wasteful for simple tasks. Modern models with extended thinking do this automatically.

---

## Coding

### Q: How do I get code that matches my project's style?
Reference existing files: "Follow the pattern in src/api/users.ts." This is more effective than describing the style.

### Q: How do I prevent the model from over-engineering?
Add: "Implement the simplest solution that meets the requirements. Do not add abstractions, flexibility, or features not explicitly requested."

### Q: Should I paste full files or relevant snippets?
Relevant snippets when possible. Full files when the model needs full context (e.g., refactoring, understanding dependencies). Use line ranges when you know the location.

### Q: How do I get the model to write good tests?
Specify: framework, mock strategy, test categories (happy path, validation, errors, edge cases), and the Arrange→Act→Assert pattern. Reference existing test files for style.

---

## Tools

### Q: Copilot instructions vs CLAUDE.md vs .cursorrules — which do I need?
Each tool reads its own instruction file. Create all that apply to your team. Keep content consistent but format tool-specific. See the [Tool Comparison](../../01-Meta-Prompting%20Framework%20%26%20AI-Assisted%20Development%20Guide/08-reference/tool-comparison.md) doc.

### Q: How many rules should my instructions file have?
10-15 critical rules. Beyond 15, compliance drops. Let linters handle style — use instructions for things linters can't check.

### Q: Do .prompt.md files work across tools?
Currently GitHub Copilot specific. The concept (reusable prompt templates) applies everywhere; the format may vary.

---

## Advanced

### Q: What is meta-prompting useful for?
Generating new prompts, improving existing ones, creating templates. Useful when you need many prompts and want consistency.

### Q: When should I use prompt chaining vs one big prompt?
Chain when: you need to inspect intermediate results, different steps need different expertise, or the task has clear sequential phases. One prompt when: the task is straightforward and doesn't need intermediate inspection.

### Q: Is prompt engineering becoming obsolete?
No — it's evolving into context engineering. The core skills (clarity, specificity, examples, constraints) remain essential even as models improve.
