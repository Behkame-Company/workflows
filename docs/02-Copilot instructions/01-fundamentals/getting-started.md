# Getting Started with Copilot Instructions

> Your first `copilot-instructions.md` in 10 minutes.

---

## What You'll Learn

By the end of this guide, you'll have a working instruction file that improves Copilot's output for your project.

---

## Step 1: Create the File

```bash
mkdir -p .github
touch .github/copilot-instructions.md
```

On Windows:
```powershell
New-Item -ItemType Directory -Path .github -Force
New-Item -ItemType File -Path .github\copilot-instructions.md
```

---

## Step 2: Add Your First Instructions

Open `.github/copilot-instructions.md` and paste this starter:

```markdown
# Project Instructions

## Project
This is a [describe your project] built with [your stack].

## Commands
- `npm install` — Install dependencies
- `npm run dev` — Start dev server
- `npm run test` — Run tests
- `npm run lint` — Lint code

## Conventions
- [Your most important coding rule]
- [Another important rule]
- [One more]

## Do Not
- Never commit secrets or API keys
- Never use console.log in production code
```

💡 **Start with just 10-20 lines.** You can always add more later.

---

## Step 3: Verify It Works

1. Open your project in VS Code
2. Open Copilot Chat (Ctrl+Shift+I or Cmd+Shift+I)
3. Ask: "What are the project conventions?"
4. Expand the **References** section at the top of the response
5. Confirm `copilot-instructions.md` appears in the list

If it appears — congratulations! Copilot is reading your instructions.

---

## Step 4: Test Behavioral Impact

Ask Copilot to do something your instructions address:

**If your instructions say "Use TypeScript strict mode; no `any`":**
- Ask: "Create a utility function to parse dates"
- ✅ **Good**: Copilot produces typed function with explicit return types
- ❌ **Without instructions**: Copilot might use `any` or loose types

**If your instructions say "Always validate input with Zod":**
- Ask: "Create an API endpoint for user registration"
- ✅ **Good**: Copilot includes Zod schema validation
- ❌ **Without instructions**: Copilot might skip validation entirely

---

## What Happens Next

| When you're ready for... | Read... |
|--------------------------|---------|
| Understanding the file format | [Anatomy of Instructions](anatomy-of-instructions.md) |
| How Copilot decides what to apply | [Instruction Hierarchy](instruction-hierarchy.md) |
| Making instructions more effective | [Writing Effective Instructions](../03-best-practices/writing-effective-instructions.md) |
| Domain-specific rules | [Domain Guides](../06-domain-guides/) |
| Avoiding common mistakes | [Common Mistakes](../04-anti-patterns/common-mistakes.md) |

---

## Quick Reference: File Locations

| File | Path | Purpose |
|------|------|---------|
| Repository-wide | `.github/copilot-instructions.md` | Rules for the whole repo |
| Path-specific | `.github/instructions/NAME.instructions.md` | Rules for matching files |
| Prompt files | `.github/prompts/NAME.prompt.md` | Reusable task workflows |
| Cross-tool | `AGENTS.md` (repo root) | Universal AI agent instructions |
