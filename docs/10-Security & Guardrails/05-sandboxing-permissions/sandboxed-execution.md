# Sandboxed Execution for AI Agents

> Running AI coding agents in isolated environments so that even if prompt injection succeeds or the AI makes a mistake, the blast radius is contained to a disposable sandbox.

---

## Why Sandbox AI Agents?

Permission models control what the AI **is allowed** to do. Sandboxing controls what the AI **is able** to do — even if permissions fail.

```
┌──────────────────────────────────────────────────────┐
│              DEFENSE IN DEPTH                        │
│                                                      │
│  Layer 1: Permission Model                           │
│    → AI asks before acting                           │
│    → Can be bypassed by prompt injection             │
│                                                      │
│  Layer 2: Sandbox                                    │
│    → AI physically cannot access resources           │
│    → Even if injection succeeds, damage is limited   │
│                                                      │
│  Layer 3: Human Review                               │
│    → Changes reviewed before merging                 │
│    → Final safety net                                │
└──────────────────────────────────────────────────────┘
```

If prompt injection convinces the AI to ignore its instructions, the sandbox ensures it **still can't** access your secrets, delete your files, or reach your production systems.

## Sandbox Options

### 1. Docker Containers

The most practical option for local development. Mount only what's needed, disable network, drop capabilities.

```dockerfile
# Dockerfile.ai-sandbox
FROM node:20-slim

# Create non-root user
RUN useradd -m -s /bin/bash aiagent
USER aiagent
WORKDIR /workspace

# No secrets, no SSH keys, no git credentials
# Only project files are mounted at runtime
```

```bash
# Run AI agent in a sandboxed container
docker run --rm \
  -v "$(pwd):/workspace" \
  --network=none \
  --cap-drop=ALL \
  --read-only \
  --tmpfs /tmp:noexec,nosuid,size=100m \
  --memory=2g \
  --cpus=1 \
  --security-opt=no-new-privileges \
  ai-sandbox
```

**Flag breakdown:**

| Flag | Purpose |
|------|---------|
| `--network=none` | No network access — can't exfiltrate data |
| `--cap-drop=ALL` | Drop all Linux capabilities |
| `--read-only` | Root filesystem is read-only |
| `--tmpfs /tmp` | Writable temp space with limits |
| `--memory=2g` | Prevent resource exhaustion |
| `--cpus=1` | Limit CPU consumption |
| `--security-opt=no-new-privileges` | Can't escalate privileges |

### 2. Virtual Machines

Full isolation for maximum security. Best for untrusted repositories.

```
┌──────────────────────────────────────────────────────┐
│                    HOST MACHINE                       │
│                                                      │
│  ┌────────────────────────────────────────────┐      │
│  │              VIRTUAL MACHINE                │      │
│  │                                             │      │
│  │  ┌──────────────────────────────────┐      │      │
│  │  │      AI Agent Workspace          │      │      │
│  │  │                                  │      │      │
│  │  │  • Cloned repo (read-only src)   │      │      │
│  │  │  • Limited tools installed       │      │      │
│  │  │  • No access to host filesystem  │      │      │
│  │  │  • No access to host network     │      │      │
│  │  │  • Snapshot/restore available    │      │      │
│  │  └──────────────────────────────────┘      │      │
│  │                                             │      │
│  │  ❌ No host filesystem access               │      │
│  │  ❌ No SSH keys or credentials              │      │
│  │  ❌ No production network access            │      │
│  └────────────────────────────────────────────┘      │
│                                                      │
│  ✅ Host machine is completely protected              │
└──────────────────────────────────────────────────────┘
```

**Tools for VM-based sandboxing:**
- **Multipass** — lightweight Ubuntu VMs (cross-platform)
- **Lima** — Linux VMs on macOS
- **Hyper-V** — Windows-native virtualization
- **VirtualBox** — cross-platform, free

### 3. GitHub Codespaces

Cloud-based development containers managed by GitHub. Excellent for team-wide sandboxing.

```jsonc
// .devcontainer/devcontainer.json
{
  "name": "AI Agent Sandbox",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },

  // Restrict port forwarding
  "forwardPorts": [],
  "portsAttributes": {
    "*": { "onAutoForward": "ignore" }
  },

  // Post-create setup — install only what's needed
  "postCreateCommand": "npm ci --ignore-scripts",

  // Environment restrictions
  "containerEnv": {
    "NODE_ENV": "development",
    "CI": "true"
  },

  // Mount restrictions — no host filesystem access
  "mounts": [],

  // Run as non-root
  "remoteUser": "node"
}
```

**Codespace advantages:**
- Ephemeral — destroy and recreate easily
- No access to local machine
- Configurable resource limits
- Team-wide consistency via devcontainer config
- GitHub manages the infrastructure

### 4. Copilot Coding Agent (GitHub-Managed)

When using Copilot Coding Agent (the GitHub-hosted agent that works on issues/PRs), the agent runs in a **GitHub-managed container**:

```
┌──────────────────────────────────────────────────────┐
│          COPILOT CODING AGENT SANDBOX                │
│                                                      │
│  ✅ Runs in isolated GitHub-managed container        │
│  ✅ Only has access to the repository checkout       │
│  ✅ Cannot access other repos or GitHub resources    │
│  ✅ Changes are submitted as PRs for review          │
│  ✅ Uses repository's devcontainer.json if present   │
│                                                      │
│  Configuration via devcontainer.json:                │
│    • Control installed tools and runtimes            │
│    • Set environment variables                       │
│    • Configure build and test commands               │
│                                                      │
│  Customize agent behavior with:                      │
│    • copilot-setup-steps.yml (environment setup)     │
│    • AGENTS.md (instructions and guidelines)         │
└──────────────────────────────────────────────────────┘
```

```yaml
# .github/copilot-setup-steps.yml
# Defines how the Copilot Coding Agent sets up its environment
steps:
  - name: Install dependencies
    run: npm ci --ignore-scripts
  - name: Build project
    run: npm run build
  - name: Verify environment
    run: npm test -- --passWithNoTests
```

### 5. WSL with Restricted Permissions

For Windows users who want lightweight isolation:

```powershell
# Create a dedicated WSL instance for AI work
wsl --import AISandbox C:\WSL\AISandbox ubuntu-base.tar

# Run AI agent in the restricted WSL instance
wsl -d AISandbox -- bash -c "cd /workspace && copilot-cli"
```

```bash
# Inside WSL — restrict the user
sudo useradd -m -s /bin/bash aiagent
sudo chown -R aiagent:aiagent /workspace
sudo -u aiagent copilot-cli --allow-tool='write'
```

## Docker Compose Template for AI Sandboxing

A complete template for running AI agents in Docker:

```yaml
# docker-compose.ai-sandbox.yml
version: "3.8"

services:
  ai-agent:
    build:
      context: .
      dockerfile: Dockerfile.ai-sandbox
    volumes:
      # Mount project files — read-write for workspace
      - ./src:/workspace/src
      - ./tests:/workspace/tests
      - ./package.json:/workspace/package.json:ro
      - ./tsconfig.json:/workspace/tsconfig.json:ro
    # No network access by default
    network_mode: none
    # Resource limits
    deploy:
      resources:
        limits:
          cpus: "2"
          memory: 4G
    # Security settings
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
    # No access to host secrets
    environment:
      - HOME=/home/aiagent
      - NODE_ENV=development
```

## Devcontainer Template for Copilot

```jsonc
// .devcontainer/devcontainer.json — optimized for AI agents
{
  "name": "Secure AI Development",
  "build": {
    "dockerfile": "Dockerfile",
    "context": ".."
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "github.copilot",
        "github.copilot-chat"
      ]
    }
  },
  // Only forward ports explicitly needed
  "forwardPorts": [3000],
  // Prevent automatic port forwarding
  "portsAttributes": {
    "*": { "onAutoForward": "ignore" }
  },
  // Lifecycle scripts
  "postCreateCommand": "npm ci",
  "postStartCommand": "npm run build",
  // Run as non-root user
  "remoteUser": "node",
  // Feature installations
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {},
    "ghcr.io/devcontainers/features/node:1": {
      "version": "20"
    }
  }
}
```

## When to Sandbox

| Scenario | Sandbox Level | Recommendation |
|----------|--------------|----------------|
| Your own trusted project | Optional | Permissions may suffice |
| Open-source contribution | Recommended | Docker container minimum |
| Untrusted repository | **Required** | VM or Codespace |
| Production codebase | **Required** | Docker or Codespace |
| Running MCP servers | Recommended | Container per MCP server |
| CI/CD with AI agents | **Required** | Ephemeral containers only |
| Security-sensitive code | **Required** | VM with no network |

## Tips

✅ **Do:**
- Default to Docker containers for everyday AI agent work
- Use `--network=none` unless the task specifically requires network access
- Destroy and recreate sandboxes between tasks on different repositories
- Use devcontainer.json to standardize sandbox configuration for your team
- Snapshot VM state before running AI agents on important work

❌ **Don't:**
- Run AI agents directly on your host machine for untrusted repositories
- Mount your home directory or SSH keys into a sandbox
- Give sandbox containers access to Docker socket (`/var/run/docker.sock`)
- Assume cloud-based environments (Codespaces) are automatically fully secure
- Share sandboxed environments between different untrusted repositories
