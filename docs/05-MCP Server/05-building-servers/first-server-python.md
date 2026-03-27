# Your First MCP Server (Python)

> Build a working MCP server in Python in under 10 minutes using the FastMCP framework.

---

## Prerequisites

- Python 3.10+
- Basic Python knowledge
- A terminal

---

## Step 1: Set Up the Project

```bash
# Create project directory
mkdir my-mcp-server && cd my-mcp-server

# Create virtual environment (using uv — recommended)
uv init
uv venv
source .venv/bin/activate    # Linux/macOS
# .venv\Scripts\activate     # Windows

# Install MCP SDK
uv add "mcp[cli]"
```

Or with pip:
```bash
python -m venv .venv
source .venv/bin/activate
pip install "mcp[cli]"
```

---

## Step 2: Create the Server

Create `server.py`:

```python
from mcp.server.fastmcp import FastMCP

# Create server instance
mcp = FastMCP("my-first-server")


# ── Tool: A function the AI can call ──
@mcp.tool
async def add_numbers(a: float, b: float) -> str:
    """Add two numbers together. Use this for arithmetic addition."""
    result = a + b
    return f"{a} + {b} = {result}"


@mcp.tool
async def get_weather(city: str) -> str:
    """Get the current weather for a city. Returns a mock response for demo purposes."""
    # In production, you'd call a real weather API
    return f"Weather in {city}: 72°F, Sunny, Humidity: 45%"


@mcp.tool
async def count_words(text: str) -> str:
    """Count the number of words in the given text."""
    word_count = len(text.split())
    return f"The text contains {word_count} words."


# ── Resource: Read-only data the AI can access ──
@mcp.resource("config://app/info")
async def get_app_info() -> str:
    """Basic application information."""
    return "App: My First MCP Server\nVersion: 1.0.0\nStatus: Running"


# ── Prompt: A reusable template ──
@mcp.prompt
async def analyze_text(text: str) -> str:
    """Analyze a piece of text for readability and key themes."""
    return f"""Please analyze the following text:

{text}

Provide:
1. Main theme
2. Key points (bullet list)
3. Readability assessment
4. Suggested improvements"""


# Run the server
if __name__ == "__main__":
    mcp.run()
```

---

## Step 3: Test Locally

### With MCP Inspector
```bash
mcp dev server.py
```
This opens a web-based inspector where you can test tools, resources, and prompts interactively.

### With Claude Desktop

Add to `claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "my-first-server": {
      "command": "python",
      "args": ["/path/to/my-mcp-server/server.py"]
    }
  }
}
```

### With VS Code

Add to `.vscode/settings.json`:
```json
{
  "mcp": {
    "servers": {
      "my-first-server": {
        "command": "python",
        "args": ["${workspaceFolder}/server.py"]
      }
    }
  }
}
```

---

## Step 4: Verify It Works

In your AI assistant, try:
- "Add 42 and 58" → Should call `add_numbers`
- "What's the weather in Tokyo?" → Should call `get_weather`
- "Count the words in: The quick brown fox jumps over the lazy dog" → Should call `count_words`

---

## Step 5: Add a Real-World Tool

Here's a practical example — a tool that reads files:

```python
import os

@mcp.tool
async def read_file(path: str) -> str:
    """Read the contents of a file. Provide the absolute file path."""
    try:
        with open(path, "r", encoding="utf-8") as f:
            content = f.read()
        if len(content) > 10000:
            return content[:10000] + f"\n\n[Truncated — file is {len(content)} chars]"
        return content
    except FileNotFoundError:
        return f"Error: File not found: {path}"
    except PermissionError:
        return f"Error: Permission denied: {path}"
```

---

## Project Structure

```
my-mcp-server/
├── server.py          # Main server file
├── pyproject.toml     # Dependencies
└── README.md          # Documentation
```

For larger servers:
```
my-mcp-server/
├── src/
│   ├── __init__.py
│   ├── server.py      # MCP server setup
│   ├── tools/         # Tool implementations
│   │   ├── github.py
│   │   └── database.py
│   └── resources/     # Resource providers
│       └── config.py
├── tests/
├── pyproject.toml
└── README.md
```

---

## Key Takeaways

1. **FastMCP** makes Python MCP servers simple — decorators define tools/resources/prompts
2. **Test with `mcp dev`** before connecting to an AI client
3. **Tools need clear descriptions** — the AI reads them to decide when to use each tool
4. **Handle errors gracefully** — return error messages, don't crash the server
5. **Start simple, iterate** — get basic tools working, then add complexity

---

## Next Steps

- 🔗 [Your First Server (TypeScript)](first-server-typescript.md) — The TypeScript equivalent
- 🔗 [Tool Design Patterns](tool-design-patterns.md) — Design tools AI agents love
- 🔗 [Testing & Debugging](testing-and-debugging.md) — Thorough testing strategies
