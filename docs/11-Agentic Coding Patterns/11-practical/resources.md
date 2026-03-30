# Agentic Coding Resources

> Curated resources for going deeper — official documentation, research papers, frameworks, tools, and community links.

---

## Official Documentation

### Anthropic

- **[Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)** — Anthropic's definitive guide to agent architecture. Covers the augmented LLM, workflow patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer), and when to go fully agentic. Essential reading.

- **[Claude Agent SDK](https://github.com/anthropics/anthropic-sdk-python)** — Python SDK for building Claude-powered agents with tool use, streaming, and multi-turn conversations.

- **[Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)** — Official docs for Claude Code, the terminal-native agentic coding tool. Covers CLAUDE.md, slash commands, MCP integration, and multi-agent patterns.

- **[Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook)** — Practical examples and recipes for building with Claude, including tool use, agents, and advanced prompting.

### GitHub Copilot

- **[GitHub Copilot Documentation](https://docs.github.com/en/copilot)** — Official documentation for all Copilot features including agent mode, coding agent, and extensions.

- **[Copilot CLI](https://githubnext.com/projects/copilot-cli)** — Documentation for the terminal-based Copilot agent with auto-compaction, skills, and custom agent support.

- **[Agent Skills Spec](https://github.com/agentskills/agentskills)** — Open specification for defining portable agent skills. Used by Copilot and compatible tools for `.github/skills/` folders.

- **[anthropics/skills](https://github.com/anthropics/skills)** — Anthropic's collection of reusable agent skills for common development tasks.

- **[github/awesome-copilot](https://github.com/github/awesome-copilot)** — Community-curated list of Copilot resources, extensions, tips, and integrations.

### MCP (Model Context Protocol)

- **[MCP Specification](https://modelcontextprotocol.io/)** — The open standard for connecting AI models to tools and data sources. Defines how agents discover and use external capabilities.

- **[MCP Servers Registry](https://github.com/modelcontextprotocol/servers)** — Directory of available MCP servers for databases, APIs, file systems, and more.

---

## Research

- **[SWE-bench](https://www.swebench.com/)** — Benchmark for evaluating AI coding agents on real GitHub issues. The standard evaluation for agentic coding capability. Tracks agent performance across repositories and issue types.

- **[ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)** — The foundational paper for the Reasoning + Acting pattern used in most coding agents. Demonstrates how interleaving reasoning traces with tool actions improves task completion.

- **[Toolformer: Language Models Can Teach Themselves to Use Tools](https://arxiv.org/abs/2302.04761)** — Research on how LLMs learn to use external tools effectively. Informs how agents decide when and how to invoke tools.

- **[Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs/2307.03172)** — Research demonstrating the U-shaped attention curve in LLMs. Explains why agents lose information in the middle of long contexts and informs context management strategies.

- **[Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366)** — How agents can learn from their mistakes within a session using verbal self-reflection. Basis for self-correction patterns in agentic workflows.

---

## Frameworks & SDKs

- **[Claude Agent SDK](https://github.com/anthropics/anthropic-sdk-python)** — Anthropic's official SDK for building Claude-powered agents. Supports tool use, streaming, structured output, and multi-turn agent loops.

- **[Strands Agents SDK](https://github.com/strands-agents/sdk-python)** — AWS-backed open-source SDK for building AI agents. Model-agnostic with built-in tool support, multi-agent coordination, and observability.

- **[Rivet](https://rivet.ironcladapp.com/)** — Visual programming environment for building AI agent workflows. Useful for prototyping complex chains and orchestration patterns before coding them.

- **[Vellum](https://www.vellum.ai/)** — Platform for building, testing, and deploying AI workflows. Includes evaluation tools, prompt management, and workflow visualization.

- **[LangGraph](https://github.com/langchain-ai/langgraph)** — Library for building stateful, multi-agent applications as graphs. Good for complex orchestration patterns with branching and cycles.

---

## Tools

### Agent-Specific

- **[GitHub Copilot Agent Mode](https://docs.github.com/en/copilot/using-github-copilot/using-copilot-agent-mode)** — Autonomous agent capabilities within VS Code / JetBrains. Iterates on code changes, runs terminal commands, and self-corrects.

- **[GitHub Copilot Coding Agent](https://docs.github.com/en/copilot/using-github-copilot/using-copilot-coding-agent)** — Assigns GitHub issues to Copilot for autonomous implementation in a dedicated environment.

- **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** — Terminal-native agentic coding tool. Reads/writes files, runs commands, and works through complex tasks autonomously.

### MCP Ecosystem

- **[MCP Server: Filesystem](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem)** — Read/write files with sandboxed access control.

- **[MCP Server: PostgreSQL](https://github.com/modelcontextprotocol/servers/tree/main/src/postgres)** — Query and manage PostgreSQL databases.

- **[MCP Server: GitHub](https://github.com/modelcontextprotocol/servers/tree/main/src/github)** — Interact with GitHub repositories, issues, and pull requests.

### Testing & Evaluation

- **[SWE-bench](https://github.com/princeton-nlp/SWE-bench)** — Evaluation harness for coding agents. Test your agent's ability to resolve real GitHub issues.

- **[Promptfoo](https://github.com/promptfoo/promptfoo)** — Open-source tool for testing and evaluating LLM outputs. Useful for building eval suites for agent workflows.

---

## Community

- **[GitHub Community Discussions](https://github.com/orgs/community/discussions/categories/copilot)** — Official discussion forum for GitHub Copilot. Share experiences, ask questions, and learn from other users.

- **[Anthropic Discord](https://discord.gg/anthropic)** — Community Discord for Claude users. Channels for Claude Code, agent development, and prompt engineering.

- **[r/ChatGPTCoding](https://www.reddit.com/r/ChatGPTCoding/)** — Reddit community for AI-assisted coding across all tools. Active discussions on agentic workflows.

- **[AI Engineer Community](https://www.latent.space/)** — Newsletter and community focused on building with LLMs. Covers agent architectures and coding tools.

---

## Cross-References: This Documentation Series

Navigate to related sections in this guide:

| Topic | Section | Description |
|-------|---------|-------------|
| **Fundamentals** | [Agentic Coding Fundamentals](../01-fundamentals/what-is-agentic-coding.md) | Core concepts and the agentic spectrum |
| **Prompt Engineering** | [Prompt Engineering for Agents](../01-fundamentals/agent-computer-interface.md) | Designing effective agent-computer interfaces |
| **Workflow Patterns** | [Prompt Chaining](../02-workflow-patterns/prompt-chaining.md) | Building reliable multi-step workflows |
| **Agent Architectures** | [Agent Architectures](../03-agent-architectures/react-pattern.md) | ReAct, tool use, and planning patterns |
| **Tool Use** | [Tool Use & MCP](../04-tool-use/mcp-fundamentals.md) | MCP servers, tool design, and integration |
| **Multi-Agent** | [Multi-Agent Systems](../05-multi-agent/orchestrator-workers.md) | Orchestrator-workers and parallel patterns |
| **Copilot Patterns** | [Copilot Agent Patterns](../06-copilot-agent-patterns/copilot-instructions.md) | copilot-instructions.md, agent mode, coding agent |
| **Claude Code** | [Claude Code Patterns](../07-claude-code-patterns/claude-md.md) | CLAUDE.md, slash commands, hooks |
| **Skills** | [Skills & Custom Agents](../08-skills-and-custom-agents/skill-creation.md) | Building custom skills and agents |
| **Best Practices** | [Best Practices](../09-best-practices/testing-strategy.md) | Testing, security, context optimization |
| **Anti-Patterns** | [Anti-Patterns](../10-anti-patterns/over-engineering.md) | What to avoid in agentic workflows |
| **Context Window** | [Context Window Optimization](../09-best-practices/context-window-optimization.md) | Managing context in long sessions |
| **Security** | [Security Best Practices](../09-best-practices/security.md) | Securing agentic workflows |
| **Testing** | [Testing Strategy](../09-best-practices/testing-strategy.md) | Testing agent-generated code |

---

## Recommended Reading Order

If you're new to agentic coding, follow this path:

```
1. Anthropic's "Building Effective Agents" (external)
2. This series: 01-Fundamentals → 02-Workflow Patterns
3. Pick your tool: 06-Copilot Patterns OR 07-Claude Code Patterns
4. 09-Best Practices (testing, security, context)
5. 10-Anti-Patterns (what to avoid)
6. 11-Practical (templates, troubleshooting)
7. Deep dives as needed: Multi-Agent, Skills, MCP
```
