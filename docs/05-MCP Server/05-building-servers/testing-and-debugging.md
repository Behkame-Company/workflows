# Testing & Debugging MCP Servers

> How to validate that your MCP server works correctly before connecting it to an AI assistant.

---

## Testing Tools

### 1. MCP Inspector (Recommended for Development)

The official interactive testing tool:

```bash
# Python
mcp dev server.py

# TypeScript
npx @modelcontextprotocol/inspector npx tsx src/server.ts
```

The Inspector provides:
- Visual tool/resource/prompt listing
- Interactive tool execution with parameter input
- Response inspection
- Error visualization

### 2. MCP CLI

Command-line testing without a GUI:

```bash
# List tools
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | python server.py

# Call a tool
echo '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"add_numbers","arguments":{"a":5,"b":3}}}' | python server.py
```

### 3. Unit Tests

Test tool logic independently:

```python
# test_tools.py
import pytest
from server import add_numbers, get_weather

@pytest.mark.asyncio
async def test_add_numbers():
    result = await add_numbers(5, 3)
    assert result == "5 + 3 = 8"

@pytest.mark.asyncio
async def test_add_negative():
    result = await add_numbers(-1, 1)
    assert result == "-1 + 1 = 0"

@pytest.mark.asyncio
async def test_weather():
    result = await get_weather("Tokyo")
    assert "Tokyo" in result
```

---

## Debugging Checklist

### Server Won't Start
- [ ] Check Python/Node version matches requirements
- [ ] Verify all dependencies are installed
- [ ] Look at stderr output for error messages
- [ ] Ensure no syntax errors in server code

### Client Can't Connect
- [ ] Verify the command path is correct in config
- [ ] Check environment variables are set (API keys, etc.)
- [ ] Try running the server command manually in terminal
- [ ] Check for port conflicts (HTTP transport)

### Tools Not Appearing
- [ ] Verify `tools/list` returns your tools (use Inspector)
- [ ] Check tool descriptions are present and non-empty
- [ ] Ensure capabilities include `tools: {}`
- [ ] Restart the AI client after config changes

### Tool Calls Failing
- [ ] Check parameter types match the schema
- [ ] Verify required parameters are being passed
- [ ] Look at stderr for error stack traces
- [ ] Test the tool function independently (unit test)

### AI Choosing Wrong Tool
- [ ] Review tool descriptions — are they specific enough?
- [ ] Check for overlapping tool names/descriptions
- [ ] Ensure description explains **when** to use the tool, not just **what** it does

---

## Logging Best Practices

```python
import sys
import logging

# Configure logging to stderr (never stdout!)
logging.basicConfig(
    stream=sys.stderr,
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(message)s"
)
logger = logging.getLogger("my-server")

@mcp.tool
async def query_database(sql: str) -> str:
    """Run a read-only SQL query."""
    logger.info(f"Executing query: {sql}")
    try:
        result = await db.execute(sql)
        logger.debug(f"Query returned {len(result)} rows")
        return format_results(result)
    except Exception as e:
        logger.error(f"Query failed: {e}", exc_info=True)
        return f"Error: {str(e)}"
```

---

## Key Takeaways

1. **Use MCP Inspector** for interactive development testing
2. **Write unit tests** for tool logic independently
3. **Log to stderr**, never stdout
4. **Test tool descriptions** — if the AI picks the wrong tool, the description needs work
5. **Check the basics first** — most issues are path, config, or dependency problems

---

## Next Steps

- 🔗 [Server Design Principles](../06-best-practices/server-design-principles.md) — Core design rules
- 🔗 [Troubleshooting Guide](../11-practical/troubleshooting.md) — Common problems and fixes
