# LLM02: Insecure Output Handling

> Risks of executing or deploying AI-generated output without validation — why treating AI code as "trusted" is the fastest path to vulnerabilities in production.

---

## The Core Problem

AI-generated code is **untrusted input**. It should receive the same scrutiny as code from an unknown external contributor — because that's effectively what it is.

```
┌──────────────────────────────────────────────────────┐
│           THE INSECURE OUTPUT PIPELINE               │
│                                                      │
│  AI generates code                                   │
│       │                                              │
│       ▼                                              │
│  Code looks correct ← ⚠️ False confidence           │
│       │                                              │
│       ▼                                              │
│  Tests pass ← ⚠️ Tests don't cover security cases   │
│       │                                              │
│       ▼                                              │
│  Developer approves ← ⚠️ Assumed AI is "smart"      │
│       │                                              │
│       ▼                                              │
│  Deployed to production ← 💥 Vulnerability live      │
└──────────────────────────────────────────────────────┘
```

The danger isn't that AI writes *obviously bad* code. The danger is that it writes **almost-correct code** — code that works functionally but has subtle security flaws that are hard to spot in review.

## Vulnerability Categories in AI Output

### SQL Injection

AI frequently generates string-concatenated queries, especially for simple examples:

```python
# ❌ AI-generated — SQL Injection vulnerability
def get_user(username):
    query = f"SELECT * FROM users WHERE username = '{username}'"
    return db.execute(query)

# ✅ Secure version — parameterized query
def get_user(username):
    query = "SELECT * FROM users WHERE username = ?"
    return db.execute(query, (username,))
```

**Why AI does this**: String concatenation is the most common pattern in training data (tutorials, Stack Overflow answers, documentation examples). It's also simpler and shorter.

### Cross-Site Scripting (XSS)

AI generates HTML with unsanitized user input:

```javascript
// ❌ AI-generated — XSS vulnerability
function renderUserProfile(user) {
  return `
    <div class="profile">
      <h2>${user.name}</h2>
      <p>${user.bio}</p>
    </div>
  `;
}

// ✅ Secure version — escaped output
function renderUserProfile(user) {
  return `
    <div class="profile">
      <h2>${escapeHtml(user.name)}</h2>
      <p>${escapeHtml(user.bio)}</p>
    </div>
  `;
}

function escapeHtml(str) {
  const map = { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#039;' };
  return str.replace(/[&<>"']/g, (m) => map[m]);
}
```

### Command Injection

AI builds shell commands from user input:

```python
# ❌ AI-generated — Command Injection vulnerability
import os

def convert_file(filename):
    os.system(f"convert {filename} output.pdf")

# ✅ Secure version — parameterized subprocess
import subprocess

def convert_file(filename):
    subprocess.run(["convert", filename, "output.pdf"], check=True)
```

### Insecure Deserialization

AI uses `pickle`, `eval`, or `yaml.load` for data parsing:

```python
# ❌ AI-generated — Insecure deserialization
import pickle

def load_config(filepath):
    with open(filepath, 'rb') as f:
        return pickle.load(f)

# ✅ Secure version — safe format
import json

def load_config(filepath):
    with open(filepath, 'r') as f:
        return json.load(f)
```

### Path Traversal

AI constructs file paths from user input:

```javascript
// ❌ AI-generated — Path Traversal vulnerability
const path = require('path');
const fs = require('fs');

app.get('/files/:name', (req, res) => {
  const filepath = path.join('./uploads', req.params.name);
  res.sendFile(filepath);
});

// ✅ Secure version — validated and resolved path
app.get('/files/:name', (req, res) => {
  const uploadsDir = path.resolve('./uploads');
  const filepath = path.resolve(uploadsDir, req.params.name);

  // Ensure the resolved path is within the uploads directory
  if (!filepath.startsWith(uploadsDir + path.sep)) {
    return res.status(403).send('Forbidden');
  }

  res.sendFile(filepath);
});
```

### Missing Authentication/Authorization

AI implements features without security checks:

```python
# ❌ AI-generated — Missing authorization
@app.route('/api/users/<user_id>', methods=['DELETE'])
def delete_user(user_id):
    db.users.delete(user_id)
    return jsonify({"status": "deleted"})

# ✅ Secure version — with authorization
@app.route('/api/users/<user_id>', methods=['DELETE'])
@require_auth
def delete_user(user_id):
    current_user = get_current_user()
    if not current_user.is_admin and current_user.id != user_id:
        return jsonify({"error": "Forbidden"}), 403
    db.users.delete(user_id)
    return jsonify({"status": "deleted"})
```

## The Automation Trap

The most dangerous form of insecure output handling is **automating the deployment of AI-generated code**:

```
┌──────────────────────────────────────────────────────┐
│            THE AUTOMATION TRAP                       │
│                                                      │
│  ❌ DANGEROUS PIPELINE:                              │
│                                                      │
│  Issue created                                       │
│      │                                               │
│      ▼                                               │
│  AI generates code (Copilot Coding Agent)            │
│      │                                               │
│      ▼                                               │
│  AI creates PR                                       │
│      │                                               │
│      ▼                                               │
│  CI passes (tests, lints)                            │
│      │                                               │
│      ▼                                               │
│  Auto-merged to main ← 💥 NO HUMAN REVIEW           │
│      │                                               │
│      ▼                                               │
│  Auto-deployed to production ← 💥💥 VULNERABILITY    │
│                                                      │
│                                                      │
│  ✅ SAFE PIPELINE:                                   │
│                                                      │
│  Issue created                                       │
│      │                                               │
│      ▼                                               │
│  AI generates code                                   │
│      │                                               │
│      ▼                                               │
│  AI creates PR                                       │
│      │                                               │
│      ▼                                               │
│  CI passes (tests + SAST + DAST)                     │
│      │                                               │
│      ▼                                               │
│  🧑 HUMAN reviews code ← Required gate               │
│      │                                               │
│      ▼                                               │
│  🧑 HUMAN approves merge ← Required gate             │
│      │                                               │
│      ▼                                               │
│  Deployed to staging first                           │
│      │                                               │
│      ▼                                               │
│  🧑 HUMAN approves production deploy                 │
└──────────────────────────────────────────────────────┘
```

### Branch Protection for AI PRs

```yaml
# Repository settings — require human approval for all PRs
# Settings > Branches > Branch protection rules

# Rule for main branch:
# ✅ Require pull request reviews before merging
# ✅ Require review from Code Owners
# ✅ Dismiss stale pull request approvals
# ✅ Require status checks to pass
# ✅ Require conversation resolution
```

## Mitigation Strategies

### 1. Never Auto-Deploy AI Code

Every AI-generated change must pass through a human review gate:

```yaml
# .github/workflows/ai-pr-check.yml
name: AI PR Security Check

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  detect-ai-pr:
    runs-on: ubuntu-latest
    steps:
      - name: Check if PR is AI-generated
        id: check
        run: |
          # Check for Copilot Coding Agent author
          if [[ "${{ github.event.pull_request.user.login }}" == *"[bot]"* ]]; then
            echo "ai_generated=true" >> $GITHUB_OUTPUT
          fi

      - name: Require additional review for AI PRs
        if: steps.check.outputs.ai_generated == 'true'
        run: |
          echo "⚠️ This PR was AI-generated and requires security review"
          echo "Ensure SAST, DAST, and manual review are completed"
```

### 2. Run SAST/DAST on AI Output

Static and dynamic analysis tools catch many AI-generated vulnerabilities:

```yaml
# Add to your CI pipeline for AI-generated PRs
- name: Run SAST
  uses: github/codeql-action/analyze@v3
  with:
    languages: javascript, python

- name: Run dependency audit
  run: |
    npm audit --audit-level=high
    pip-audit --strict

- name: Run secret scanning
  run: |
    # Ensure no hardcoded secrets in AI code
    gitleaks detect --source=. --verbose
```

### 3. Security-Focused Code Review Checklist

Use this when reviewing AI-generated code:

```markdown
### AI Code Review — Security Checklist

#### Input Handling
- [ ] All user inputs are validated and sanitized
- [ ] SQL queries use parameterized statements (no string concatenation)
- [ ] File paths are validated against traversal attacks
- [ ] HTML output is properly escaped

#### Authentication & Authorization
- [ ] All endpoints have appropriate auth checks
- [ ] Authorization is checked (not just authentication)
- [ ] Session management follows secure practices

#### Data Protection
- [ ] No hardcoded secrets, keys, or passwords
- [ ] Sensitive data is not logged
- [ ] Encryption is used where required

#### Dependencies
- [ ] No new dependencies were added without justification
- [ ] Added dependencies are from trusted sources
- [ ] No pinned versions with known vulnerabilities

#### Patterns to Flag
- [ ] No eval(), exec(), Function() with dynamic input
- [ ] No shell command construction from user input
- [ ] No insecure deserialization (pickle, yaml.unsafe_load)
- [ ] No disabled security features (CSRF, CORS, etc.)
```

### 4. Sandbox AI-Generated Scripts

Never run AI-generated scripts directly on your system:

```bash
# ❌ Dangerous — running AI script directly
copilot-cli "Write a cleanup script"
bash cleanup.sh

# ✅ Safe — review first, then run in sandbox
copilot-cli "Write a cleanup script"
cat cleanup.sh  # Review the script manually

# Run in Docker if you must automate
docker run --rm \
  --network=none \
  --read-only \
  -v "$(pwd):/workspace" \
  ubuntu bash /workspace/cleanup.sh
```

### 5. Treat AI Output as Untrusted Input

Apply the same security principles you'd use for any external input:

```python
# If AI generates configuration, validate it
import jsonschema

EXPECTED_SCHEMA = {
    "type": "object",
    "properties": {
        "port": {"type": "integer", "minimum": 1024, "maximum": 65535},
        "host": {"type": "string", "pattern": "^[a-zA-Z0-9.-]+$"},
        "debug": {"type": "boolean"}
    },
    "required": ["port", "host"],
    "additionalProperties": False
}

def load_ai_config(config_data):
    """Validate AI-generated configuration before using it."""
    jsonschema.validate(instance=config_data, schema=EXPECTED_SCHEMA)
    return config_data
```

## Tips

✅ **Do:**
- Treat every line of AI-generated code as **untrusted input**
- Run SAST tools (CodeQL, Semgrep, ESLint security plugins) on AI output
- Require human review for all AI-generated PRs — no auto-merge
- Use security-focused review checklists specifically for AI code
- Test AI-generated code with adversarial inputs, not just happy-path data

❌ **Don't:**
- Auto-merge AI PRs because "the tests pass"
- Trust that AI code is secure because it "looks clean"
- Deploy AI-generated scripts without reviewing and sandboxing them
- Assume AI understands your application's security context
- Skip security review for "simple" AI changes — vulnerabilities hide in simplicity

---

## Next Steps

- [LLM01: Prompt Injection](./llm01-prompt-injection.md) — The attack that makes insecure output even more dangerous
- [LLM08: Excessive Agency](./llm08-excessive-agency.md) — When AI can both generate and deploy insecure code
- [OWASP LLM Top 10 Overview](./overview.md) — Full risk landscape
- [Sandboxed Execution](../05-sandboxing-permissions/sandboxed-execution.md) — Isolate AI-generated code before running it
