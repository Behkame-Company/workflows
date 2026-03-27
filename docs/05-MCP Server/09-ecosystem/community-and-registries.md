# Community & Registries

> Where to find, share, and discover MCP servers.

---

## Official Resources

### MCP Specification
- **Website**: [modelcontextprotocol.io](https://modelcontextprotocol.io)
- **GitHub**: [github.com/modelcontextprotocol](https://github.com/modelcontextprotocol)
- **Spec**: Full protocol specification with examples

### Official Server Repository
- **Repo**: [github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)
- **Contains**: Reference implementations maintained by Anthropic
- **Includes**: GitHub, Filesystem, PostgreSQL, Brave Search, Puppeteer, etc.

---

## Server Registries & Directories

### 1. Awesome MCP Servers
- **URL**: [github.com/punkpeye/awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)
- **Type**: Community-curated list
- **Size**: 1,000+ servers (and growing)
- **Organized by**: Category (databases, dev tools, web, etc.)

### 2. mcp.so
- **URL**: [mcp.so](https://mcp.so)
- **Type**: Searchable registry website
- **Features**: Search, categories, ratings, install instructions
- **Best for**: Discovering servers by use case

### 3. Smithery
- **URL**: [smithery.ai](https://smithery.ai)
- **Type**: MCP server marketplace
- **Features**: One-click install, version management, reviews
- **Best for**: Easy installation and management

### 4. npm Registry
- **Search**: `npm search @modelcontextprotocol`
- **Best for**: TypeScript/Node.js servers
- **Install**: `npx -y @scope/server-name`

### 5. PyPI
- **Search**: `pip search mcp` (or search on pypi.org)
- **Best for**: Python servers
- **Install**: `pip install server-name && server-name`

---

## Evaluating Servers

Before installing a server, check:

| Criteria | What to Check |
|----------|---------------|
| **Source** | Official repo, known organization, or random author? |
| **Stars/Downloads** | Widely used or brand new? |
| **Maintenance** | Recent commits? Active issues? |
| **Security** | Open source? Reviewed code? No suspicious permissions? |
| **Dependencies** | Minimal deps? Known vulnerabilities? |
| **Documentation** | Clear setup instructions? Tool descriptions? |

### Red Flags 🚩
- No source code available
- Requires excessive permissions
- No documentation
- Single author, no community
- Hasn't been updated in 6+ months
- Asks for credentials it shouldn't need

---

## Contributing Your Own Server

### Publishing Checklist
1. **Open source** the code on GitHub
2. **Document** all tools, resources, and prompts
3. **Include** a README with setup instructions
4. **Publish** to npm or PyPI
5. **Submit** to Awesome MCP Servers via PR
6. **Register** on mcp.so or Smithery
7. **Test** with at least 2-3 different clients

---

## Key Takeaways

1. **Start with official servers** — They're maintained, tested, and trusted
2. **Awesome MCP Servers is the best discovery tool** — Curated and categorized
3. **Verify before installing** — Check source, popularity, and maintenance
4. **Contribute back** — Share your servers with the community

---

## Next Steps

- 🔗 [MCP in Prompt Files](mcp-in-prompt-files.md) — Referencing tools in `.prompt.md`
- 🔗 [Popular Servers](popular-servers.md) — Top 10 essential servers
