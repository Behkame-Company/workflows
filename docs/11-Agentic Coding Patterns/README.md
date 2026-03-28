# 11 — Agentic Coding Patterns

> Comprehensive guide to building with AI coding agents — from workflow patterns and multi-agent orchestration to tool use, custom agents, and autonomous development workflows.

---

## Why This Section Exists

AI coding has evolved from autocomplete to **autonomous agents** that can plan, execute, test, and iterate on complex tasks. Understanding agentic patterns — how to structure workflows, orchestrate multiple agents, design tool interfaces, and build custom agents — is essential for getting reliable, high-quality results from AI coding tools.

This section draws heavily from Anthropic's "Building Effective Agents" research and practical patterns from GitHub Copilot CLI, Claude Code, and the broader agentic AI ecosystem.

---

## Table of Contents

### 01 — Fundamentals
- [What Are Agentic Systems?](01-fundamentals/what-are-agentic-systems.md) — Workflows vs agents, when to use each
- [The Augmented LLM](01-fundamentals/augmented-llm.md) — The foundational building block: LLM + tools + memory
- [When to Go Agentic](01-fundamentals/when-to-go-agentic.md) — Complexity tradeoffs and decision framework
- [Agent-Computer Interface](01-fundamentals/agent-computer-interface.md) — Designing how agents interact with your system

### 02 — Workflow Patterns
- [Prompt Chaining](02-workflow-patterns/prompt-chaining.md) — Sequential decomposition with gates
- [Routing](02-workflow-patterns/routing.md) — Classification and specialized handling
- [Parallelization](02-workflow-patterns/parallelization.md) — Sectioning and voting patterns
- [Orchestrator-Workers](02-workflow-patterns/orchestrator-workers.md) — Dynamic task decomposition and delegation
- [Evaluator-Optimizer](02-workflow-patterns/evaluator-optimizer.md) — Generate-evaluate-refine loops

### 03 — Agent Architectures
- [Autonomous Agents](03-agent-architectures/autonomous-agents.md) — Self-directed task completion
- [ReAct Pattern](03-agent-architectures/react-pattern.md) — Reasoning + Acting in a loop
- [Plan-and-Execute](03-agent-architectures/plan-and-execute.md) — Planning before acting
- [Reflection and Self-Critique](03-agent-architectures/reflection.md) — Agents that evaluate their own work

### 04 — Tool Use
- [Tool Design Principles](04-tool-use/tool-design-principles.md) — Building effective agent-computer interfaces
- [Shell and File Tools](04-tool-use/shell-and-file-tools.md) — Command execution and file manipulation
- [MCP Tools Integration](04-tool-use/mcp-tools-integration.md) — Model Context Protocol in agentic workflows
- [Custom Tool Development](04-tool-use/custom-tool-development.md) — Building tools for your agents

### 05 — Multi-Agent Systems
- [Multi-Agent Patterns](05-multi-agent/multi-agent-patterns.md) — Coordinator, pipeline, debate, and swarm
- [Initializer-Worker Pattern](05-multi-agent/initializer-worker.md) — Scaffolding agent + coding agents
- [Agent Communication](05-multi-agent/agent-communication.md) — How agents share context and results
- [Parallel Agent Execution](05-multi-agent/parallel-execution.md) — Running agents concurrently

### 06 — Copilot Agent Patterns
- [Copilot Coding Agent](06-copilot-agent-patterns/coding-agent.md) — GitHub's cloud-based autonomous agent
- [Copilot CLI Agent Mode](06-copilot-agent-patterns/cli-agent-mode.md) — Local autonomous workflows
- [Custom Instructions for Agents](06-copilot-agent-patterns/custom-instructions.md) — Configuring agent behavior
- [Copilot Skills](06-copilot-agent-patterns/copilot-skills.md) — Building specialized agent capabilities

### 07 — Claude Code Patterns
- [Claude Code Workflows](07-claude-code-patterns/workflows.md) — Common patterns and best practices
- [Multi-Window Development](07-claude-code-patterns/multi-window.md) — Parallel agent sessions
- [CLAUDE.md for Agents](07-claude-code-patterns/claude-md-agents.md) — Configuring Claude Code as an agent
- [Headless and CI Mode](07-claude-code-patterns/headless-ci.md) — Non-interactive agent execution

### 08 — Skills & Custom Agents
- [Agent Skills Specification](08-skills-and-custom-agents/skills-spec.md) — The open standard for agent capabilities
- [Building Custom Skills](08-skills-and-custom-agents/building-skills.md) — Creating reusable agent skills
- [Custom Agent Design](08-skills-and-custom-agents/custom-agent-design.md) — Designing specialized agents
- [Agent Composition](08-skills-and-custom-agents/agent-composition.md) — Combining skills and agents

### 09 — Best Practices
- [Start Simple](09-best-practices/start-simple.md) — Simplicity over complexity
- [Transparency and Observability](09-best-practices/transparency.md) — Making agent behavior visible
- [Error Recovery](09-best-practices/error-recovery.md) — Handling failures gracefully
- [Testing Agentic Systems](09-best-practices/testing-agents.md) — Evaluating agent behavior

### 10 — Anti-Patterns
- [Over-Engineering Agents](10-anti-patterns/over-engineering.md) — Adding complexity without value
- [Infinite Loops](10-anti-patterns/infinite-loops.md) — Agents stuck in retry cycles
- [Context Starvation](10-anti-patterns/context-starvation.md) — Agents running out of context
- [Blind Delegation](10-anti-patterns/blind-delegation.md) — Trusting agents without verification

### 11 — Practical
- [Templates](11-practical/templates.md) — Agent configuration templates
- [Troubleshooting](11-practical/troubleshooting.md) — Common agent issues and fixes
- [FAQ](11-practical/faq.md) — Frequently asked questions
- [Resources](11-practical/resources.md) — Links, tools, and further reading

---

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Workflow** | LLMs orchestrated through predefined code paths |
| **Agent** | LLMs that dynamically direct their own processes and tool usage |
| **Augmented LLM** | LLM enhanced with retrieval, tools, and memory |
| **ACI** | Agent-Computer Interface — how agents interact with systems |
| **Prompt Chaining** | Sequential LLM calls where each processes previous output |
| **Orchestrator-Workers** | Central LLM delegates subtasks to worker LLMs |
| **ReAct** | Reasoning + Acting loop for autonomous problem solving |
| **Skills** | Folders of instructions, scripts, and resources for specialized tasks |

---

## Anthropic's Three Core Principles

From "Building Effective Agents":

1. **Simplicity** — Maintain simplicity in your agent's design
2. **Transparency** — Explicitly show the agent's planning steps
3. **ACI Quality** — Invest in agent-computer interface design through thorough tool documentation and testing

---

## Quick Start

1. **New to agents?** → [What Are Agentic Systems?](01-fundamentals/what-are-agentic-systems.md)
2. **Choosing a pattern?** → [When to Go Agentic](01-fundamentals/when-to-go-agentic.md)
3. **Using Copilot?** → [Copilot CLI Agent Mode](06-copilot-agent-patterns/cli-agent-mode.md)
4. **Using Claude Code?** → [Claude Code Workflows](07-claude-code-patterns/workflows.md)
5. **Building skills?** → [Agent Skills Specification](08-skills-and-custom-agents/skills-spec.md)

---

## Cross-References

| Related Section | Connection |
|----------------|------------|
| [Prompt Engineering Patterns](../08-Prompt%20Engineering%20Patterns/) | Prompt patterns used within agents |
| [Context Window Management](../07-Context%20Window%20Management/) | Managing agent context across turns |
| [MCP Server](../05-MCP%20Server/) | MCP tools powering agent capabilities |
| [Security & Guardrails](../10-Security%20%26%20Guardrails/) | Securing agent permissions and execution |
| [Testing Strategy](../09-Testing%20Strategy%20with%20AI/) | Testing agent-generated code |

---

*45 documents across 11 sections — from workflow patterns to custom agent design.*
