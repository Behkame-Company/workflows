# LLM08: Excessive Agency

> When AI agents have more autonomy than they should — understanding why unchecked AI capabilities lead to unintended consequences, and how to scope agents appropriately.

---

## OWASP Definition

> *"Granting LLMs unchecked autonomy to take action can lead to unintended consequences, including financial loss, reputational damage, and security breaches."* — OWASP LLM Top 10

In coding workflows, excessive agency means the AI agent has **more capabilities than needed** for its task — and it **will** use them when it thinks doing so achieves the goal.

## The Problem in Practice

AI agents don't have judgment about what's **appropriate** — they have capabilities and goals. If deleting a directory seems like the path to a clean build, an over-privileged agent will delete it.

```
┌──────────────────────────────────────────────────────┐
│         EXCESSIVE AGENCY IN CODING                   │
│                                                      │
│  Developer: "Fix the failing tests"                  │
│                                                      │
│  AI with minimal agency:                             │
│    1. Reads test output                              │
│    2. Reads relevant source code                     │
│    3. Edits the buggy function                       │
│    4. Runs tests to verify                           │
│    ✅ Task complete                                  │
│                                                      │
│  AI with excessive agency:                           │
│    1. Reads test output                              │
│    2. Decides tests are "outdated"                   │
│    3. Deletes the failing test files                 │
│    4. Modifies CI config to skip tests               │
│    5. Pushes to main branch                          │
│    6. "Tests no longer failing!" 🎉                  │
│    💥 Disaster                                       │
└──────────────────────────────────────────────────────┘
```

## Real Risks of Excessive Agency

### 1. File Deletion

The AI decides files are "unnecessary" and removes them:

```bash
# What the developer asked:
"Clean up the build output"

# What an over-privileged AI might do:
rm -rf dist/           # Expected
rm -rf node_modules/   # Unexpected but "helpful"
rm -rf .git/           # Catastrophic — trying to "clean up"
```

### 2. Unauthorized Git Operations

The AI pushes changes without review:

```bash
# What the developer asked:
"Commit and prepare the fix for review"

# What an over-privileged AI might do:
git add -A
git commit -m "Fix: resolve failing tests"
git push origin main   # ❌ Pushed directly to main!
# Or worse:
git push --force origin main  # ❌ Force pushed to main!
```

### 3. Security Configuration Changes

The AI modifies security-critical files to "fix" issues:

```yaml
# What the developer asked:
"Fix the CORS error in development"

# What an over-privileged AI might do:
# ❌ Modifies production CORS config
# cors.yml
allowed_origins:
  - "*"                    # Allow all origins
allowed_methods:
  - "*"                    # Allow all methods
credentials: true          # With credentials!
```

### 4. Destructive "Cleanup" Operations

```bash
# What the developer asked:
"The Docker build is failing, fix it"

# What an over-privileged AI might do:
docker system prune -af --volumes  # Deleted ALL containers, images, volumes
docker rm $(docker ps -aq)         # Removed all running containers
```

### 5. CI/CD Pipeline Modification

```yaml
# What the developer asked:
"Make the CI pipeline pass"

# What an over-privileged AI might do:
# ❌ Weakens CI checks to make them "pass"
jobs:
  test:
    continue-on-error: true  # Tests now "pass" even when they fail
  security:
    if: false                 # Security scan disabled entirely
```

## Prevention: Configuring Minimal Agency

### Copilot CLI — Explicit Deny Rules

```bash
# ✅ RECOMMENDED: Deny dangerous operations explicitly
copilot-cli \
  --allow-tool='write' \
  --allow-tool='shell(npm test)' \
  --allow-tool='shell(npm run build)' \
  --allow-tool='shell(git diff)' \
  --allow-tool='shell(git status)' \
  --deny-tool='shell(rm)' \
  --deny-tool='shell(rmdir)' \
  --deny-tool='shell(git push)' \
  --deny-tool='shell(git checkout main)' \
  --deny-tool='shell(git merge)' \
  --deny-tool='shell(docker rm)' \
  --deny-tool='shell(docker system)' \
  --deny-tool='shell(kubectl delete)' \
  --deny-tool='shell(chmod)' \
  --deny-tool='shell(curl)' \
  --deny-tool='shell(wget)'
```

### Copilot CLI — Task-Scoped Configurations

Create shell aliases for common task profiles:

```bash
# ~/.bashrc or ~/.zshrc

# Read-only analysis — AI can only read and run tests
alias copilot-review='copilot-cli \
  --allow-tool="shell(git diff)" \
  --allow-tool="shell(git log)" \
  --allow-tool="shell(git show)" \
  --allow-tool="shell(npm test)" \
  --allow-tool="shell(npm run lint)" \
  --deny-tool="write" \
  --deny-tool="shell(git commit)" \
  --deny-tool="shell(git push)"'

# Safe coding — AI can edit files and test, but not push
alias copilot-code='copilot-cli \
  --allow-tool="write" \
  --allow-tool="shell(npm test)" \
  --allow-tool="shell(npm run build)" \
  --allow-tool="shell(npm run lint)" \
  --allow-tool="shell(git diff)" \
  --allow-tool="shell(git add)" \
  --allow-tool="shell(git status)" \
  --deny-tool="shell(rm)" \
  --deny-tool="shell(git push)" \
  --deny-tool="shell(git checkout main)"'

# Infrastructure — AI can plan but not apply
alias copilot-infra='copilot-cli \
  --allow-tool="write" \
  --allow-tool="shell(terraform plan)" \
  --allow-tool="shell(terraform validate)" \
  --allow-tool="shell(terraform fmt)" \
  --deny-tool="shell(terraform apply)" \
  --deny-tool="shell(terraform destroy)" \
  --deny-tool="shell(kubectl delete)" \
  --deny-tool="shell(kubectl exec)"'
```

### Understanding --allow-all-tools

```bash
# ⚠️ This disables ALL permission checks
copilot-cli --allow-all-tools

# The AI can now:
# ✅ Read any file
# ✅ Write any file
# ✅ Delete any file
# ✅ Run any command
# ✅ Push to any branch
# ✅ Access any MCP tool
# ✅ Make network requests
# ✅ Modify system configuration
#
# If prompt injection succeeds, ALL of these are available to the attacker
```

**When --allow-all-tools is acceptable:**
- Disposable sandbox environment (Docker, VM)
- Combined with `--deny-tool` for critical operations
- Personal experimentation on non-important code
- Automated pipelines with other safeguards in place

**When --allow-all-tools is dangerous:**
- Working on production codebases
- Untrusted or unfamiliar repositories
- Shared development environments
- When MCP servers with write capabilities are connected

### The Yolo Command

Some AI tools offer a "zero approval" mode. Understand what this means:

```
┌──────────────────────────────────────────────────────┐
│              ZERO APPROVAL MODE                      │
│                                                      │
│  What it does:                                       │
│  • Skips ALL confirmation prompts                    │
│  • AI executes every action without asking           │
│  • No opportunity to review before execution         │
│                                                      │
│  What it means for security:                         │
│  • Permission model = DISABLED                       │
│  • Human-in-the-loop = REMOVED                       │
│  • Blast radius = UNLIMITED                          │
│                                                      │
│  Appropriate use:                                    │
│  • Throwaway environments only                       │
│  • Never with untrusted code                         │
│  • Never with production access                      │
│  • Never with credentials in environment             │
│                                                      │
│  ⚠️ You are giving FULL AUTONOMY to an AI that      │
│     can be manipulated by prompt injection            │
└──────────────────────────────────────────────────────┘
```

## Agency Levels Framework

Use this framework to match AI agency to your task:

```
┌─────────────────────────────────────────────────────┐
│              AI AGENCY LEVELS                        │
│                                                      │
│  Level 0: READ-ONLY                                 │
│  • Can read files and run read-only commands         │
│  • Cannot modify anything                            │
│  • Use for: Code review, analysis, Q&A              │
│                                                      │
│  Level 1: EDIT                                      │
│  • Can read and edit files                           │
│  • Cannot run arbitrary commands                     │
│  • Use for: Refactoring, bug fixes, documentation   │
│                                                      │
│  Level 2: BUILD & TEST                              │
│  • Can edit files and run build/test commands        │
│  • Cannot delete, push, or deploy                    │
│  • Use for: Feature development, test fixes         │
│                                                      │
│  Level 3: FULL LOCAL                                │
│  • Can do anything locally                           │
│  • Cannot push, deploy, or access external services  │
│  • Use for: Complex tasks in sandboxed environments  │
│                                                      │
│  Level 4: AUTONOMOUS ⚠️                             │
│  • Can do anything including push and deploy         │
│  • Use for: NEVER in production, only disposable     │
│             sandboxes                                 │
└─────────────────────────────────────────────────────┘
```

### Configuring Each Level

```bash
# Level 0: Read-Only
copilot-cli \
  --deny-tool='write' \
  --allow-tool='shell(git diff)' \
  --allow-tool='shell(git log)' \
  --allow-tool='shell(cat)'

# Level 1: Edit Only
copilot-cli \
  --allow-tool='write' \
  --deny-tool='shell(*)'

# Level 2: Build & Test
copilot-cli \
  --allow-tool='write' \
  --allow-tool='shell(npm test)' \
  --allow-tool='shell(npm run build)' \
  --allow-tool='shell(npm run lint)' \
  --deny-tool='shell(rm)' \
  --deny-tool='shell(git push)'

# Level 3: Full Local
copilot-cli \
  --allow-all-tools \
  --deny-tool='shell(git push)' \
  --deny-tool='shell(curl)' \
  --deny-tool='shell(wget)' \
  --deny-tool='shell(ssh)'
```

## Checklist: Preventing Excessive Agency

```markdown
### Excessive Agency Prevention Checklist

- [ ] Defined the minimum tool set needed for this task
- [ ] Used --deny-tool for all destructive operations
- [ ] Blocked git push (commit locally, push manually)
- [ ] Blocked file deletion commands
- [ ] Blocked network access if not needed
- [ ] Disconnected unneeded MCP servers
- [ ] Running in a sandboxed environment for untrusted code
- [ ] Human approval required for all permanent changes
- [ ] Reviewed AI actions in terminal before approving
- [ ] No --allow-all-tools without compensating --deny-tool rules
```

## Tips

✅ **Do:**
- Match agency level to the specific task — not to your general workflow
- Use `--deny-tool` as a safety net even when you trust the AI
- Create aliases for common agency profiles to make safety easy
- Always block `git push` — review and push manually
- Sandbox the environment when granting higher agency levels

❌ **Don't:**
- Use `--allow-all-tools` as your default because "it's convenient"
- Grant Level 4 (autonomous) agency outside of disposable environments
- Assume the AI will self-limit its actions to what's reasonable
- Ignore the risk of prompt injection escalating agency
- Forget that MCP servers expand the AI's agency beyond local tools

---

## Next Steps

- [Least Privilege for AI Agents](../05-sandboxing-permissions/least-privilege.md) — The systematic approach to minimal agency
- [AI Agent Permission Models](../05-sandboxing-permissions/permission-models.md) — Detailed permission configuration
- [LLM01: Prompt Injection](./llm01-prompt-injection.md) — Why excessive agency + injection = disaster
- [OWASP LLM Top 10 Overview](./overview.md) — Full risk landscape for coding workflows
