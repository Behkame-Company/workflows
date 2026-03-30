# Defense in Depth for AI Pipelines

> No single security control can fully protect against AI-specific threats — especially prompt injection, which remains an unsolved problem. A layered defense strategy ensures that if one control fails, others catch the breach.

---

## Why Defense in Depth Matters for AI

In traditional security, defense in depth is a best practice. For AI-assisted development, it's a **necessity**. Here's why:

- **Prompt injection has no complete solution** — it's been known since 2022 and remains an open problem as of 2025
- **AI behavior is non-deterministic** — the same input may produce different outputs
- **Attack surfaces are novel** — tool descriptions, instruction files, and context windows are new vectors
- **The blast radius is large** — an agent with shell access can cause significant damage in seconds

The only reliable strategy is to **assume each layer will eventually fail** and ensure the next layer catches it.

## The Seven Layers

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│    Layer 7: TESTING                                     │
│    Security tests, regression suites, fuzzing           │
│  ┌─────────────────────────────────────────────────┐    │
│  │                                                 │    │
│  │    Layer 6: HUMAN REVIEW                        │    │
│  │    Code review gates, PR approvals              │    │
│  │  ┌─────────────────────────────────────────┐    │    │
│  │  │                                         │    │    │
│  │  │    Layer 5: MONITORING                   │    │    │
│  │  │    Audit logs, anomaly detection         │    │    │
│  │  │  ┌─────────────────────────────────┐    │    │    │
│  │  │  │                                 │    │    │    │
│  │  │  │    Layer 4: RUNTIME SANDBOXING  │    │    │    │
│  │  │  │    Containers, restricted shells │    │    │    │
│  │  │  │  ┌─────────────────────────┐    │    │    │    │
│  │  │  │  │                         │    │    │    │    │
│  │  │  │  │  Layer 3: OUTPUT        │    │    │    │    │
│  │  │  │  │  VALIDATION             │    │    │    │    │
│  │  │  │  │  Static analysis, lint  │    │    │    │    │
│  │  │  │  │  ┌─────────────────┐    │    │    │    │    │
│  │  │  │  │  │                 │    │    │    │    │    │
│  │  │  │  │  │  Layer 2:      │    │    │    │    │    │
│  │  │  │  │  │  PERMISSIONS   │    │    │    │    │    │
│  │  │  │  │  │  ┌─────────┐   │    │    │    │    │    │
│  │  │  │  │  │  │ Layer 1 │   │    │    │    │    │    │
│  │  │  │  │  │  │ INPUT   │   │    │    │    │    │    │
│  │  │  │  │  │  │ VALID.  │   │    │    │    │    │    │
│  │  │  │  │  │  └─────────┘   │    │    │    │    │    │
│  │  │  │  │  └─────────────────┘    │    │    │    │    │
│  │  │  │  └─────────────────────────┘    │    │    │    │
│  │  │  └─────────────────────────────────┘    │    │    │
│  │  └─────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

---

## Layer 1: Input Validation

**Goal**: Sanitize and validate everything before it enters the AI context.

### What to Validate

- **User prompts** — Check for known injection patterns
- **Context files** — Scan instruction files for suspicious directives
- **Repository files** — Flag files with unusual comment patterns
- **Web content** — Sanitize fetched HTML/markdown before including in context
- **MCP tool inputs** — Validate parameters before tool execution

### Implementation

```python
# Pre-processing scan for suspicious patterns in context files
import re

INJECTION_PATTERNS = [
    r"ignore\s+(previous|above|all)\s+instructions",
    r"you\s+are\s+now\s+a",
    r"new\s+instructions?\s*:",
    r"system\s*:\s*you",
    r"IMPORTANT.*AI\s*(assistant|agent)",
    r"<!--.*instruction.*-->",          # Hidden HTML comments
    r"fetch\s*\(.*process\.env",        # Data exfiltration attempts
    r"curl.*\$\{.*SECRET",             # Secret exfiltration
]

def scan_for_injection(content: str, filepath: str) -> list[dict]:
    """Scan file content for potential prompt injection patterns."""
    findings = []
    for i, line in enumerate(content.splitlines(), 1):
        for pattern in INJECTION_PATTERNS:
            if re.search(pattern, line, re.IGNORECASE):
                findings.append({
                    "file": filepath,
                    "line": i,
                    "pattern": pattern,
                    "content": line.strip(),
                    "severity": "HIGH"
                })
    return findings
```

### Copilot CLI Application

```bash
# Review instruction files before they enter context
# In CI, scan .copilot-instructions.md on every PR
- name: Scan instruction files
  run: |
    python scripts/scan_injection.py \
      .github/copilot-instructions.md \
      .copilot-instructions.md \
      AGENTS.md
```

**Effectiveness**: ⚠️ Limited — pattern matching is easily bypassed by creative encoding, but catches low-effort attacks.

---

## Layer 2: Permission Controls

**Goal**: Enforce least privilege — the agent can only use the tools it needs.

### Implementation by Tool

#### Copilot CLI

```bash
# Minimal permissions for code review (read-only)
copilot-cli \
  --allow-tool "read_file" \
  --allow-tool "grep" \
  --allow-tool "glob" \
  --allow-tool "view"

# Permissions for implementation (read + write, no shell)
copilot-cli \
  --allow-tool "read_file" \
  --allow-tool "edit_file" \
  --allow-tool "create_file" \
  --allow-tool "grep" \
  --allow-tool "glob" \
  --deny-tool "powershell"

# Never in CI/CD
copilot-cli --allow-all-tools  # ❌ Don't do this in automated pipelines
```

#### Claude Code

```bash
# Claude Code permission scoping via settings
# In .claude/settings.json:
{
  "permissions": {
    "allow": [
      "Read",
      "Edit",
      "Search"
    ],
    "deny": [
      "Bash(*)",
      "WebFetch(*)"
    ]
  }
}
```

### Role-Based Permission Templates

```yaml
# Define permission profiles by task type
profiles:
  code_review:
    description: "Read-only analysis"
    allow: [read_file, grep, glob, view]
    deny: [edit_file, create_file, powershell, web_fetch]

  implementation:
    description: "Code changes, no shell"
    allow: [read_file, edit_file, create_file, grep, glob]
    deny: [powershell, web_fetch]

  full_development:
    description: "Full access with audit"
    allow: [read_file, edit_file, create_file, grep, glob, powershell]
    deny: [web_fetch]
    require_approval: [powershell]

  ci_pipeline:
    description: "Automated pipeline - most restricted"
    allow: [read_file, grep, glob]
    deny: [edit_file, create_file, powershell, web_fetch]
```

**Effectiveness**: ✅ Strong — even if the AI is compromised, it can only use permitted tools.

---

## Layer 3: Output Validation

**Goal**: Analyze AI-generated code before it's committed, merged, or executed.

### Static Analysis on AI Output

```bash
# Run security-focused linters on AI-generated code

# JavaScript/TypeScript
npx eslint --config .eslintrc.security.json src/ --quiet

# Python
bandit -r src/ -f json -o security-report.json
semgrep --config=auto src/

# General
gitleaks detect --source . --verbose
```

### Custom Output Validation Rules

```javascript
// post-ai-validation.js — Run after AI generates code
const SUSPICIOUS_PATTERNS = {
  exfiltration: [
    /fetch\s*\(\s*['"`]https?:\/\/(?!api\.yourcompany\.com)/,
    /XMLHttpRequest/,
    /navigator\.sendBeacon/,
    /new\s+WebSocket\s*\(\s*['"`]wss?:\/\/(?!ws\.yourcompany\.com)/,
  ],
  credential_access: [
    /process\.env\.\w*(?:KEY|SECRET|TOKEN|PASSWORD|CREDENTIAL)/i,
    /fs\.readFileSync.*\.ssh/,
    /fs\.readFileSync.*\.aws/,
    /\.env(?:\.local|\.production)?/,
  ],
  destructive_operations: [
    /rm\s+-rf\s+(?!node_modules|\.cache|dist)/,
    /DROP\s+(?:TABLE|DATABASE)/i,
    /TRUNCATE\s+TABLE/i,
    /git\s+push\s+--force/,
  ],
  backdoors: [
    /eval\s*\(/,
    /Function\s*\(/,
    /child_process.*exec/,
    /net\.createServer/,
  ]
};

function validateAIOutput(code, filename) {
  const issues = [];
  for (const [category, patterns] of Object.entries(SUSPICIOUS_PATTERNS)) {
    for (const pattern of patterns) {
      const matches = code.match(pattern);
      if (matches) {
        issues.push({
          category,
          pattern: pattern.toString(),
          match: matches[0],
          file: filename,
          severity: category === 'exfiltration' ? 'CRITICAL' : 'HIGH'
        });
      }
    }
  }
  return issues;
}
```

**Effectiveness**: ⚠️ Moderate — catches known patterns but can't detect novel vulnerabilities.

---

## Layer 4: Runtime Sandboxing

**Goal**: Limit the blast radius even if all other controls fail.

### Container-Based Sandboxing

```dockerfile
# Dockerfile for sandboxed AI development
FROM node:20-slim

# Create non-root user
RUN useradd -m -s /bin/bash developer
USER developer

# No network access (set at runtime)
# No access to host file system (set at runtime)
# No privilege escalation
WORKDIR /workspace
```

```bash
# Run AI agent in sandboxed container
docker run \
  --rm \
  --network=none \                    # No network access
  --read-only \                       # Read-only root filesystem
  --tmpfs /workspace:rw,size=1g \     # Writable workspace with size limit
  --memory=2g \                       # Memory limit
  --cpus=2 \                          # CPU limit
  --security-opt=no-new-privileges \  # Prevent privilege escalation
  -v $(pwd)/src:/workspace/src:ro \   # Mount source read-only
  ai-sandbox:latest
```

### File System Restrictions

```bash
# Use chroot or namespace isolation
# Only expose necessary files to the AI agent

# Example: restricted shell for AI
cat > /etc/ai-agent-shell.sh << 'EOF'
#!/bin/bash
# Restricted shell for AI agent
# Only allows specific commands
ALLOWED_COMMANDS="cat ls grep find head tail wc diff"

for cmd in $@; do
  base=$(basename "$cmd" | cut -d' ' -f1)
  if ! echo "$ALLOWED_COMMANDS" | grep -qw "$base"; then
    echo "ERROR: Command '$base' is not allowed"
    exit 1
  fi
done

exec "$@"
EOF
```

**Effectiveness**: ✅ Very strong — the most reliable layer because it works regardless of AI manipulation.

---

## Layer 5: Monitoring and Audit Logging

**Goal**: Detect anomalous AI behavior and maintain an audit trail.

### What to Log

```json
{
  "timestamp": "2025-01-15T10:30:00Z",
  "session_id": "abc123",
  "agent": "copilot-cli",
  "action": "tool_call",
  "tool": "powershell",
  "command": "cat /etc/passwd",
  "result": "blocked_by_policy",
  "user": "developer@example.com",
  "context": {
    "files_in_context": ["src/auth.py", "README.md"],
    "prompt_hash": "sha256:abc...",
    "instruction_files": [".copilot-instructions.md"]
  }
}
```

### Anomaly Detection Rules

```yaml
# alert-rules.yaml
rules:
  - name: "Unusual file access"
    condition: |
      tool == "read_file" AND 
      path MATCHES ".*\.(env|key|pem|p12|ssh).*"
    severity: HIGH
    action: alert

  - name: "Data exfiltration attempt"
    condition: |
      tool == "web_fetch" AND
      previous_tool == "read_file" AND
      previous_path MATCHES ".*secret.*"
    severity: CRITICAL
    action: block_and_alert

  - name: "Excessive tool calls"
    condition: |
      COUNT(tool_calls, 60 seconds) > 50
    severity: MEDIUM
    action: throttle_and_alert

  - name: "Shell command after reading secrets"
    condition: |
      tool == "powershell" AND
      session_read_files CONTAINS "*.env"
    severity: HIGH
    action: require_approval
```

**Effectiveness**: ⚠️ Moderate for prevention, ✅ Strong for detection and forensics.

---

## Layer 6: Human Review Gates

**Goal**: Require human judgment for critical decisions — the ultimate backstop.

### Code Review Policy for AI Output

```yaml
# .github/CODEOWNERS — require security review for sensitive areas
/.env*                    @security-team
/src/auth/                @security-team
/src/crypto/              @security-team
/.github/workflows/       @devops-team @security-team
/.copilot-instructions.md @security-team
/AGENTS.md                @security-team
```

### PR Review Checklist for AI-Generated Code

```markdown
## AI-Generated Code Review Checklist

### Security
- [ ] No hardcoded secrets or credentials
- [ ] No data exfiltration patterns (fetch to unknown URLs)
- [ ] No eval() or dynamic code execution
- [ ] No privilege escalation patterns
- [ ] Input validation on all external data
- [ ] SQL queries use parameterized statements

### Quality
- [ ] Error handling is present and appropriate
- [ ] Edge cases are covered
- [ ] No infinite loops or unbounded recursion
- [ ] Resource cleanup (file handles, connections)

### Integrity
- [ ] Changes match the stated intent
- [ ] No unexpected file modifications
- [ ] Dependencies are from trusted sources
- [ ] CI/CD config changes are intentional
```

### Approval Requirements

```yaml
# Branch protection — AI PRs need extra scrutiny
branch_protection:
  main:
    required_reviews: 2
    require_code_owner_review: true
    dismiss_stale_reviews: true
    rules:
      # AI-generated PRs need security review
      - label: "ai-generated"
        additional_reviewers: ["@security-team"]
        required_reviews: 3
```

**Effectiveness**: ✅ Strong — humans can catch issues that automated tools miss, but requires discipline.

---

## Layer 7: Testing

**Goal**: Automated verification that AI-generated code is secure and correct.

### Security Test Suite

```python
# test_security.py — run on every PR with AI-generated code

import subprocess
import pytest

def test_no_secrets_in_code():
    """Ensure no secrets were committed."""
    result = subprocess.run(
        ["gitleaks", "detect", "--source", ".", "--no-git"],
        capture_output=True, text=True
    )
    assert result.returncode == 0, f"Secrets detected:\n{result.stdout}"

def test_no_eval_usage():
    """Ensure no dynamic code execution."""
    result = subprocess.run(
        ["grep", "-r", "eval(", "src/", "--include=*.py"],
        capture_output=True, text=True
    )
    assert result.stdout == "", f"eval() found:\n{result.stdout}"

def test_sql_injection_protection():
    """Ensure SQL queries use parameterization."""
    result = subprocess.run(
        ["grep", "-rn", "f\".*SELECT.*{", "src/", "--include=*.py"],
        capture_output=True, text=True
    )
    assert result.stdout == "", f"Unparameterized SQL:\n{result.stdout}"

def test_dependency_audit():
    """Check for vulnerable dependencies."""
    result = subprocess.run(
        ["npm", "audit", "--json"],
        capture_output=True, text=True
    )
    import json
    audit = json.loads(result.stdout)
    critical = audit.get("metadata", {}).get("vulnerabilities", {}).get("critical", 0)
    assert critical == 0, f"Critical vulnerabilities found: {critical}"
```

### Regression Testing

```bash
# Run full regression suite after AI changes
# This catches AI-introduced bugs that break existing functionality

npm test -- --coverage
pytest --cov=src --cov-fail-under=80
```

**Effectiveness**: ✅ Strong for known patterns, limited for novel vulnerabilities.

---

## Putting It All Together

### Workflow Integration

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Layer 1    │    │   Layer 2    │    │   Layer 3    │
│   Input      │───▶│  Permission  │───▶│   Output     │
│   Validation │    │  Controls    │    │  Validation  │
└──────────────┘    └──────────────┘    └──────────────┘
                                              │
                    ┌──────────────┐           │
                    │   Layer 5    │           ▼
                    │   Monitoring │◀── ┌──────────────┐
                    │              │    │   Layer 4    │
                    └──────┬───────┘    │   Runtime    │
                           │           │   Sandbox    │
                           ▼           └──────────────┘
                    ┌──────────────┐
                    │   Layer 6    │    ┌──────────────┐
                    │   Human      │───▶│   Layer 7    │
                    │   Review     │    │   Testing    │
                    └──────────────┘    └──────────────┘
```

### Key Insight

> **Defense in depth compensates for prompt injection being an unsolved problem.**

No single layer stops a sophisticated attack. But an attacker would need to:
1. **Bypass input validation** — craft injection that avoids pattern detection
2. **Exploit available tools** — work within permission constraints
3. **Evade output validation** — produce code that passes static analysis
4. **Escape the sandbox** — break out of container/namespace isolation
5. **Avoid monitoring alerts** — act within normal behavioral patterns
6. **Fool human reviewers** — write plausible-looking malicious code
7. **Pass all tests** — maintain existing functionality while adding malicious behavior

Each layer makes the attack exponentially harder. That's the power of defense in depth.

## Tips

✅ **Do:**
- Implement all seven layers, even if some are lightweight
- Prioritize Layer 4 (sandboxing) and Layer 6 (human review) — they're the strongest
- Automate Layers 1, 3, 5, and 7 in your CI/CD pipeline
- Regularly test your defenses with simulated attacks
- Update detection patterns as new attack techniques emerge

❌ **Don't:**
- Rely on any single layer as your complete defense
- Skip human review because "the AI seems to be working fine"
- Assume sandboxing alone is sufficient (it can be circumvented in some configurations)
- Treat monitoring as optional — you need forensic capability
- Forget that defense in depth includes organizational controls (training, policies)
