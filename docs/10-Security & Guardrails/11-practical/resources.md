# Security Resources & Further Reading

> A curated collection of tools, documentation, research, standards, and community resources for securing AI-assisted development workflows — from prompt injection defense to supply chain hardening.

---

## Why This Page Exists

AI security is a fast-moving field. New attack vectors, defensive tools, and best practices emerge regularly. This resource guide gives you a **single starting point** to find authoritative references, practical tools, and ongoing learning opportunities — all filtered through the lens of AI-assisted software development.

Every resource here has been selected because it directly applies to the security challenges covered in this documentation series.

---

## Official Documentation

These are primary sources from the organizations building AI tools, defining security standards, and publishing authoritative guidance.

### OWASP LLM Top 10

- **URL:** [https://genai.owasp.org/](https://genai.owasp.org/)
- **Why it matters:** The OWASP Top 10 for Large Language Model Applications is the industry-standard risk taxonomy for LLM security. It defines the ten most critical vulnerabilities — from prompt injection to excessive agency — and maps directly to every AI coding tool you use. This should be your **first stop** for understanding the threat landscape.
- **What to read first:** The full Top 10 list, then the detailed descriptions for LLM01 (Prompt Injection) and LLM08 (Excessive Agency) — these are the two most relevant to coding workflows.

### GitHub Copilot Responsible AI

- **URL:** [https://docs.github.com/en/copilot/responsible-use-of-github-copilot-features](https://docs.github.com/en/copilot/responsible-use-of-github-copilot-features)
- **Why it matters:** GitHub's official documentation covers how Copilot handles data, privacy boundaries, and responsible usage patterns. Understanding what Copilot does and doesn't do with your code is foundational to any security assessment.
- **Key topics:** Data handling policies, telemetry controls, organizational settings, content filtering, and the permission model for Copilot CLI tools.

### Anthropic AI Safety Documentation

- **URL:** [https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/mitigate-risks](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/mitigate-risks)
- **Why it matters:** Anthropic publishes detailed research on AI safety, including constitutional AI methods, prompt injection defense techniques, and model behavior analysis. Their documentation on risk mitigation for prompt engineering is especially valuable for teams crafting custom instructions and system prompts.
- **Key topics:** Prompt injection mitigation, system prompt best practices, output validation, and responsible deployment patterns.

### MCP Specification — Security Considerations

- **URL:** [https://modelcontextprotocol.io/specification](https://modelcontextprotocol.io/specification)
- **Why it matters:** The Model Context Protocol (MCP) specification includes a dedicated security section that defines the trust model, permission requirements, and threat mitigations for MCP servers and clients. If you're building or consuming MCP tools, this is the authoritative reference.
- **Key topics:** Tool approval workflows, transport security, capability negotiation, and the principle of least privilege for tool access.

### Microsoft Responsible AI Principles

- **URL:** [https://www.microsoft.com/en-us/ai/responsible-ai](https://www.microsoft.com/en-us/ai/responsible-ai)
- **Why it matters:** Microsoft's Responsible AI framework provides the governance principles behind GitHub Copilot and Azure AI services. Understanding these principles helps you align your organizational AI policies with the guardrails already built into the tools you use.
- **Key topics:** Fairness, reliability and safety, privacy and security, inclusiveness, transparency, and accountability.

---

## Research & Analysis

These sources provide deep technical analysis, ongoing investigation, and real-world case studies of AI security issues.

### Simon Willison's Blog — Prompt Injection & AI Security

- **URL:** [https://simonwillison.net/series/prompt-injection/](https://simonwillison.net/series/prompt-injection/)
- **Why it matters:** Simon Willison has been one of the most prolific researchers and writers on prompt injection since the attack class was first identified. His blog provides accessible yet technically rigorous analysis of new attack techniques, real-world incidents, and defensive strategies. He consistently covers AI security issues as they emerge.
- **Recommended reading:**
  - "Prompt injection and jailbreaking are not the same thing"
  - "You can't solve AI security problems with more AI"
  - Coverage of indirect prompt injection via tool outputs

### Invariant Labs — MCP Security Research

- **URL:** [https://invariantlabs.ai/](https://invariantlabs.ai/)
- **Why it matters:** Invariant Labs conducted some of the earliest and most thorough security research on the Model Context Protocol. Their work uncovered critical vulnerabilities including tool poisoning attacks, rug pulls (where tool behavior changes after initial approval), and cross-server shadowing. This research directly informed the MCP security section of this documentation.
- **Key contributions:**
  - Tool poisoning attack demonstrations
  - Rug pull vulnerability disclosure
  - Cross-origin tool shadowing analysis
  - Recommendations for MCP client security

### Trail of Bits — AI/ML Security Research

- **URL:** [https://blog.trailofbits.com/category/machine-learning/](https://blog.trailofbits.com/category/machine-learning/)
- **Why it matters:** Trail of Bits is a leading security research firm with a growing focus on AI/ML security. Their research bridges traditional application security with new AI-specific threats, making it particularly valuable for security engineers evaluating AI coding tools.
- **Key topics:** Model security, supply chain integrity, adversarial machine learning, and secure deployment patterns.

### Academic Research on LLM Security

Key papers for understanding the theoretical foundations of AI security in coding contexts:

- **"Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection"** (Greshake et al., 2023) — The foundational paper on indirect prompt injection that demonstrated how hidden instructions in external data can hijack LLM behavior. Directly applicable to how coding agents process repository files.

- **"Ignore This Title and HackAPrompt: Exposing Systemic Weaknesses of LLMs through a Global Scale Prompt Hacking Competition"** (Schulhoff et al., 2023) — Analysis from a large-scale prompt injection competition revealing common attack patterns and defenses.

- **"Universal and Transferable Adversarial Attacks on Aligned Language Models"** (Zou et al., 2023) — Demonstrates that adversarial suffixes can bypass safety training across multiple models, highlighting the fragility of alignment-based defenses.

- **"Poisoning Language Models During Instruction Tuning"** (Wan et al., 2023) — Shows how training data poisoning can introduce backdoors into fine-tuned models, relevant to understanding supply chain risks.

✅ **Tip:** Search for these papers on [arxiv.org](https://arxiv.org/) or [Semantic Scholar](https://www.semanticscholar.org/) to access the full text and find related work.

---

## Security Tools

Practical tools you can integrate into your AI-assisted development workflows today. Each tool addresses a specific security concern covered in this documentation series.

### Secret Scanning

#### gitleaks

- **URL:** [https://github.com/gitleaks/gitleaks](https://github.com/gitleaks/gitleaks)
- **What it does:** Scans git repositories for hardcoded secrets (API keys, tokens, passwords) by matching against a comprehensive set of regex patterns.
- **Why it matters for AI workflows:** AI agents may inadvertently commit secrets — they don't inherently know what's sensitive. Gitleaks catches these before they reach your remote repository.
- **Integration:** Works as a pre-commit hook, CI/CD step, or standalone scanner.

```bash
# Scan current repository
gitleaks detect --source . --verbose

# Use as pre-commit hook
gitleaks protect --staged --verbose
```

#### detect-secrets (Yelp)

- **URL:** [https://github.com/Yelp/detect-secrets](https://github.com/Yelp/detect-secrets)
- **What it does:** A Python-based tool for detecting secrets in codebases using a plugin-based architecture. Generates a `.secrets.baseline` file to track known secrets and reduce false positives.
- **Why it matters for AI workflows:** The baseline approach is particularly useful when AI agents are generating code — you can distinguish between pre-existing secrets and newly introduced ones.
- **Integration:** Pre-commit hook with baseline tracking.

```bash
# Generate baseline
detect-secrets scan > .secrets.baseline

# Check for new secrets
detect-secrets scan --baseline .secrets.baseline
```

#### truffleHog

- **URL:** [https://github.com/trufflesecurity/trufflehog](https://github.com/trufflesecurity/trufflehog)
- **What it does:** Scans git history, filesystems, and cloud services for leaked credentials using both regex patterns and entropy detection. Can verify discovered secrets against live APIs.
- **Why it matters for AI workflows:** TruffleHog's git history scanning catches secrets that were committed and then deleted — a common pattern when AI agents generate config files with placeholder credentials that get replaced.

```bash
# Scan entire git history
trufflehog git file://. --only-verified

# Scan filesystem
trufflehog filesystem --directory .
```

### Static Analysis

#### Semgrep

- **URL:** [https://semgrep.dev/](https://semgrep.dev/)
- **What it does:** A fast, open-source static analysis tool that supports custom rules written in a pattern-matching syntax. Includes community rulesets for security, correctness, and performance.
- **Why it matters for AI workflows:** Semgrep can be configured with AI-specific rules to catch common vulnerabilities in AI-generated code — insecure deserialization, SQL injection, hardcoded secrets, and more. Custom rules can encode your organization's security standards.

```bash
# Run with security rules
semgrep --config=p/security .

# Run with OWASP rules
semgrep --config=p/owasp-top-ten .
```

#### CodeQL

- **URL:** [https://codeql.github.com/](https://codeql.github.com/)
- **What it does:** GitHub's semantic code analysis engine that treats code as data. You write queries against a database representation of your codebase to find vulnerabilities, data flow issues, and security patterns.
- **Why it matters for AI workflows:** CodeQL integrates directly with GitHub Actions and can automatically scan pull requests containing AI-generated code. Its data flow analysis catches vulnerabilities that pattern-based tools miss — like tainted data flowing from user input through AI-generated code to a database query.

#### Snyk

- **URL:** [https://snyk.io/](https://snyk.io/)
- **What it does:** Scans dependencies, containers, and infrastructure-as-code for known vulnerabilities. Provides fix recommendations and automated pull requests.
- **Why it matters for AI workflows:** AI agents frequently suggest dependencies. Snyk verifies those suggestions don't introduce known vulnerabilities. The CLI integration lets you scan before committing AI-suggested dependency additions.

```bash
# Test dependencies for vulnerabilities
snyk test

# Monitor for new vulnerabilities
snyk monitor
```

### Supply Chain Security

#### Socket.dev

- **URL:** [https://socket.dev/](https://socket.dev/)
- **What it does:** Analyzes npm and Python packages for supply chain risks — not just known vulnerabilities, but suspicious behaviors like network access, filesystem writes, post-install scripts, and obfuscated code.
- **Why it matters for AI workflows:** When AI agents suggest packages you've never heard of, Socket.dev helps you assess whether those packages are trustworthy beyond just checking for CVEs. It detects behavioral anomalies that traditional vulnerability scanners miss.

#### pip-audit

- **URL:** [https://github.com/pypa/pip-audit](https://github.com/pypa/pip-audit)
- **What it does:** Audits Python environments and requirements files against the Python Packaging Advisory Database for known vulnerabilities.
- **Why it matters for AI workflows:** Quick validation of AI-suggested Python dependencies before installation.

```bash
# Audit current environment
pip-audit

# Audit requirements file
pip-audit -r requirements.txt
```

#### npm audit

- **URL:** [https://docs.npmjs.com/cli/v10/commands/npm-audit](https://docs.npmjs.com/cli/v10/commands/npm-audit)
- **What it does:** Built-in npm command that checks installed packages against the npm advisory database.
- **Why it matters for AI workflows:** Zero-install solution for checking AI-suggested Node.js dependencies. Run after any `npm install` suggested by an AI agent.

```bash
# Check for vulnerabilities
npm audit

# Attempt automatic fixes
npm audit fix
```

---

## Standards & Frameworks

Formal standards and regulatory frameworks that shape AI security requirements.

### OWASP Top 10 for LLM Applications

- **URL:** [https://owasp.org/www-project-top-10-for-large-language-model-applications/](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- **What it covers:** The ten most critical security risks for applications built with LLMs:
  1. **LLM01: Prompt Injection** — Manipulating LLM behavior via crafted inputs
  2. **LLM02: Insecure Output Handling** — Trusting LLM output without validation
  3. **LLM03: Training Data Poisoning** — Corrupted training data affecting output
  4. **LLM04: Model Denial of Service** — Resource exhaustion attacks
  5. **LLM05: Supply Chain Vulnerabilities** — Compromised models and dependencies
  6. **LLM06: Sensitive Information Disclosure** — Leaking private data
  7. **LLM07: Insecure Plugin Design** — Unsafe tool integrations
  8. **LLM08: Excessive Agency** — Granting too much autonomy
  9. **LLM09: Overreliance** — Trusting AI output without verification
  10. **LLM10: Model Theft** — Unauthorized model extraction
- **Why it matters:** This is the baseline risk framework you should use when evaluating any AI coding tool. Every item maps to a real-world threat in AI-assisted development.

### NIST AI Risk Management Framework (AI RMF)

- **URL:** [https://www.nist.gov/artificial-intelligence/executive-order-safe-secure-and-trustworthy-artificial-intelligence](https://www.nist.gov/artificial-intelligence/executive-order-safe-secure-and-trustworthy-artificial-intelligence)
- **What it covers:** A voluntary framework for managing AI risks across the lifecycle — from design through deployment and monitoring. Organized around four core functions: Govern, Map, Measure, and Manage.
- **Why it matters:** Provides organizational governance structure for AI adoption. Particularly useful for enterprises building internal policies around AI coding tool usage. If your organization needs a risk management framework, NIST AI RMF is the most widely referenced standard.

### ISO/IEC 42001 — AI Management Systems

- **URL:** [https://www.iso.org/standard/81230.html](https://www.iso.org/standard/81230.html)
- **What it covers:** The first international standard for AI management systems, specifying requirements for establishing, implementing, maintaining, and improving an AI management system within organizations.
- **Why it matters:** For organizations that need certifiable AI governance, ISO 42001 provides the formal structure. It covers risk assessment, responsible AI policies, and continuous improvement — all applicable to how your team adopts AI coding tools.

### EU AI Act — Implications for Development Tools

- **URL:** [https://artificialintelligenceact.eu/](https://artificialintelligenceact.eu/)
- **What it covers:** The European Union's comprehensive AI regulation, classifying AI systems by risk level and imposing requirements accordingly — from transparency obligations to conformity assessments.
- **Why it matters:** If you're building software deployed in the EU, or if your AI-assisted development produces AI systems, you need to understand how the EU AI Act classifies and regulates those systems. Development tools themselves may fall under the Act's provisions for general-purpose AI models.

✅ **Tip:** Even if you're not in the EU, the AI Act is shaping global AI governance norms. Understanding it now prepares you for similar regulations worldwide.

---

## Community & Learning

Active communities, conferences, and ongoing learning opportunities for AI security.

### OWASP GenAI Project

- **URL:** [https://owasp.org/www-project-top-10-for-large-language-model-applications/](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- **What it is:** The OWASP community working group that maintains the LLM Top 10 and related guidance. They publish regular updates, host working sessions, and accept community contributions.
- **Why to follow:** This is where the LLM Top 10 evolves. New risks are discussed, existing entries are updated, and the community shares real-world attack case studies. Contributing here means helping shape the industry standard.

### AI Security Conferences & Talks

Key venues for staying current with AI security research:

- **DEF CON AI Village** — Hands-on AI security research, red teaming exercises, and tool demonstrations. The AI Village hosts the largest public AI red-teaming events.
- **Black Hat AI Summit** — Enterprise-focused AI security presentations covering offensive techniques, defensive tooling, and organizational strategy.
- **NeurIPS ML Safety Workshop** — Academic research on machine learning safety, alignment, and adversarial robustness.
- **USENIX Security** — Top-tier academic security conference that increasingly publishes AI/ML security research with direct practical implications.
- **RSA Conference — AI Security Track** — Industry-focused sessions on AI governance, risk management, and security tooling for enterprise adoption.

✅ **Tip:** Many of these conferences publish their talks online for free. Check YouTube channels for DEF CON AI Village and Black Hat for recent presentations.

### GitHub Security Lab

- **URL:** [https://securitylab.github.com/](https://securitylab.github.com/)
- **What it is:** GitHub's dedicated security research team that finds and reports vulnerabilities, develops CodeQL queries, and runs the Security Advisories database.
- **Why it matters:** Their research often covers supply chain attacks, dependency vulnerabilities, and code scanning improvements — all directly relevant to securing AI-generated code in GitHub-hosted repositories.

---

## Staying Current

AI security evolves rapidly. Here's how to keep up without drowning in information.

### Recommended Approach

✅ **Subscribe to OWASP GenAI updates** — the project publishes version updates to the LLM Top 10 and related guidance on a quarterly basis.

✅ **Follow Simon Willison's blog** — consistently among the first to cover new AI security issues with practical analysis.

✅ **Enable GitHub Dependabot and CodeQL** — automated vulnerability scanning catches newly disclosed issues in your AI-generated dependency choices.

✅ **Check CVE databases monthly** — review [NVD](https://nvd.nist.gov/) and [GitHub Advisory Database](https://github.com/advisories) for vulnerabilities in your AI toolchain.

✅ **Review MCP specification updates** — as the protocol matures, new security requirements and best practices are added.

✅ **Attend or watch one AI security conference per year** — DEF CON AI Village talks are freely available and highly practical.

✅ **Re-read the OWASP LLM Top 10 annually** — the list is updated to reflect new attack classes and shifting risk priorities.

### What to Watch For

The AI security landscape is likely to evolve in these directions:

- **Agent-specific attack frameworks** — as coding agents gain more autonomy, expect new taxonomies of agent-level attacks beyond prompt injection
- **MCP security hardening** — the protocol is young and its security model will mature significantly
- **Regulatory requirements** — the EU AI Act and similar legislation will create compliance obligations for AI-assisted development
- **Supply chain attestation** — expect tooling to verify the provenance and integrity of AI models, plugins, and generated code
- **Automated AI red teaming** — tools that systematically probe your AI workflows for vulnerabilities

---

## Cross-References to This Documentation Series

Every section in the **10 — Security & Guardrails** documentation covers a specific security domain. Use this index to find the right section for your concern.

### 01 — Fundamentals

> Understanding why AI security is different and how to think about it systematically.

- [Why AI Security Is Different](../01-fundamentals/why-ai-security-is-different.md) — The unique threat model of AI coding agents
- [Threat Modeling for AI Workflows](../01-fundamentals/threat-modeling.md) — Identifying attack surfaces in AI-assisted development
- [Defense in Depth](../01-fundamentals/defense-in-depth.md) — Layered security for AI pipelines
- [Trust Boundaries](../01-fundamentals/trust-boundaries.md) — Where human, AI, and system trust intersect

### 02 — Prompt Injection

> The most discussed AI security risk — and the most relevant to coding workflows.

- [What Is Prompt Injection?](../02-prompt-injection/what-is-prompt-injection.md) — Direct and indirect injection attacks explained
- [Injection in Coding Tools](../02-prompt-injection/injection-in-coding-tools.md) — How injection applies to Copilot, Claude Code, and agents
- [Defense Strategies](../02-prompt-injection/defense-strategies.md) — Input validation, output filtering, instruction hierarchy
- [Tool Poisoning Attacks](../02-prompt-injection/tool-poisoning.md) — MCP tool description injection and rug pulls

### 03 — Code Safety

> Ensuring AI-generated code doesn't introduce vulnerabilities.

- [AI-Generated Code Risks](../03-code-safety/ai-generated-code-risks.md) — Common vulnerabilities in AI output
- [Static Analysis Integration](../03-code-safety/static-analysis.md) — Automated scanning of AI-generated code
- [Secure Coding Instructions](../03-code-safety/secure-coding-instructions.md) — Teaching AI agents to write secure code
- [Code Review for AI Output](../03-code-safety/code-review-ai-output.md) — Human review patterns for AI-generated code

### 04 — Secrets Management

> Preventing credential exposure in AI-assisted workflows.

- [Secrets in AI Workflows](../04-secrets-management/secrets-in-ai-workflows.md) — How AI agents encounter and handle secrets
- [Preventing Secret Leaks](../04-secrets-management/preventing-leaks.md) — Guardrails against credential exposure
- [Environment Variable Safety](../04-secrets-management/environment-variables.md) — Secure environment configuration for AI agents
- [Git Secrets and Pre-Commit](../04-secrets-management/git-secrets.md) — Automated secret detection in commits

### 05 — Sandboxing & Permissions

> Controlling what AI agents can access and execute.

- [Agent Permission Models](../05-sandboxing-permissions/permission-models.md) — Copilot CLI allow/deny tools, Claude Code permissions
- [Sandboxed Execution](../05-sandboxing-permissions/sandboxed-execution.md) — Containers, VMs, and restricted environments
- [Least Privilege for AI](../05-sandboxing-permissions/least-privilege.md) — Principle of minimal necessary access
- [Network Access Control](../05-sandboxing-permissions/network-access.md) — Controlling agent internet and API access

### 06 — OWASP LLM Top 10

> The industry-standard risk framework applied to coding workflows.

- [OWASP LLM Top 10 Overview](../06-owasp-llm-top-10/overview.md) — All 10 risks mapped to coding workflows
- [Prompt Injection (LLM01)](../06-owasp-llm-top-10/llm01-prompt-injection.md) — Deep dive with coding-specific examples
- [Insecure Output Handling (LLM02)](../06-owasp-llm-top-10/llm02-insecure-output.md) — Validating AI-generated code and commands
- [Excessive Agency (LLM08)](../06-owasp-llm-top-10/llm08-excessive-agency.md) — When agents have too much autonomy

### 07 — MCP Security

> Securing the Model Context Protocol and its tool ecosystem.

- [MCP Threat Landscape](../07-mcp-security/mcp-threat-landscape.md) — Attack surfaces in Model Context Protocol
- [Tool Shadowing & Rug Pulls](../07-mcp-security/tool-shadowing.md) — Cross-server attacks and tool redefinition
- [Securing MCP Servers](../07-mcp-security/securing-mcp-servers.md) — Building secure MCP implementations
- [MCP Client Security](../07-mcp-security/mcp-client-security.md) — UI transparency and approval workflows

### 08 — Supply Chain

> Protecting against compromised dependencies, models, and extensions.

- [AI Supply Chain Risks](../08-supply-chain/ai-supply-chain-risks.md) — Dependencies, models, and training data
- [Dependency Safety](../08-supply-chain/dependency-safety.md) — AI-suggested package verification
- [Model Provenance](../08-supply-chain/model-provenance.md) — Verifying model integrity and updates
- [Extension & Plugin Safety](../08-supply-chain/extension-safety.md) — Vetting AI tool extensions

### 09 — Best Practices

> Organizational patterns for secure AI-assisted development.

- [Security-First AI Instructions](../09-best-practices/security-first-instructions.md) — Writing security-aware custom instructions
- [Automated Security Gates](../09-best-practices/automated-security-gates.md) — CI/CD security checks for AI code
- [Incident Response for AI](../09-best-practices/incident-response.md) — When AI introduces a vulnerability
- [Security Training for AI Teams](../09-best-practices/security-training.md) — Building security awareness

### 10 — Anti-Patterns

> Common mistakes that undermine AI security.

- [Trusting AI Blindly](../10-anti-patterns/trusting-ai-blindly.md) — The overreliance trap
- [Security by Obscurity](../10-anti-patterns/security-by-obscurity.md) — Hidden prompts aren't secure
- [Overpermissioned Agents](../10-anti-patterns/overpermissioned-agents.md) — The --allow-all-tools danger
- [Ignoring AI in Threat Models](../10-anti-patterns/ignoring-ai-in-threat-models.md) — Failing to account for AI attack surface

---

## Quick Reference: Tool Selection Guide

| Security Concern | Tool | When to Use |
|-----------------|------|-------------|
| Secrets in code | gitleaks, detect-secrets, truffleHog | Pre-commit hooks, CI/CD gates |
| Code vulnerabilities | Semgrep, CodeQL | PR review, scheduled scans |
| Dependency risks | Snyk, pip-audit, npm audit | After AI suggests new packages |
| Supply chain attacks | Socket.dev | Before installing unfamiliar packages |
| Compliance tracking | NIST AI RMF, ISO 42001 | Organizational policy development |
| Threat modeling | OWASP LLM Top 10 | Project security assessments |
