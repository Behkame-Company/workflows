# Getting Started with AGENTS.md

> Your first AGENTS.md file in 5 minutes.

---

## What Is AGENTS.md?

AGENTS.md is a Markdown file that tells **any AI coding agent** how to work with your project. Think of it as a README for machines — while README.md onboards human developers, AGENTS.md onboards AI agents.

It's supported by all major tools:
- GitHub Copilot Coding Agent
- OpenAI Codex CLI
- Claude Code (as fallback)
- Cursor
- Google Gemini CLI / Jules
- Continue.dev, Zed, and more

---

## Step 1: Create the File

```bash
touch AGENTS.md
```

On Windows:
```powershell
New-Item -ItemType File -Path AGENTS.md
```

Place it at the **root** of your repository.

---

## Step 2: Add Your First Instructions

```markdown
# AGENTS.md

## Setup
- Install dependencies: `npm install`
- Start dev server: `npm run dev`
- Run tests: `npm run test`
- Lint code: `npm run lint`
- Build: `npm run build`

## Stack
TypeScript, React 18, Next.js 15, Tailwind CSS, Prisma, PostgreSQL.

## Conventions
- TypeScript strict mode; never use `any`
- Named exports only
- Validate API input with Zod

## Boundaries
- Never commit secrets or .env files
- Never modify files in `legacy/`
- Never add dependencies without noting in PR description

## Testing
- Framework: Vitest + React Testing Library
- Run: `npm run test`
- All new logic must have tests
```

💡 **Start with 20-30 lines.** You can always expand later.

---

## Step 3: Commit and Push

```bash
git add AGENTS.md
git commit -m "docs: add AGENTS.md for AI coding agents"
git push
```

AGENTS.md takes effect immediately — any AI agent that reads your repo will use these instructions.

---

## Step 4: Verify It Works

### With GitHub Copilot Coding Agent
1. Create a GitHub Issue describing a task
2. Assign it to Copilot
3. Review the resulting PR — check if it follows your conventions

### With Copilot Chat (VS Code)
1. Open VS Code with your project
2. Ask Copilot Chat: "What are the project conventions?"
3. Copilot should reference your AGENTS.md content

### With Claude Code
1. Run `claude` in your project directory
2. Ask: "What are the setup instructions for this project?"
3. Claude reads AGENTS.md automatically

### With Codex CLI
1. Run `codex` in your project
2. It automatically reads AGENTS.md for project context

---

## What to Read Next

| When you want to... | Read... |
|---------------------|---------|
| Understand the standard | [The Standard & AAIF](the-standard-and-aaif.md) |
| Learn the file structure | [Anatomy of AGENTS.md](anatomy-of-agents-md.md) |
| Use multiple AI tools | [Cross-Tool Matrix](../05-tool-compatibility/cross-tool-matrix.md) |
| See real examples | [Examples Gallery](../08-practical/examples-gallery.md) |
| Avoid common mistakes | [Common Mistakes](../04-anti-patterns/common-mistakes.md) |

---

## Quick Facts

| Property | Value |
|----------|-------|
| Filename | `AGENTS.md` (case-sensitive) |
| Location | Repository root (or per-directory for monorepos) |
| Format | Standard Markdown |
| Auto-loaded | Yes — by all supporting tools |
| Max recommended size | ~300 lines / 1,000 words |
| Governed by | Agentic AI Foundation (Linux Foundation) |

---

*Next: [The Standard & AAIF](the-standard-and-aaif.md) →*
