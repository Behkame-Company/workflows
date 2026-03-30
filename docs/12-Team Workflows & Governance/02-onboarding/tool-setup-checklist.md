# Tool Setup Checklist

> A standardized environment checklist ensuring every team member has identical AI tool configuration. Print this out, pin it to your monitor, and check every box.

---

## Copilot CLI

### Installation and Authentication
- [ ] Install GitHub Copilot CLI (`npm install -g @githubnext/github-copilot-cli` or via GitHub CLI extension)
- [ ] Authenticate with GitHub account (`github-copilot-cli auth`)
- [ ] Verify license is active (Copilot Individual, Business, or Enterprise)
- [ ] Confirm CLI responds to prompts

### Configuration
- [ ] Configure default model (if team has a preference)
- [ ] Add team-approved MCP servers to configuration
- [ ] Verify `copilot-instructions.md` is detected in the repo root or `.github/`
- [ ] Test that instruction file rules are followed

### Verification
```bash
# Test that Copilot CLI is working
copilot-cli --version

# Test a simple prompt to verify authentication
copilot "explain what this project does"
```

---

## Claude Code

### Installation and Authentication
- [ ] Install Claude Code CLI (`npm install -g @anthropic-ai/claude-code`)
- [ ] Authenticate with Anthropic account or team API key
- [ ] Verify license tier matches team plan
- [ ] Confirm CLI responds to prompts

### Configuration
- [ ] Verify `CLAUDE.md` is detected in the repo root
- [ ] Configure any team-specific settings (model, permissions)
- [ ] Set up approved MCP servers if applicable
- [ ] Configure allowed/denied tools per team policy

### Verification
```bash
# Test that Claude Code is working
claude --version

# Test instruction file loading
claude "What conventions does this project follow?"
# Should reference rules from CLAUDE.md
```

---

## VS Code

### Extensions
- [ ] GitHub Copilot extension installed and signed in
- [ ] GitHub Copilot Chat extension installed
- [ ] Claude Code extension installed (if team uses Claude)
- [ ] Any team-specific extensions from the extensions list

### Settings
- [ ] AI-related `settings.json` configured per team standard:

```jsonc
// .vscode/settings.json (team shared)
{
  // Copilot settings
  "github.copilot.enable": {
    "*": true,
    "plaintext": false,
    "markdown": true
  },

  // File associations for instruction files
  "files.associations": {
    "copilot-instructions.md": "markdown",
    "CLAUDE.md": "markdown"
  },

  // Editor settings that improve AI interaction
  "editor.inlineSuggest.enabled": true,
  "editor.inlineSuggest.showToolbar": "onHover"
}
```

- [ ] Workspace settings committed to repo (`.vscode/settings.json`)
- [ ] Extension recommendations in `.vscode/extensions.json`

```jsonc
// .vscode/extensions.json
{
  "recommendations": [
    "GitHub.copilot",
    "GitHub.copilot-chat"
  ]
}
```

---

## Git Configuration

### Pre-commit Hooks
- [ ] Secret scanning hook installed (e.g., `gitleaks`, `detect-secrets`)
- [ ] Hook verified to catch test secrets

```bash
# Test that secret scanning catches secrets
echo 'API_KEY="sk-test-1234567890abcdef"' > test-secret.txt
git add test-secret.txt
git commit -m "test"
# Should be BLOCKED by pre-commit hook
rm test-secret.txt
```

### Ignore Files
- [ ] `.copilotignore` configured for sensitive files:

```gitignore
# .copilotignore
# Prevent AI tools from reading sensitive files
.env
.env.*
**/secrets/**
**/credentials/**
*.pem
*.key
config/production.yml
```

- [ ] `.gitignore` includes AI tool artifacts:

```gitignore
# AI tool artifacts
.claude/
.copilot-session/
*.ai-draft
```

### Git Configuration
- [ ] Commit template includes AI disclosure (optional):

```bash
git config commit.template .gitmessage
```

```
# .gitmessage
# Subject line

# Body

# AI Assistance: [none | copilot | claude | other]
# AI Scope: [autocomplete | generation | review | refactor]
```

---

## Team Standards

### Prompt Library
- [ ] Clone or access team prompt library
- [ ] Verify prompt files are accessible (`.github/prompts/` or shared location)
- [ ] Read through the top 10 most-used prompts
- [ ] Test at least 3 prompts to confirm they work

### Shared Skills
- [ ] Install team-shared custom skills (if applicable)
- [ ] Verify custom skills load correctly
- [ ] Test one shared skill to confirm setup

### Shared MCP Servers
- [ ] Configure team-approved MCP servers:

```jsonc
// MCP server configuration
{
  "mcpServers": {
    "team-docs": {
      "command": "npx",
      "args": ["@team/mcp-docs-server"],
      "env": {
        "DOCS_PATH": "./docs"
      }
    }
  }
}
```

- [ ] Verify each MCP server connects successfully
- [ ] Test a query against each server

---

## Final Verification

Run these test prompts to confirm everything is working together:

### Test 1: Instruction File Active
```
Prompt: "What are the coding conventions for this project?"
Expected: AI references rules from the instruction file
```

### Test 2: Code Generation Follows Standards
```
Prompt: "Create a function that validates an email address"
Expected: Output follows team naming conventions, style, and patterns
```

### Test 3: Security Guardrails Work
```
Prompt: "Create a function that connects to the database"
Expected: Output uses parameterized queries, no hardcoded credentials
```

### Test 4: Prompt Library Accessible
```
Action: Use a prompt from the team prompt library
Expected: Prompt loads and produces expected output format
```

---

## Printable Checklist

```
TEAM AI TOOL SETUP — Developer: __________ Date: __________

COPILOT CLI
□ Installed    □ Authenticated    □ Configured    □ Instruction file verified

CLAUDE CODE
□ Installed    □ Authenticated    □ Configured    □ Instruction file verified

VS CODE
□ Extensions   □ Settings         □ Workspace config

GIT
□ Pre-commit hooks    □ .copilotignore    □ .gitignore    □ Commit template

TEAM STANDARDS
□ Prompt library      □ Shared skills     □ MCP servers

VERIFICATION
□ Test 1: Instruction file    □ Test 2: Code standards
□ Test 3: Security guardrails □ Test 4: Prompt library

Buddy sign-off: _________________ Date: __________
```

---

## Troubleshooting Common Setup Issues

| Issue | Solution |
|---|---|
| Copilot not suggesting code | Check license, restart editor, verify extension is enabled |
| Instruction file not loaded | Check file location (root or `.github/`), verify file name |
| MCP server won't connect | Check network/proxy settings, verify server command |
| Pre-commit hook not running | Run `chmod +x .git/hooks/pre-commit` or reinstall hooks |
| AI output ignores conventions | Verify instruction file content, check for conflicting settings |
