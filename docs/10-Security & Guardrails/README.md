# 10 — Security & Guardrails for AI-Assisted Development

> Comprehensive guide to securing AI coding workflows — from prompt injection defense to secrets management, sandboxing, and the OWASP LLM Top 10.

---

## Why This Section Exists

AI coding agents can **read your files**, **execute commands**, and **modify your codebase**. That power demands rigorous security thinking. This section covers every security dimension of AI-assisted development: protecting against prompt injection, preventing secret leaks, sandboxing agent execution, securing MCP servers, and building organizational guardrails.

---

## Table of Contents

### 01 — Fundamentals
- [Why AI Security Is Different](01-fundamentals/why-ai-security-is-different.md) — Unique threat model of AI coding agents
- [Threat Modeling for AI Workflows](01-fundamentals/threat-modeling.md) — Identifying attack surfaces in AI-assisted development
- [Defense in Depth](01-fundamentals/defense-in-depth.md) — Layered security for AI pipelines
- [Trust Boundaries](01-fundamentals/trust-boundaries.md) — Where human, AI, and system trust intersect

### 02 — Prompt Injection
- [What Is Prompt Injection?](02-prompt-injection/what-is-prompt-injection.md) — Direct and indirect injection attacks
- [Injection in Coding Tools](02-prompt-injection/injection-in-coding-tools.md) — How injection applies to Copilot, Claude Code, and agents
- [Defense Strategies](02-prompt-injection/defense-strategies.md) — Input validation, output filtering, instruction hierarchy
- [Tool Poisoning Attacks](02-prompt-injection/tool-poisoning.md) — MCP tool description injection and rug pulls

### 03 — Code Safety
- [AI-Generated Code Risks](03-code-safety/ai-generated-code-risks.md) — Common vulnerabilities in AI output
- [Static Analysis Integration](03-code-safety/static-analysis.md) — Automated scanning of AI-generated code
- [Secure Coding Instructions](03-code-safety/secure-coding-instructions.md) — Teaching AI agents to write secure code
- [Code Review for AI Output](03-code-safety/code-review-ai-output.md) — Human review patterns for AI-generated code

### 04 — Secrets Management
- [Secrets in AI Workflows](04-secrets-management/secrets-in-ai-workflows.md) — How AI agents encounter and handle secrets
- [Preventing Secret Leaks](04-secrets-management/preventing-leaks.md) — Guardrails against credential exposure
- [Environment Variable Safety](04-secrets-management/environment-variables.md) — Secure environment configuration for AI agents
- [Git Secrets and Pre-Commit](04-secrets-management/git-secrets.md) — Automated secret detection in commits

### 05 — Sandboxing & Permissions
- [Agent Permission Models](05-sandboxing-permissions/permission-models.md) — Copilot CLI allow/deny tools, Claude Code permissions
- [Sandboxed Execution](05-sandboxing-permissions/sandboxed-execution.md) — Containers, VMs, and restricted environments
- [Least Privilege for AI](05-sandboxing-permissions/least-privilege.md) — Principle of minimal necessary access
- [Network Access Control](05-sandboxing-permissions/network-access.md) — Controlling agent internet and API access

### 06 — OWASP LLM Top 10
- [OWASP LLM Top 10 Overview](06-owasp-llm-top-10/overview.md) — All 10 risks mapped to coding workflows
- [Prompt Injection (LLM01)](06-owasp-llm-top-10/llm01-prompt-injection.md) — Deep dive with coding-specific examples
- [Insecure Output Handling (LLM02)](06-owasp-llm-top-10/llm02-insecure-output.md) — Validating AI-generated code and commands
- [Excessive Agency (LLM08)](06-owasp-llm-top-10/llm08-excessive-agency.md) — When agents have too much autonomy

### 07 — MCP Security
- [MCP Threat Landscape](07-mcp-security/mcp-threat-landscape.md) — Attack surfaces in Model Context Protocol
- [Tool Shadowing & Rug Pulls](07-mcp-security/tool-shadowing.md) — Cross-server attacks and tool redefinition
- [Securing MCP Servers](07-mcp-security/securing-mcp-servers.md) — Building secure MCP implementations
- [MCP Client Security](07-mcp-security/mcp-client-security.md) — UI transparency and approval workflows

### 08 — Supply Chain
- [AI Supply Chain Risks](08-supply-chain/ai-supply-chain-risks.md) — Dependencies, models, and training data
- [Dependency Safety](08-supply-chain/dependency-safety.md) — AI-suggested package verification
- [Model Provenance](08-supply-chain/model-provenance.md) — Verifying model integrity and updates
- [Extension & Plugin Safety](08-supply-chain/extension-safety.md) — Vetting AI tool extensions

### 09 — Best Practices
- [Security-First AI Instructions](09-best-practices/security-first-instructions.md) — Writing security-aware custom instructions
- [Automated Security Gates](09-best-practices/automated-security-gates.md) — CI/CD security checks for AI code
- [Incident Response for AI](09-best-practices/incident-response.md) — When AI introduces a vulnerability
- [Security Training for AI Teams](09-best-practices/security-training.md) — Building security awareness

### 10 — Anti-Patterns
- [Trusting AI Blindly](10-anti-patterns/trusting-ai-blindly.md) — The overreliance trap
- [Security by Obscurity](10-anti-patterns/security-by-obscurity.md) — Hidden prompts aren't secure
- [Overpermissioned Agents](10-anti-patterns/overpermissioned-agents.md) — The --allow-all-tools danger
- [Ignoring AI in Threat Models](10-anti-patterns/ignoring-ai-in-threat-models.md) — Failing to account for AI attack surface

### 11 — Practical
- [Templates](11-practical/templates.md) — Security instruction templates and checklists
- [Troubleshooting](11-practical/troubleshooting.md) — Common security issues and fixes
- [FAQ](11-practical/faq.md) — Frequently asked security questions
- [Resources](11-practical/resources.md) — Links, tools, and further reading

---

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Prompt Injection** | Manipulating AI behavior through crafted inputs hidden in data |
| **Tool Poisoning** | Malicious instructions embedded in MCP tool descriptions |
| **Rug Pull** | MCP tools that silently change behavior after initial approval |
| **Excessive Agency** | AI agents with more autonomy/permissions than necessary |
| **Data Exfiltration** | Extracting private data through AI agent tool calls |
| **Supply Chain Attack** | Compromised dependencies, models, or extensions |

---

## Quick Start

1. **New to AI security?** → [Why AI Security Is Different](01-fundamentals/why-ai-security-is-different.md)
2. **Worried about injection?** → [What Is Prompt Injection?](02-prompt-injection/what-is-prompt-injection.md)
3. **Setting up permissions?** → [Agent Permission Models](05-sandboxing-permissions/permission-models.md)
4. **Using MCP?** → [MCP Threat Landscape](07-mcp-security/mcp-threat-landscape.md)
5. **Want a checklist?** → [Templates](11-practical/templates.md)

---

## Cross-References

| Related Section | Connection |
|----------------|------------|
| [Copilot Instructions](../Copilot%20instructions/) | Security rules in custom instructions |
| [MCP Server](../05-MCP%20Server/) | MCP architecture and building servers |
| [AI Code Review & CI/CD](../06-AI%20Code%20Review%20%26%20CI-CD%20Integration/) | Security gates in CI/CD pipelines |
| [Context Window Management](../07-Context%20Window%20Management/) | Context poisoning and injection via context |
| [Testing Strategy](../09-Testing%20Strategy%20with%20AI/) | Security testing and verification |

---

*43 documents across 11 sections — from threat modeling to incident response.*
