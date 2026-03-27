# MCP vs Alternatives

> How MCP compares to function calling, plugin systems, LangChain tools, and custom API integrations.

---

## Comparison Matrix

| Feature | MCP | OpenAI Function Calling | ChatGPT Plugins (Deprecated) | LangChain Tools | Custom API Wrappers |
|---------|-----|------------------------|------------------------------|-----------------|-------------------|
| **Vendor neutral** | ✅ | ❌ OpenAI only | ❌ OpenAI only | ⚠️ Python-centric | ✅ |
| **Standardized protocol** | ✅ JSON-RPC | ❌ Vendor schema | ❌ OpenAPI subset | ❌ Framework API | ❌ Bespoke |
| **Bidirectional** | ✅ Server→Client sampling | ❌ One-way | ❌ One-way | ❌ One-way | Varies |
| **Discovery** | ✅ Dynamic `tools/list` | ❌ Static definition | ⚠️ Manifest file | ❌ Code-defined | ❌ Hard-coded |
| **Transport options** | ✅ stdio + HTTP | N/A (API only) | HTTP only | N/A | Varies |
| **Built-in auth** | ✅ OAuth 2.1 | N/A | OAuth | ❌ Manual | ❌ Manual |
| **Human consent** | ✅ Host-mediated | ❌ No model | ⚠️ Install-time | ❌ None | ❌ None |
| **Streaming** | ✅ SSE in HTTP | ✅ Streaming | ❌ No | ⚠️ Callbacks | Varies |
| **Resources (read-only data)** | ✅ First-class | ❌ No concept | ❌ No concept | ❌ Manual | ❌ Manual |
| **Prompt templates** | ✅ First-class | ❌ No concept | ❌ No concept | ⚠️ PromptTemplates | ❌ No |
| **Multi-model support** | ✅ Any LLM | ❌ OpenAI only | ❌ OpenAI only | ✅ Multiple | ✅ |
| **Community servers** | ✅ 10,000+ | N/A | ~1,000 (deprecated) | ~200 tools | N/A |

---

## Detailed Comparisons

### MCP vs OpenAI Function Calling

**Function calling** lets you define tools as JSON schemas that OpenAI models can invoke. MCP is a **superset** of this concept:

```
Function Calling:
  Client → "Here are tools" → Model → "Call this function" → Client executes

MCP:
  Client ←→ Server (bidirectional, stateful sessions)
  Server dynamically advertises tools, resources, prompts
  Server can request AI reasoning (sampling)
  Human consent enforced at host level
```

**When to use function calling**: Simple, single-model use cases with OpenAI only.
**When to use MCP**: Multi-tool environments, team workflows, cross-model compatibility.

### MCP vs LangChain Tools

**LangChain** provides a Python framework for building AI agent tool pipelines. MCP is the **protocol layer** underneath:

| Aspect | MCP | LangChain |
|--------|-----|-----------|
| **Level** | Protocol (wire format) | Framework (code library) |
| **Language** | Any (JSON-RPC) | Python (primarily) |
| **Coupling** | Loose (server is independent) | Tight (tool code lives in agent) |
| **Reusability** | Server works with any MCP client | Tool works only in LangChain |
| **Overhead** | Minimal (protocol only) | Heavy (framework, chains, agents) |

💡 **They're complementary**: LangChain agents can use MCP servers as their tool backend. Many teams use LangChain for orchestration + MCP for tool integration.

### MCP vs Custom API Wrappers

**Custom wrappers** are the DIY approach: write code to call APIs and feed results to your AI.

| Aspect | MCP | Custom Wrappers |
|--------|-----|----------------|
| **Development time** | Minutes (use existing servers) | Hours/days per integration |
| **Maintenance** | Community-maintained | You maintain everything |
| **Discovery** | Dynamic; AI sees available tools | Hard-coded; changes need code updates |
| **Security** | Standardized auth + consent model | Roll your own |
| **Portability** | Works across all AI tools | Locked to your implementation |

---

## When to Use What

```
Decision Tree:
├── Single model, simple tools? → Function Calling is fine
├── Python-heavy, chain orchestration? → LangChain + MCP servers
├── Multi-model team, shared tools? → MCP (primary choice)
├── Enterprise, multiple AI assistants? → MCP + Gateway
└── Rapid prototyping, one-off? → Custom wrapper, then migrate to MCP
```

---

## Migration Paths

### From Function Calling → MCP

1. Wrap your function definitions as MCP tool handlers
2. Add `tools/list` discovery endpoint
3. Implement stdio or HTTP transport
4. Your functions become reusable across any MCP client

### From LangChain Tools → MCP

1. Extract tool logic from LangChain classes
2. Create MCP server with same functionality
3. Keep LangChain for orchestration, use MCP for tools
4. Gain cross-model, cross-tool compatibility

### From Custom Wrappers → MCP

1. Identify which wrappers are most-used
2. Check if community MCP server already exists (often it does)
3. If not, wrap your existing code in an MCP server
4. Retire custom integration code

---

## Key Takeaways

1. **MCP is a protocol, not a framework** — it works with frameworks like LangChain, not against them
2. **Function calling is a subset** of what MCP offers — MCP adds resources, prompts, bidirectional communication
3. **Custom wrappers are the anti-pattern** — they create maintenance burden and vendor lock-in
4. **MCP is the migration target** — most teams should aim for MCP as their standard tool integration layer

---

## Next Steps

- 🔗 [Key Terminology](terminology.md) — Learn the MCP vocabulary
- 🔗 [The Client-Host-Server Model](../02-architecture/client-host-server.md) — Understand the architecture

---

*"Function calling is a feature. MCP is a standard."*
