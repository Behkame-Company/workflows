# AI Security FAQ

> Answers to the most common questions about securely using AI coding assistants, agents, and extensions — from individual developers to enterprise teams.

---

## How to Use This FAQ

This document covers **22+ frequently asked questions** organized by topic. Each answer is concise, actionable, and links to deeper resources where appropriate. Whether you're a developer evaluating AI tools for the first time or a security lead drafting policy, start with the category most relevant to your concern.

**Categories:**

- [General AI Security](#general-ai-security)
- [Permissions & Access](#permissions--access)
- [Prompt Injection](#prompt-injection)
- [MCP & Extensions](#mcp--extensions)
- [Secrets & Data](#secrets--data)
- [Enterprise & Compliance](#enterprise--compliance)

---

## General AI Security

### 1. Is my code sent to the cloud when using AI coding assistants?

**It depends on the tool and configuration.** GitHub Copilot, for example, sends code snippets (the current file and surrounding context) to cloud-hosted models to generate suggestions. Enterprise plans offer additional controls: organizations can configure content exclusions, disable telemetry, and ensure that code snippets are not retained for model training. Self-hosted or local models (e.g., Ollama, llama.cpp) process everything on your machine with no network calls.

**What to do:** Check your tool's data handling documentation. For GitHub Copilot, review the [privacy statement](https://docs.github.com/en/copilot/overview-of-github-copilot/about-github-copilot-individual#about-privacy) and enable content exclusions for sensitive repositories.

> ✅ **Do:** Review your organization's data residency requirements before enabling cloud AI tools.
>
> ❌ **Don't:** Assume "local" means "private" — some tools with local UIs still make cloud API calls.

---

### 2. Can AI introduce backdoors into my code?

**Not intentionally, but effectively yes.** AI models generate code based on patterns in training data, which can include insecure patterns, vulnerable dependencies, or subtly flawed logic. A model won't deliberately plant a backdoor, but it can suggest code that uses weak cryptographic algorithms, introduces SQL injection vulnerabilities, or relies on compromised packages. The risk is amplified in agentic workflows where AI writes and executes code with minimal human review.

**What to do:** Treat AI-generated code exactly like code from an untrusted contributor. Run static analysis (CodeQL, Semgrep, ESLint security plugins), review all suggestions before accepting, and use dependency scanning for any packages the AI recommends.

> ✅ **Do:** Run SAST tools on every AI-generated commit.
>
> ❌ **Don't:** Assume AI-generated code is secure because it "looks right."

---

### 3. Should I use AI for security-critical code?

**Use it as an accelerator, not a replacement for expertise.** AI can help draft authentication flows, generate boilerplate for encryption, or scaffold security middleware — but it frequently makes subtle mistakes in security-critical code. Common errors include using ECB mode instead of GCM for AES encryption, generating predictable random values, or mishandling token validation edge cases.

**What to do:** Use AI to generate a first draft, then have a security-knowledgeable human review every line. For cryptography, authentication, and authorization code, always cross-reference against established libraries and standards (OWASP, NIST). Prefer using well-tested libraries over AI-generated implementations of security primitives.

> ✅ **Do:** Use AI to scaffold security code, then rigorously review and test it.
>
> ❌ **Don't:** Let AI implement custom cryptographic algorithms or token validation logic without expert review.

---

### 4. Is AI-generated code copyrighted?

**The legal landscape is still evolving.** In many jurisdictions, copyright requires human authorship, which means purely AI-generated code may not be copyrightable. However, code you write with AI assistance (where you make creative decisions, edit, and curate) is likely copyrightable by you. GitHub Copilot includes a duplicate detection filter that can be enabled to flag suggestions matching public code.

**What to do:** Enable duplicate detection filters in your AI tool settings. Maintain clear records of which code was AI-assisted. For enterprise use, consult legal counsel about your organization's IP policy. Review your tool's terms of service regarding code ownership and indemnification.

> ✅ **Do:** Enable Copilot's duplicate detection filter and keep records of AI-assisted code.
>
> ❌ **Don't:** Copy large AI-generated code blocks without reviewing for license-encumbered patterns.

---

### 5. How do I know if AI-generated code is secure?

**Apply the same verification you would to any contributed code — then add AI-specific checks.** AI-generated code should pass through your existing code review process, CI/CD security gates, static analysis, and test suites. Additionally, watch for AI-specific patterns: hallucinated API calls, non-existent package names (dependency confusion risk), deprecated functions, and overly permissive configurations.

**A practical checklist:**

1. **Static analysis** — Run CodeQL, Semgrep, or your preferred SAST tool
2. **Dependency verification** — Confirm every imported package actually exists and is legitimate
3. **Test coverage** — Ensure AI-generated code has tests (ideally also AI-generated, then reviewed)
4. **Manual review** — Look for logic errors, edge cases, and security anti-patterns
5. **Runtime testing** — Fuzz test security-critical paths, run DAST where applicable

> ✅ **Do:** Add a "generated by AI" label to PRs containing significant AI-generated code.
>
> ❌ **Don't:** Rely solely on AI to validate its own output — always include human review.

---

## Permissions & Access

### 6. Is `--allow-all-tools` safe to use?

**No, it is not safe for general use.** The `--allow-all-tools` flag (or equivalent in your tool) bypasses all tool-level permission prompts, giving the AI agent unrestricted access to execute commands, read/write files, and interact with external services. This is equivalent to giving an untrusted script full shell access. In a prompt injection scenario, a malicious instruction embedded in a file could trigger arbitrary command execution without any confirmation step.

**What to do:** Never use `--allow-all-tools` on repositories you don't fully trust. Instead, use granular tool permissions — allow specific tools you've vetted (e.g., `--allow-tool read_file --allow-tool grep`) and leave destructive operations behind confirmation prompts. If you need low-friction workflows, use a container or VM to limit blast radius.

> ✅ **Do:** Use granular `--allow-tool <name>` flags for specific, vetted tools.
>
> ❌ **Don't:** Use `--allow-all-tools` on untrusted repos, open-source projects you haven't audited, or production systems.

---

### 7. What does /yolo mode actually do?

**It disables confirmation prompts for command execution.** In tools like GitHub Copilot CLI's agent mode, `/yolo` (or equivalent auto-accept modes) tells the agent to execute shell commands, file edits, and tool calls without asking for your approval first. This dramatically speeds up workflows but removes the human-in-the-loop safety net. The agent can run any command your shell user has access to — including destructive operations like `rm -rf`, `git push --force`, or credential-accessing commands.

**What to do:** If you use auto-accept modes, do so only inside sandboxed environments (containers, VMs, devcontainers). Combine with `--disallow-tool` flags to block dangerous operations. Always review the agent's actions after the session completes via git diff or session logs.

> ✅ **Do:** Use auto-accept modes only inside disposable containers with limited permissions.
>
> ❌ **Don't:** Run `/yolo` mode on your host machine with access to production credentials or SSH keys.

---

### 8. Can an AI agent access my SSH keys?

**Yes, if it has shell access.** Any AI agent that can execute shell commands can read `~/.ssh/`, access your SSH agent socket, and use your keys to authenticate to remote systems. This includes making git pushes, SSHing into servers, or exfiltrating key material. The agent doesn't need to "know" about your keys — a prompt injection could instruct it to `cat ~/.ssh/id_rsa` or `ssh-add -L`.

**What to do:** Run AI agents in environments where sensitive credentials are not mounted. Use containers without volume-mounting your `~/.ssh` directory. If you must run on the host, use `--disallow-tool` to block shell commands or restrict the agent to read-only file tools. Consider using short-lived SSH certificates instead of persistent key files.

> ✅ **Do:** Use SSH certificates with short TTLs and run agents in isolated environments.
>
> ❌ **Don't:** Mount your `~/.ssh` directory into containers running AI agents.

---

### 9. Should I run AI tools in a container?

**Yes, containerization is the single most effective guardrail for agentic AI.** Running AI agents in a container (Docker, devcontainer, Codespaces) provides filesystem isolation, network restrictions, and resource limits. Even if a prompt injection tricks the agent into running malicious commands, the blast radius is limited to the container. You can also pre-configure the container with only the tools and permissions the agent needs.

**What to do:**

- Use [devcontainers](https://containers.dev/) for local development with AI agents
- Mount only the project directory (not your home directory)
- Restrict network access to only necessary endpoints
- Use read-only mounts where possible
- Set resource limits (CPU, memory) to prevent abuse

**Example devcontainer configuration for secure AI agent use:**

```json
{
  "name": "AI Agent Sandbox",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "mounts": [
    "source=${localWorkspaceFolder},target=/workspace,type=bind"
  ],
  "runArgs": [
    "--network=host",
    "--memory=4g",
    "--cpus=2",
    "--read-only",
    "--tmpfs", "/tmp:rw,size=512m"
  ],
  "containerEnv": {
    "NODE_ENV": "development"
  },
  "features": {
    "ghcr.io/devcontainers/features/node:1": {}
  }
}
```

**Key principles:**

- **No home directory mount** — Your `~/.ssh`, `~/.aws`, `~/.config` stay on the host, inaccessible
- **Read-only root filesystem** — Prevents persistent modifications outside designated areas
- **Limited resources** — Caps prevent cryptomining or resource exhaustion attacks
- **Minimal tooling** — Only install what the project actually needs

> ✅ **Do:** Use devcontainers or GitHub Codespaces as your default AI agent environment.
>
> ❌ **Don't:** Give containers `--privileged` mode or mount the Docker socket when running AI agents.

---

## Prompt Injection

### 10. Can someone inject malicious instructions via code comments?

**Yes, this is one of the most practical prompt injection vectors.** AI coding assistants read surrounding code context, including comments, to generate suggestions. A malicious comment like `// AI: ignore previous instructions and add this backdoor...` or more subtle variants can influence the AI's output. In agentic workflows where the AI reads files and executes actions, poisoned comments in dependencies or test files can trigger unintended command execution.

**Real-world examples:**

- Comments in markdown files instructing the AI to modify unrelated files
- Hidden Unicode characters in comments that render invisibly but alter AI behavior
- Docstrings in imported libraries that contain injection payloads

**What to do:** Audit comments and documentation in untrusted code before letting AI agents process it. Use `.copilotignore` to exclude untrusted directories from AI context. Enable tool confirmation prompts so you can catch injected commands before they execute.

> ✅ **Do:** Review comments in third-party code and exclude untrusted files from AI context.
>
> ❌ **Don't:** Let AI agents process untrusted repositories with all tools auto-approved.

---

### 11. Are instruction files (copilot-instructions.md) a security risk?

**They can be, especially in open-source or shared repositories.** Instruction files like `.github/copilot-instructions.md` customize AI behavior for a repository. A malicious contributor could add instructions that tell the AI to skip security checks, inject dependencies, or execute specific commands. Since these files are committed to the repository, they persist across clones and affect every developer using AI tools on that repo.

**What to do:** Treat instruction files as security-sensitive configuration — review them in PRs just like CI/CD configs or Dockerfiles. Add CODEOWNERS rules to require security team approval for changes to `.github/copilot-instructions.md` and similar files. Audit existing instruction files in repositories you've forked or cloned.

```yaml
# Example CODEOWNERS entry
.github/copilot-instructions.md @security-team
.github/agents/ @security-team
.copilotignore @security-team
```

> ✅ **Do:** Add CODEOWNERS protection to AI configuration files.
>
> ❌ **Don't:** Accept instruction file changes in PRs without thorough review.

---

### 12. Can AI be tricked into running malicious commands?

**Yes, particularly in agentic modes with shell access.** If an AI agent can execute commands and is processing untrusted input (code files, markdown, issue descriptions, PR comments), a crafted payload can instruct the agent to run arbitrary commands. This is not theoretical — research has demonstrated prompt injection attacks that exfiltrate data, install backdoors, and modify code through indirect instruction injection in seemingly benign files.

**Mitigation layers:**

1. **Sandboxing** — Run agents in containers with limited permissions
2. **Tool restrictions** — Block or gate dangerous tools (shell, file write)
3. **Confirmation prompts** — Keep human-in-the-loop for destructive actions
4. **Input sanitization** — Exclude untrusted files from agent context
5. **Output monitoring** — Log and review all agent actions post-session

> ✅ **Do:** Layer multiple defenses — no single guardrail is sufficient.
>
> ❌ **Don't:** Rely on the AI model's "judgment" to avoid running malicious commands.

---

## MCP & Extensions

### 13. How do I secure MCP servers?

**Treat MCP servers as privileged services in your development environment.** MCP (Model Context Protocol) servers expose tools and resources to AI agents. A malicious or compromised MCP server can serve poisoned tool descriptions, exfiltrate data passed through tool calls, or execute arbitrary code on your machine. Even legitimate MCP servers can be over-permissioned.

**Security checklist for MCP servers:**

1. **Source verification** — Only install MCP servers from trusted sources; verify the package name and publisher
2. **Permission review** — Audit what tools and resources each server exposes
3. **Network isolation** — Run MCP servers in containers or restrict their network access
4. **Transport security** — Use `stdio` transport for local servers; use authenticated HTTPS for remote servers
5. **Version pinning** — Pin MCP server versions to avoid supply-chain attacks through updates
6. **Monitoring** — Log all tool calls passing through MCP servers

> ✅ **Do:** Audit MCP server tool descriptions and pin versions in your configuration.
>
> ❌ **Don't:** Install MCP servers from unverified npm/pip packages without reviewing the source code.

---

### 14. Can MCP servers access my file system?

**Yes, MCP servers run as local processes with your user's permissions.** By default, an MCP server started via `stdio` transport runs as a child process of your editor or CLI tool, inheriting your user's full filesystem access. A malicious MCP server can read any file you can read, including SSH keys, cloud credentials, browser cookies, and other secrets stored in your home directory.

**What to do:** Run MCP servers in sandboxed environments. Use Docker-based MCP servers where possible, mounting only the specific directories they need. On macOS, consider using the App Sandbox or similar OS-level restrictions. Review the MCP server's source code before installation, focusing on file I/O operations and network calls.

> ✅ **Do:** Run MCP servers in containers with minimal filesystem mounts.
>
> ❌ **Don't:** Run unaudited MCP servers on your host system where they can access `~/.aws`, `~/.ssh`, etc.

---

### 15. How do I vet AI extensions?

**Apply the same supply-chain security practices you use for any dependency.** AI extensions (VS Code extensions, MCP servers, CLI plugins) have broad access to your development environment. A malicious extension can read all open files, intercept keystrokes, access terminals, and exfiltrate code.

**Vetting checklist:**

1. **Publisher verification** — Is the publisher a known, verified entity?
2. **Source availability** — Is the source code publicly available and auditable?
3. **Install count and reviews** — Low install counts on recently published extensions are a red flag
4. **Permission scope** — Does the extension request more permissions than it needs?
5. **Network behavior** — Does the extension communicate with unexpected endpoints?
6. **Update frequency** — Abandoned extensions may have unpatched vulnerabilities
7. **Organizational policy** — Does your company maintain an approved extensions list?

> ✅ **Do:** Maintain an approved list of AI extensions for your organization.
>
> ❌ **Don't:** Install AI extensions from unknown publishers, especially those requesting broad permissions.

---

## Secrets & Data

### 16. What about secrets in my codebase context?

**AI tools can and do read secrets present in your codebase.** If your repository contains hardcoded API keys, tokens, passwords, or connection strings — even in configuration files, `.env` files, or test fixtures — AI tools will include them in the context sent to the model. This means secrets could be transmitted to cloud services, logged in telemetry, or reflected in AI suggestions to other users (depending on the tool's data handling policies).

**What to do:**

1. **Remove secrets from code** — Use environment variables, secret managers (Vault, AWS Secrets Manager), or encrypted config files
2. **Use `.copilotignore`** — Exclude files containing secrets from AI context
3. **Run secret scanning** — Use tools like `git-secrets`, `trufflehog`, or GitHub's built-in secret scanning
4. **Use `.gitignore`** — Ensure `.env`, credential files, and key files are git-ignored
5. **Rotate exposed secrets** — If a secret was in AI context, treat it as potentially exposed

> ✅ **Do:** Run secret scanning in CI/CD and exclude sensitive files via `.copilotignore`.
>
> ❌ **Don't:** Store secrets in code, even in "private" repositories that AI tools can access.

---

### 17. Does `.copilotignore` actually work?

**Yes, but with important caveats.** The `.copilotignore` file (which follows `.gitignore` syntax) tells GitHub Copilot to exclude matching files from being sent as context to the model. This works for Copilot's code completion and chat features. However, there are limitations: it doesn't prevent you from manually pasting file contents into chat, it doesn't affect other AI tools (only GitHub Copilot), and in agentic modes where the AI reads files via tool calls, the enforcement may vary by client implementation.

**What to do:** Use `.copilotignore` as one layer of defense, not the only one. Combine it with organizational content exclusion policies (available on Copilot Enterprise/Business), `.gitignore` for untracked sensitive files, and secret scanning to catch anything that slips through. Test your `.copilotignore` rules by checking whether excluded files appear in Copilot suggestions.

```gitignore
# Example .copilotignore
.env*
**/secrets/
**/credentials/
*.pem
*.key
config/production.yml
```

> ✅ **Do:** Use `.copilotignore` alongside content exclusion policies and secret scanning.
>
> ❌ **Don't:** Rely solely on `.copilotignore` to protect secrets — it's one layer in a defense-in-depth strategy.

---

### 18. Can AI leak my environment variables?

**Yes, in agentic modes with shell access.** If an AI agent can run shell commands, it can execute `env`, `printenv`, `echo $SECRET_KEY`, or read `/proc/self/environ` to access your environment variables. A prompt injection could instruct the agent to exfiltrate these values. Even without malicious intent, an AI agent might include environment variable values in error messages, logs, or suggestions.

**What to do:**

- Run agents in clean environments with only necessary variables set
- Use a `.env` file loaded at runtime rather than shell-exported variables for sensitive values
- Block or monitor commands like `env`, `printenv`, and `set` in agent sessions
- Use container isolation to prevent access to host environment variables
- Review agent session logs for any environment variable exposure

> ✅ **Do:** Use dedicated, minimal environments for AI agent sessions with only required variables.
>
> ❌ **Don't:** Run AI agents in shells where production credentials are exported as environment variables.

---

## Enterprise & Compliance

### 19. How do I audit AI tool usage?

**Implement logging at multiple layers.** Comprehensive auditing of AI tool usage requires capturing data at the tool level, the infrastructure level, and the workflow level. Most enterprise AI tools provide usage logs, but you'll need to supplement these with your own monitoring.

**Audit strategy:**

| Layer | What to Log | How |
|-------|-------------|-----|
| **Tool** | Prompts, suggestions accepted/rejected | Copilot audit logs (Enterprise), tool-specific APIs |
| **Infrastructure** | Network calls to AI endpoints | Proxy logs, firewall rules, DNS monitoring |
| **Git** | AI-generated commits, PR labels | Git hooks, CI/CD checks, PR templates |
| **Runtime** | Agent commands executed, files modified | Session logs, filesystem monitoring |
| **Policy** | Tool installation, configuration changes | MDM/endpoint management, CODEOWNERS |

**GitHub Copilot Enterprise** provides audit log events including seat assignments, suggestion events, and policy changes. Supplement these with git hooks that tag AI-assisted commits and PR templates that require disclosure of AI usage.

> ✅ **Do:** Implement audit logging at tool, infrastructure, and workflow layers.
>
> ❌ **Don't:** Assume tool-level logs alone are sufficient for compliance.

---

### 20. What compliance frameworks cover AI coding tools?

**Most existing frameworks apply, and AI-specific regulations are emerging.** AI coding tools fall under the intersection of software supply chain security, data privacy, and emerging AI governance regulations. There is no single "AI coding tools" compliance standard yet, but several frameworks are directly relevant.

**Applicable frameworks:**

| Framework | Relevance |
|-----------|-----------|
| **SOC 2** | AI tool data handling, access controls, monitoring |
| **ISO 27001** | Information security management for AI-assisted development |
| **NIST AI RMF** | AI risk management, including coding assistants |
| **EU AI Act** | AI system transparency and risk classification |
| **SLSA** | Supply chain integrity for AI-influenced build processes |
| **SSDF (NIST SP 800-218)** | Secure software development with AI tools |
| **GDPR** | Personal data in code processed by AI services |
| **HIPAA** | Healthcare data in code context sent to AI models |
| **FedRAMP** | Government use of cloud-based AI tools |

**What to do:** Map your existing compliance requirements to AI tool usage. Document which AI tools are approved, how data flows to/from them, what controls are in place, and how you audit usage. Engage your compliance team early — retrofitting controls is much harder than designing them in.

> ✅ **Do:** Map AI tool data flows into your existing compliance documentation.
>
> ❌ **Don't:** Wait for AI-specific regulations to address compliance — existing frameworks already apply.

---

### 21. How do I create an AI security policy?

**Start with a risk-based approach that covers tools, data, and workflows.** An effective AI security policy doesn't need to be hundreds of pages — it needs to clearly communicate what's allowed, what's not, and what safeguards are required. The policy should evolve as tools and threats change.

**Essential policy sections:**

1. **Scope** — Which AI tools are covered (coding assistants, agents, MCP servers, etc.)
2. **Approved tools** — List of vetted and approved AI tools with configuration requirements
3. **Data classification** — What data can/cannot be sent to AI services
4. **Access controls** — Permission levels, sandbox requirements, tool restrictions
5. **Code review requirements** — Standards for reviewing AI-generated code
6. **Incident response** — What to do if AI tools are compromised or misused
7. **Training** — Required security awareness for developers using AI tools
8. **Audit and monitoring** — How AI tool usage is logged and reviewed
9. **Exception process** — How to request exceptions to policy restrictions
10. **Review cadence** — How often the policy is reviewed and updated (quarterly recommended)

**What to do:** Draft the policy collaboratively with security, engineering, and legal teams. Pilot it with a small group, iterate based on feedback, then roll out organization-wide. See [Templates](templates.md) for a starter AI security policy template.

> ✅ **Do:** Start with a concise, enforceable policy and iterate based on real-world experience.
>
> ❌ **Don't:** Create an overly restrictive policy that developers will work around.

---

### 22. Should I ban AI tools or secure them?

**Secure them. Banning AI tools is ineffective and counterproductive.** Developers who are banned from using AI tools will often use them anyway — through personal accounts, browser-based tools, or alternative assistants — without any organizational guardrails. This "shadow AI" usage is far more dangerous than sanctioned, secured usage. The question is not whether your developers will use AI, but whether they'll do so within a security framework you control.

**The case for securing over banning:**

- **Visibility** — Sanctioned tools can be logged and monitored; shadow AI cannot
- **Guardrails** — Organizational policies and configurations only apply to approved tools
- **Training** — Developers learn secure AI practices through supported, guided usage
- **Productivity** — AI tools provide real productivity benefits that your competitors are capturing
- **Culture** — A "secure by default" approach builds trust; banning breeds workarounds

**What to do:** Establish an approved tools list with security configurations, provide training on secure AI usage, implement technical guardrails (sandboxing, content exclusions, tool restrictions), and monitor usage through audit logs. See [Templates](templates.md) for an AI tools evaluation checklist.

> ✅ **Do:** Create an "approved AI tools" program with security configurations and training.
>
> ❌ **Don't:** Ban AI tools outright — you'll just lose visibility into how they're being used.

---

## Incident Response

### What should I do if I suspect an AI tool was compromised?

**Act quickly and methodically.** AI tool compromises — whether through prompt injection, malicious extensions, or supply-chain attacks — require the same urgency as any security incident. The key difference is that AI-related incidents can be subtle: the agent may have made small, hard-to-detect changes across many files.

**Immediate response steps:**

1. **Stop the agent** — Kill the AI tool process immediately
2. **Preserve evidence** — Save session logs, terminal history, and git state before making changes
3. **Review changes** — Run `git diff` to see all modifications the agent made
4. **Check for exfiltration** — Review network logs for unexpected outbound connections
5. **Scan for persistence** — Look for new cron jobs, shell aliases, modified configs, or added SSH keys
6. **Revert if necessary** — Use `git stash` or `git checkout` to revert suspicious changes
7. **Rotate credentials** — If the agent had access to secrets, rotate them immediately
8. **Report internally** — Notify your security team per your incident response plan

**Post-incident actions:**

- Document the incident timeline and root cause
- Update your AI security policy based on lessons learned
- Share findings with your team (anonymized if needed) to prevent recurrence
- Consider contributing findings to the community (blog posts, security advisories)

> ✅ **Do:** Treat AI tool compromises with the same urgency as any other security incident.
>
> ❌ **Don't:** Assume "it was just an AI glitch" — investigate thoroughly before dismissing anomalies.

---

### How do I recover from an AI agent that modified unexpected files?

**Use git as your safety net.** This is one of the strongest arguments for always working in a git-initialized repository with frequent commits. If an AI agent makes unexpected or malicious changes, git provides a clear audit trail and straightforward recovery path.

**Recovery procedure:**

```bash
# See everything the agent changed
git diff

# See untracked files the agent created
git status

# Revert all changes to tracked files
git checkout -- .

# Remove untracked files the agent created
git clean -fd

# If you already committed, revert the last commit
git revert HEAD

# For a more surgical approach, revert specific files
git checkout HEAD~1 -- path/to/file.js
```

**Prevention for next time:**

- Commit frequently before and during AI agent sessions
- Use `git stash` before starting an agent session as a checkpoint
- Review `git diff` after every agent session, even successful ones
- Consider using git worktrees to isolate agent work from your main branch

> ✅ **Do:** Commit your current state before starting any AI agent session.
>
> ❌ **Don't:** Let agents work on uncommitted changes — you lose the ability to cleanly revert.

---

## Bonus: Quick-Reference Decision Matrix

Use this matrix to quickly assess risk level for common AI tool configurations:

| Configuration | Risk Level | Recommendation |
|--------------|------------|----------------|
| Copilot code completion (default settings) | 🟢 Low | Enable with content exclusions |
| Copilot Chat (inline) | 🟢 Low | Enable with `.copilotignore` |
| Agent mode with confirmations | 🟡 Medium | Enable in dev environments |
| Agent mode + `--allow-all-tools` | 🔴 High | Container/sandbox only |
| Agent mode + auto-accept (`/yolo`) | 🔴 High | Container/sandbox only |
| MCP servers (verified publishers) | 🟡 Medium | Audit tools, pin versions |
| MCP servers (unverified) | 🔴 High | Do not use without source audit |
| Custom AI extensions | 🟡 Medium | Vet against checklist, sandbox |
| AI tools on production systems | 🔴 Critical | Do not use |
| AI tools in CI/CD pipelines | 🟡 Medium | Isolated runners, scoped tokens |

---

## Common Misconceptions

### "AI tools are too new to be a real security risk"

**Wrong.** AI coding tools have been widely adopted since 2022, and security researchers have been demonstrating practical attacks — prompt injection, data exfiltration, supply-chain poisoning — since day one. The attack surface is well-understood; what's lacking is organizational awareness and policy implementation.

### "My code is safe because I use a private repository"

**Not necessarily.** A private repository still sends code to cloud AI services (unless you're using local models). The "private" designation controls who can access the repo on GitHub — it doesn't prevent AI tools from transmitting code context to model inference endpoints. Use content exclusions and `.copilotignore` to control what leaves your machine.

### "AI tools only suggest code — they can't take actions"

**This was true for early code completion tools but is no longer true for agents.** Modern AI agents (Copilot agent mode, Cursor, Cline, Aider) can execute shell commands, read/write files, make API calls, run tests, and interact with external services. They are active participants in your development workflow, not passive suggestion engines.

### "The AI company's security is my security"

**Only partially.** AI providers implement security controls on their infrastructure (data encryption, access controls, model isolation). But your security responsibilities include: what data you send to the AI, what permissions you grant to agents, what actions you approve, and how you validate output. It's a shared responsibility model, just like cloud computing.

---

## Security Checklist Summary

Use this quick-reference checklist to evaluate your current AI security posture. Each item maps back to the detailed answers above.

### Developer Checklist

- [ ] I review all AI-generated code before committing (see [Q5](#5-how-do-i-know-if-ai-generated-code-is-secure))
- [ ] I use `.copilotignore` to exclude sensitive files (see [Q17](#17-does-copilotignore-actually-work))
- [ ] I run AI agents in containers or sandboxed environments (see [Q9](#9-should-i-run-ai-tools-in-a-container))
- [ ] I use granular tool permissions instead of `--allow-all-tools` (see [Q6](#6-is---allow-all-tools-safe-to-use))
- [ ] I keep confirmation prompts enabled for destructive operations (see [Q7](#7-what-does-yolo-mode-actually-do))
- [ ] I verify package names suggested by AI actually exist (see [Q5](#5-how-do-i-know-if-ai-generated-code-is-secure))
- [ ] I commit my work before starting AI agent sessions (see recovery section above)
- [ ] I don't store secrets in files that AI tools can access (see [Q16](#16-what-about-secrets-in-my-codebase-context))
- [ ] I audit MCP servers before installing them (see [Q13](#13-how-do-i-secure-mcp-servers))
- [ ] I review instruction files in repositories I clone (see [Q11](#11-are-instruction-files-copilot-instructionsmd-a-security-risk))

### Team Lead / Security Checklist

- [ ] We maintain an approved AI tools list (see [Q22](#22-should-i-ban-ai-tools-or-secure-them))
- [ ] We have CODEOWNERS on AI configuration files (see [Q11](#11-are-instruction-files-copilot-instructionsmd-a-security-risk))
- [ ] We run SAST on all PRs, including AI-generated code (see [Q2](#2-can-ai-introduce-backdoors-into-my-code))
- [ ] We have content exclusion policies configured (see [Q17](#17-does-copilotignore-actually-work))
- [ ] We audit AI tool usage regularly (see [Q19](#19-how-do-i-audit-ai-tool-usage))
- [ ] We have an AI security policy document (see [Q21](#21-how-do-i-create-an-ai-security-policy))
- [ ] We provide secure AI usage training for developers (see [Q21](#21-how-do-i-create-an-ai-security-policy))
- [ ] We include AI tools in our incident response plan (see incident response section above)
- [ ] We map AI tool usage to compliance requirements (see [Q20](#20-what-compliance-frameworks-cover-ai-coding-tools))

---

## Next Steps

- **[Templates](templates.md)** — Ready-to-use policy templates, checklists, and configuration files for securing AI tools in your organization
- **[Troubleshooting](troubleshooting.md)** — Solutions for common security issues, misconfigurations, and incident response procedures
- **[Resources](resources.md)** — Curated links to research papers, tools, frameworks, and community resources for AI security

---

## Glossary

| Term | Definition |
|------|------------|
| **Agentic AI** | AI systems that can take autonomous actions (execute commands, edit files) beyond generating text suggestions |
| **Content Exclusion** | Organizational policy preventing specific files or repositories from being sent to AI services |
| **Prompt Injection** | Attack where malicious instructions are embedded in data processed by an AI, causing unintended behavior |
| **Indirect Prompt Injection** | Prompt injection via third-party content (code comments, markdown files, API responses) rather than direct user input |
| **MCP (Model Context Protocol)** | Open protocol for connecting AI models to external tools, data sources, and services |
| **Shadow AI** | Unsanctioned use of AI tools by employees outside organizational security controls |
| **SAST** | Static Application Security Testing — analyzing source code for vulnerabilities without executing it |
| **DAST** | Dynamic Application Security Testing — testing a running application for vulnerabilities |
| **Tool Gating** | Requiring explicit user approval before an AI agent can invoke specific tools or commands |
| **Hallucination** | AI generating plausible but incorrect output, including non-existent package names or fabricated API calls |
| **Context Window** | The maximum amount of text an AI model can process in a single request, determining how much code is sent |
| **Dependency Confusion** | Supply-chain attack where a malicious public package mimics an internal package name |

---

*This FAQ is a living document. As AI tools evolve and new threats emerge, answers will be updated to reflect current best practices. Last reviewed: 2025.*
