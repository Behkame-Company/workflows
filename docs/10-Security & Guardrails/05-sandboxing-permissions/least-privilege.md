# Least Privilege for AI Agents

> Applying the principle of least privilege to AI coding tools — granting only the minimum permissions needed for the task at hand, and nothing more.

---

## The Principle

**Least privilege** is a foundational security concept: every actor in a system should have only the permissions necessary to perform its function. Applied to AI agents, this means:

```
┌──────────────────────────────────────────────────────┐
│           LEAST PRIVILEGE FOR AI AGENTS               │
│                                                      │
│  ❌ "Let the AI do whatever it needs"                │
│  ✅ "Let the AI do exactly what this task requires"  │
│                                                      │
│  Scope down:                                         │
│    📁 File access    → Only relevant directories     │
│    🖥️  Shell commands → Only specific tools needed    │
│    🌐 Network        → Only required endpoints       │
│    🔧 MCP servers    → Only needed tools             │
│    ⏱️  Duration       → Only for the task lifetime    │
└──────────────────────────────────────────────────────┘
```

## Why It Matters for AI

Traditional least privilege assumes actors are **intentional** — a user might try to escalate privileges, but they know what they're doing. AI agents are different:

- **They don't understand security context** — an AI won't hesitate to `rm -rf` if it thinks it helps
- **They can be manipulated** — prompt injection can redirect their actions
- **They take multi-step actions** — a series of individually safe operations can be dangerous together
- **They operate at machine speed** — mistakes compound before a human can intervene

Least privilege ensures that even if the AI is **compromised or confused**, the damage is bounded.

## Four Dimensions of AI Privilege

### 1. File Access

Limit which files and directories the AI can read and modify.

```
┌─────────────────────────────────────────────────────┐
│  PROJECT ROOT                                        │
│  ├── src/                  ← ✅ AI can read/write   │
│  │   ├── components/       ← ✅ Task-relevant       │
│  │   ├── auth/             ← ⚠️ Sensitive — review  │
│  │   └── utils/            ← ✅ Task-relevant       │
│  ├── tests/                ← ✅ AI can read/write   │
│  ├── .env                  ← ❌ NEVER expose        │
│  ├── .env.local            ← ❌ NEVER expose        │
│  ├── secrets/              ← ❌ NEVER expose        │
│  ├── deploy/               ← ❌ Block access        │
│  └── .github/workflows/    ← ⚠️ Review all changes  │
└─────────────────────────────────────────────────────┘
```

**Practical approach — launch from the right directory:**

```bash
# ❌ Bad: Launch from home directory — AI sees everything
cd ~
copilot-cli "Fix the login bug"

# ✅ Good: Launch from project directory — AI scope is limited
cd ~/projects/my-app
copilot-cli "Fix the login bug"

# ✅ Better: Launch from specific subdirectory
cd ~/projects/my-app/src/auth
copilot-cli "Fix the login bug in this module"
```

> **Trusted directories concept**: Copilot CLI considers the directory it's launched from as the workspace scope. Launching from a narrow directory naturally limits file access.

### 2. Shell Commands

Only allow the specific commands the task needs.

```bash
# ❌ Over-privileged: allow everything
copilot-cli --allow-all-tools

# ✅ Least privilege: only what's needed for a "fix tests" task
copilot-cli \
  --allow-tool='write' \
  --allow-tool='shell(npm test)' \
  --allow-tool='shell(npm run lint)' \
  --deny-tool='shell(rm)' \
  --deny-tool='shell(git push)' \
  --deny-tool='shell(npm install)' \
  --deny-tool='shell(curl)'
```

**Command privilege levels:**

| Level | Commands | Risk |
|-------|----------|------|
| **Read-only** | `cat`, `grep`, `find`, `git log`, `git diff` | Low |
| **Build/Test** | `npm test`, `npm run build`, `pytest`, `go test` | Low-Medium |
| **Install** | `npm install`, `pip install`, `apt-get` | Medium |
| **Write** | `git commit`, `git add`, file edits | Medium |
| **Network** | `curl`, `wget`, `fetch` | High |
| **Destructive** | `rm`, `git push`, `docker rm`, `kubectl delete` | **Critical** |
| **Admin** | `sudo`, `chmod`, `chown`, service management | **Critical** |

### 3. Network Access

Restrict which network resources the AI can reach.

```bash
# For dependency installation (needs network temporarily)
copilot-cli \
  --allow-tool='shell(npm install)' \
  --allow-tool='shell(npm test)'

# For offline coding (no network needed)
# Use in conjunction with Docker --network=none
docker run --network=none -v $(pwd):/workspace ai-sandbox
```

See [Network Access Control](./network-access.md) for detailed network policies.

### 4. MCP Server Tools

Only expose the MCP tools required for the current task.

```bash
# ❌ Over-privileged: allow all GitHub operations
copilot-cli --allow-tool='github(*)'

# ✅ Least privilege: only what's needed for code review
copilot-cli \
  --allow-tool='github(get_pull_request)' \
  --allow-tool='github(list_pull_request_files)' \
  --deny-tool='github(merge_pull_request)' \
  --deny-tool='github(create_issue)'
```

## Workflow-Specific Configurations

### Frontend Development

```bash
# Task: Fix CSS styling issues
copilot-cli \
  --allow-tool='write' \
  --allow-tool='shell(npm run dev)' \
  --allow-tool='shell(npm test)' \
  --allow-tool='shell(npm run lint)' \
  --allow-tool='shell(npx prettier)' \
  --allow-tool='shell(git diff)' \
  --allow-tool='shell(git status)' \
  --deny-tool='shell(rm)' \
  --deny-tool='shell(git push)' \
  --deny-tool='shell(npm publish)' \
  --deny-tool='shell(npm install)'
```

### Backend API Development

```bash
# Task: Implement a new API endpoint
copilot-cli \
  --allow-tool='write' \
  --allow-tool='shell(python)' \
  --allow-tool='shell(pip install)' \
  --allow-tool='shell(pytest)' \
  --allow-tool='shell(flask run)' \
  --allow-tool='shell(git diff)' \
  --deny-tool='shell(rm)' \
  --deny-tool='shell(git push)' \
  --deny-tool='shell(curl)' \
  --deny-tool='shell(wget)' \
  --deny-tool='shell(ssh)'
```

### DevOps / Infrastructure

```bash
# Task: Update Terraform configuration
copilot-cli \
  --allow-tool='write' \
  --allow-tool='shell(terraform plan)' \
  --allow-tool='shell(terraform fmt)' \
  --allow-tool='shell(terraform validate)' \
  --allow-tool='shell(git diff)' \
  --deny-tool='shell(terraform apply)' \
  --deny-tool='shell(terraform destroy)' \
  --deny-tool='shell(kubectl delete)' \
  --deny-tool='shell(kubectl exec)' \
  --deny-tool='shell(aws)' \
  --deny-tool='shell(rm)'
```

### Code Review (Read-Only)

```bash
# Task: Review code quality — no modifications allowed
copilot-cli \
  --allow-tool='shell(git diff)' \
  --allow-tool='shell(git log)' \
  --allow-tool='shell(git show)' \
  --allow-tool='shell(npm test)' \
  --allow-tool='shell(npm run lint)' \
  --deny-tool='write' \
  --deny-tool='shell(git commit)' \
  --deny-tool='shell(git push)'
```

## Privilege Escalation Checklist

Before expanding AI permissions, verify each step:

```markdown
### Permission Escalation Checklist

- [ ] The AI explicitly stated why it needs the additional permission
- [ ] The permission is necessary for the current task (not "nice to have")
- [ ] A narrower permission wouldn't achieve the same result
- [ ] The permission is time-bounded (session-only, not permanent)
- [ ] I've reviewed what the AI plans to do with the permission
- [ ] Destructive permissions (rm, push, deploy) have deny rules in place
- [ ] I'm comfortable with the blast radius if the AI misbehaves
```

## Tips

✅ **Do:**
- Start with **zero permissions** and add only what's needed
- Use `--deny-tool` as a safety net even when you trust the workflow
- Launch Copilot CLI from the **narrowest relevant directory**
- Create shell aliases or scripts for common permission configurations
- Review AI actions in the terminal before approving multi-step operations

❌ **Don't:**
- Use `--allow-all-tools` as a default because "it's faster"
- Grant network access for tasks that don't need it
- Allow `git push` — commit locally and push yourself after review
- Leave MCP servers connected when they're not needed for the task
- Assume the AI will self-limit — it will use any capability available
