# Prompt Engineering Patterns

> A comprehensive guide to prompt engineering techniques for AI-assisted software development — from fundamentals to advanced agentic patterns.

---

## Why Prompt Engineering Matters

The difference between a useful AI response and a useless one is almost always the prompt. Prompt engineering is the discipline of crafting inputs that reliably produce the outputs you need — correct code, accurate analysis, consistent formatting.

**Key insight**: Modern AI models are brilliant but lack your context. The prompt is how you bridge that gap.

---

## Table of Contents

### [01 — Fundamentals](01-fundamentals/)
| Document | Description |
|----------|-------------|
| [What Is Prompt Engineering?](01-fundamentals/what-is-prompt-engineering.md) | Definition, evolution, why it matters for developers |
| [Anatomy of a Prompt](01-fundamentals/prompt-anatomy.md) | The 5 components: instruction, context, examples, constraints, format |
| [Mental Models](01-fundamentals/mental-models.md) | How LLMs process prompts — attention, tokens, position effects |
| [Prompt vs Context Engineering](01-fundamentals/prompt-vs-context-engineering.md) | Where prompt engineering fits in the bigger picture |

### [02 — Core Techniques](02-core-techniques/)
| Document | Description |
|----------|-------------|
| [Zero-Shot Prompting](02-core-techniques/zero-shot-prompting.md) | Direct instructions without examples |
| [Few-Shot Prompting](02-core-techniques/few-shot-prompting.md) | Learning from in-context examples |
| [Chain-of-Thought](02-core-techniques/chain-of-thought.md) | Step-by-step reasoning for complex tasks |
| [Role Prompting](02-core-techniques/role-prompting.md) | Persona and expertise assignment |
| [Instruction Following](02-core-techniques/instruction-following.md) | Writing clear, actionable instructions |

### [03 — Structured Output](03-structured-output/)
| Document | Description |
|----------|-------------|
| [JSON Output Patterns](03-structured-output/json-output-patterns.md) | Reliable JSON generation techniques |
| [XML Structuring](03-structured-output/xml-structuring.md) | Using XML tags for prompt organization |
| [Schema Enforcement](03-structured-output/schema-enforcement.md) | Constraining output to match schemas |
| [Markdown Output](03-structured-output/markdown-output.md) | Generating well-structured documentation |

### [04 — Advanced Patterns](04-advanced-patterns/)
| Document | Description |
|----------|-------------|
| [Meta-Prompting](04-advanced-patterns/meta-prompting.md) | Prompts that generate or improve prompts |
| [Self-Consistency](04-advanced-patterns/self-consistency.md) | Multiple reasoning paths with voting |
| [Tree of Thought](04-advanced-patterns/tree-of-thought.md) | Branching exploration with backtracking |
| [Reflection & Critique](04-advanced-patterns/reflection-and-critique.md) | Self-evaluation and iterative improvement |
| [Decomposition](04-advanced-patterns/decomposition.md) | Breaking complex tasks into subtasks |

### [05 — Coding-Specific Patterns](05-coding-specific/)
| Document | Description |
|----------|-------------|
| [Code Generation Prompts](05-coding-specific/code-generation-prompts.md) | Writing effective prompts for code creation |
| [Refactoring Prompts](05-coding-specific/refactoring-prompts.md) | Prompts for code improvement and cleanup |
| [Debugging Prompts](05-coding-specific/debugging-prompts.md) | Prompts for finding and fixing bugs |
| [Test Generation Prompts](05-coding-specific/test-generation-prompts.md) | Prompts for writing comprehensive tests |
| [Code Review Prompts](05-coding-specific/code-review-prompts.md) | Prompts for thorough code review |

### [06 — System Prompts](06-system-prompts/)
| Document | Description |
|----------|-------------|
| [System Prompt Design](06-system-prompts/system-prompt-design.md) | Principles of effective system prompts |
| [Instruction Hierarchy](06-system-prompts/instruction-hierarchy.md) | Priority layers: system > developer > user |
| [Persona Engineering](06-system-prompts/persona-engineering.md) | Creating effective AI personas for development |
| [Constraint Specification](06-system-prompts/constraint-specification.md) | Defining boundaries and guardrails |

### [07 — Prompt Composition](07-prompt-composition/)
| Document | Description |
|----------|-------------|
| [Prompt Chaining](07-prompt-composition/prompt-chaining.md) | Sequential multi-step prompt workflows |
| [Prompt Routing](07-prompt-composition/prompt-routing.md) | Directing tasks to specialized prompts |
| [Dynamic Prompt Assembly](07-prompt-composition/dynamic-prompt-assembly.md) | Building prompts from components at runtime |

### [08 — ReAct & Agentic Patterns](08-react-and-agents/)
| Document | Description |
|----------|-------------|
| [ReAct Pattern](08-react-and-agents/react-pattern.md) | Reasoning + Acting interleaved framework |
| [Tool-Augmented Prompting](08-react-and-agents/tool-augmented-prompting.md) | Prompting models to use tools effectively |
| [Agentic Prompting](08-react-and-agents/agentic-prompting.md) | Prompts for autonomous AI agents |

### [09 — Best Practices](09-best-practices/)
| Document | Description |
|----------|-------------|
| [Clarity & Specificity](09-best-practices/clarity-and-specificity.md) | Being precise in what you ask for |
| [Context Management](09-best-practices/context-management-in-prompts.md) | Right amount of context for each task |
| [Iterative Refinement](09-best-practices/iterative-refinement.md) | Improving prompts through testing |
| [Tool-Specific Prompting](09-best-practices/tool-specific-prompting.md) | Copilot, Claude Code, Cursor tips |

### [10 — Anti-Patterns](10-anti-patterns/)
| Document | Description |
|----------|-------------|
| [Vague Instructions](10-anti-patterns/vague-instructions.md) | Why "make it better" fails |
| [Over-Constraining](10-anti-patterns/over-constraining.md) | Too many rules that conflict |
| [Prompt Injection Risks](10-anti-patterns/prompt-injection-risks.md) | Security considerations for prompts |
| [Cargo-Cult Prompting](10-anti-patterns/cargo-cult-prompting.md) | Copying prompts without understanding |

### [11 — Practical Resources](11-practical/)
| Document | Description |
|----------|-------------|
| [Templates](11-practical/templates.md) | Ready-to-use prompt templates |
| [Troubleshooting](11-practical/troubleshooting.md) | Common prompt issues and fixes |
| [FAQ](11-practical/faq.md) | Frequently asked questions |
| [Resources](11-practical/resources.md) | Links and further reading |

---

## Quick Start

1. **New to prompting?** → [What Is Prompt Engineering?](01-fundamentals/what-is-prompt-engineering.md)
2. **Want reliable output?** → [Chain-of-Thought](02-core-techniques/chain-of-thought.md)
3. **Need structured data?** → [JSON Output Patterns](03-structured-output/json-output-patterns.md)
4. **Writing code prompts?** → [Code Generation Prompts](05-coding-specific/code-generation-prompts.md)
5. **Building agents?** → [Agentic Prompting](08-react-and-agents/agentic-prompting.md)
