# MCP Troubleshooting Guide

> Common problems, diagnosis steps, and fixes for MCP server issues.

---

## Diagnostic Checklist

When something isn't working, check in this order:

1. **Is the server running?** Check process list
2. **Is the config correct?** Validate JSON, check paths
3. **Are environment variables set?** Missing tokens = auth failures
4. **Is the transport working?** stdio vs HTTP confusion
5. **Are tools appearing?** Client may need restart

---

## Common Problems & Fixes

### Problem: Server Not Appearing in Client

**Symptoms**: No tools listed, server appears disconnected.

| Check | Fix |
|-------|-----|
| Config file location | VS Code: `.vscode/mcp.json`. Claude: `claude_desktop_config.json` |
| JSON syntax | Validate with a JSON linter — trailing commas break it |
| Command path | Use full path or ensure `npx`/`python` is in PATH |
| Restart client | VS Code: reload window. Claude: restart app |
| Node.js version | MCP servers need Node.js ≥18 |
| Python version | FastMCP needs Python ≥3.10 |

### Problem: "Connection Refused" / "ECONNREFUSED"

```
Cause: Server process failed to start or crashed immediately.

Fixes:
1. Run the server command manually in terminal to see error
2. Check that required packages are installed
3. Verify port is not already in use (HTTP transport)
4. Check firewall isn't blocking the connection
```

### Problem: Tools Exist but Return Errors

```
Cause: Server is running but tool execution fails.

Fixes:
1. Check environment variables (API tokens, database URLs)
2. Verify the external service is accessible
3. Check server stderr output for detailed error messages
4. Try the operation manually outside MCP
```

### Problem: "stdout Not Valid JSON" (stdio)

```
Cause: Something is printing to stdout besides MCP messages.

Fixes:
1. Remove all print() / console.log() statements
2. Redirect logging to stderr: print("debug", file=sys.stderr)
3. Check dependencies — some libraries print to stdout
4. Set PYTHONDONTWRITEBYTECODE=1 to avoid .pyc messages
```

### Problem: AI Uses Wrong Tool / Ignores Tools

```
Cause: Poor tool descriptions or too many tools.

Fixes:
1. Improve tool descriptions (What? When? Returns?)
2. Reduce tool count to 10-15 per server
3. Remove overlapping tools
4. Check for tool name conflicts across servers
```

### Problem: Slow Tool Responses

```
Cause: External API latency or large payloads.

Fixes:
1. Add timeouts to external API calls
2. Paginate large results
3. Cache frequently-requested data
4. Use local servers (stdio) instead of remote when possible
```

---

## Debugging Commands

### Test Server Manually
```bash
# Python server
python -m my_server 2>server.log

# TypeScript server  
npx -y @modelcontextprotocol/server-github 2>server.log

# Then send a test message via stdin:
echo '{"jsonrpc":"2.0","method":"initialize","id":1,"params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}' | python -m my_server
```

### MCP Inspector
```bash
npx @modelcontextprotocol/inspector
# Opens web UI at http://localhost:6274
# Connect to your server and test interactively
```

### Check Server Process
```bash
# Is the server running?
ps aux | grep mcp    # Linux/Mac
Get-Process | Where-Object {$_.CommandLine -like "*mcp*"}  # Windows
```

---

## Key Takeaways

1. **Start with the config** — 80% of issues are configuration errors
2. **Test manually first** — Run the server in terminal to see errors
3. **Use MCP Inspector** — The official debugging tool
4. **Check stderr** — That's where server logs go
5. **Restart after config changes** — Clients don't always hot-reload
