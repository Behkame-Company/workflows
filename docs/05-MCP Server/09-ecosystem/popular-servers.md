# Popular MCP Servers

> The essential MCP servers every development team should know about.

---

## Top 10 Essential Servers

### 1. GitHub MCP Server
**Use**: Repository management, issues, PRs, code search  
**Install**: `npx -y @modelcontextprotocol/server-github`  
**Key Tools**: `create_issue`, `search_code`, `get_file_contents`, `list_pull_requests`  
**Why Essential**: Enables AI to interact with your entire GitHub workflow without leaving the editor.

### 2. Filesystem Server
**Use**: Read/write/search local files  
**Install**: `npx -y @modelcontextprotocol/server-filesystem /path/to/dir`  
**Key Tools**: `read_file`, `write_file`, `search_files`, `list_directory`  
**Why Essential**: The foundation of any code-editing AI workflow.

### 3. Context7
**Use**: Up-to-date library documentation  
**Install**: `npx -y @upstash/context7-mcp`  
**Key Tools**: `resolve-library-id`, `get-library-docs`  
**Why Essential**: Gives AI current documentation instead of relying on training data.

### 4. Playwright MCP
**Use**: Browser automation and testing  
**Install**: `npx -y @anthropic/mcp-playwright`  
**Key Tools**: `navigate`, `click`, `screenshot`, `fill`  
**Why Essential**: AI can visually verify web applications and run E2E tests.

### 5. PostgreSQL Server
**Use**: Database queries and schema inspection  
**Install**: `npx -y @modelcontextprotocol/server-postgres`  
**Key Tools**: `query`, `list_tables`, `describe_table`  
**Why Essential**: AI can understand your data model and write correct queries.

### 6. Sentry Server
**Use**: Error tracking and monitoring  
**Install**: `npx -y @sentry/mcp-server`  
**Key Tools**: `get_errors`, `get_error_details`, `search_errors`  
**Why Essential**: AI can find and debug production errors with real stack traces.

### 7. Figma MCP
**Use**: Design file inspection  
**Install**: Via Figma plugin  
**Key Tools**: `get_file`, `get_components`, `get_styles`  
**Why Essential**: AI can read designs and generate matching code.

### 8. Firecrawl
**Use**: Web scraping and content extraction  
**Install**: `npx -y firecrawl-mcp`  
**Key Tools**: `scrape_url`, `crawl_site`, `search_web`  
**Why Essential**: AI can research documentation, APIs, and examples from the web.

### 9. Docker MCP
**Use**: Container management  
**Install**: Via Docker Desktop extension  
**Key Tools**: `list_containers`, `run_container`, `get_logs`  
**Why Essential**: AI can manage development environments and debug container issues.

### 10. Sequential Thinking
**Use**: Dynamic problem-solving  
**Install**: `npx -y @modelcontextprotocol/server-sequential-thinking`  
**Key Tools**: `create_thought`, `revise_thought`  
**Why Essential**: Gives AI a structured "thinking space" for complex reasoning.

---

## Server Categories

| Category | Servers | Use Case |
|----------|---------|----------|
| **Code & Repos** | GitHub, GitLab, Bitbucket | Source code management |
| **Data & DB** | PostgreSQL, MySQL, SQLite, Redis | Database interaction |
| **Files** | Filesystem, Google Drive, S3 | File management |
| **Web** | Firecrawl, Playwright, Puppeteer | Web interaction |
| **DevOps** | Docker, Kubernetes, Terraform | Infrastructure |
| **Monitoring** | Sentry, Datadog, Grafana | Observability |
| **Docs** | Context7, Notion, Confluence | Documentation |
| **Design** | Figma, Sketch | Design systems |
| **Communication** | Slack, Discord, Email | Team communication |
| **AI & Search** | Exa, Brave Search, Tavily | Web search |
