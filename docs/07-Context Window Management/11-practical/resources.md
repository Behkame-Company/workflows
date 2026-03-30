# Resources

> Links, tools, and further reading for context window management.

---

## Essential Reading

| Resource | Topic |
|----------|-------|
| [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Anthropic's definitive guide — context as finite resource |
| [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) | Multi-window workflows, initializer pattern |
| [Claude 4 Best Practices](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices) | Official prompting guide with context management |
| [Context Windows Documentation](https://docs.anthropic.com/en/docs/build-with-claude/context-windows) | Technical reference for context management |
| [Server-Side Compaction](https://docs.anthropic.com/en/build-with-claude/compaction) | API reference for compaction |
| [Memory Tool](https://docs.anthropic.com/en/agents-and-tools/tool-use/memory-tool) | Persistent memory across sessions |

---

## Official Documentation

| Tool | Context Management Docs |
|------|------------------------|
| **Anthropic Claude** | [Context Windows](https://docs.anthropic.com/en/docs/build-with-claude/context-windows) |
| **GitHub Copilot** | [Best Practices](https://docs.github.com/en/copilot/using-github-copilot/best-practices-for-using-github-copilot) |
| **Copilot Instructions** | [Add Repository Instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions) |
| **Cline Memory Bank** | [Memory Bank Features](https://docs.cline.bot/features/memory-bank) |

---

## Tools

| Tool | Purpose |
|------|---------|
| **Anthropic Token Counting API** | Precise token counting before sending |
| **tiktoken** (Python) | Token counting for OpenAI models |
| **Anthropic Compaction API** | Server-side context summarization |
| **Memory Tool** | Persistent storage across sessions |

---

## Key Concepts

| Concept | Source |
|---------|--------|
| Context Rot | [Chroma Research](https://research.trychroma.com/context-rot) |
| MRCR Benchmark | [arXiv paper](https://arxiv.org/abs/2501.03276) |
| Attention Mechanism | [Attention Is All You Need](https://arxiv.org/abs/1706.03762) |
| Working Memory Limits | [Cowan (2010)](https://journals.sagepub.com/doi/abs/10.1177/0963721409359277) |
