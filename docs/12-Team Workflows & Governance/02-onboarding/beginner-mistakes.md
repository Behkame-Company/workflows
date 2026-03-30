# Common Beginner Mistakes

> What new AI tool users get wrong — and how to fix it. Recognizing these patterns early saves weeks of frustration and prevents quality issues from entering your codebase.

---

## Mistake 1: Accepting All AI Suggestions Without Review

### What happens
The developer treats AI output as correct by default, accepting suggestions with Tab/Enter without reading them. Code enters the codebase with subtle bugs, incorrect assumptions, or security issues.

### How to recognize it
- PRs with AI-generated code that the developer can't explain
- Inconsistent patterns within the same file (some AI-generated, some not)
- Bugs in code that "looked right" but has edge case failures

### How to fix it
✅ Read every line of AI-generated code before accepting
✅ Ask yourself: "Would I approve this in a code review?"
✅ Run the code through tests before committing
❌ Don't Tab-accept multi-line suggestions without reading them

---

## Mistake 2: Not Providing Enough Context

### What happens
The developer uses vague prompts like "write a login function" without specifying the framework, patterns, error handling approach, or project conventions. The AI generates generic code that doesn't fit the project.

### How to recognize it
- AI output uses different patterns than the rest of the codebase
- Generated code requires significant rework to integrate
- Developer says "the AI doesn't understand our project"

### How to fix it
✅ Include relevant context in your prompts
✅ Reference existing code: "Follow the pattern in src/auth/login.ts"
✅ Specify constraints: "Use zod validation, return Result<T, Error>"
❌ Don't use one-line prompts for complex tasks

```
❌ "Write a user service"
✅ "Create a UserService class following our repository pattern 
   (see src/services/ProductService.ts). Use TypeScript, return 
   Result<T, AppError>, validate with zod schemas, include JSDoc."
```

---

## Mistake 3: Ignoring AI-Generated Test Failures

### What happens
AI generates tests that fail, and the developer either deletes them or modifies the test assertions to match the (wrong) implementation instead of fixing the implementation.

### How to recognize it
- Tests that assert implementation details instead of behavior
- Test descriptions that don't match what the test actually checks
- Tests rewritten to match buggy behavior

### How to fix it
✅ If AI-generated tests fail, investigate whether the test or the implementation is wrong
✅ Tests define expected behavior — the implementation should conform
✅ Use failures as learning signals about edge cases
❌ Don't change test expectations to match incorrect behavior

---

## Mistake 4: Using AI for Everything

### What happens
Developer uses AI for trivial tasks where direct typing is faster, or for tasks where AI consistently produces poor results. Over-reliance on AI actually slows them down.

### How to recognize it
- Prompting AI to rename a variable or add a single line
- Waiting for AI responses on things the developer already knows
- Using AI for complex architectural decisions that need human reasoning

### How to fix it
✅ Use AI where it adds value: boilerplate, patterns, tests, documentation
✅ Type directly for small, obvious changes
✅ Use human judgment for architecture, design, and complex logic
❌ Don't prompt-engineer when you could just type the code

**AI value sweet spot:**
```
Low value:   Renaming a variable, adding an import, fixing a typo
Medium value: Writing a single function, adding error handling
High value:  Generating test suites, creating boilerplate, writing docs
Low value:   Complex architectural decisions, subtle algorithmic logic
```

---

## Mistake 5: Not Reading the Generated Code

### What happens
Developer scans AI output for the general shape but doesn't read the actual logic. This is different from Mistake 1 — here the developer thinks they reviewed it but actually only skimmed.

### How to recognize it
- "I reviewed it" but can't answer questions about specific logic choices
- Code uses libraries or functions that don't exist in the project
- Subtle type mismatches or off-by-one errors in AI output

### How to fix it
✅ Read AI output line by line, especially: conditions, loops, error paths
✅ Verify imports and dependencies actually exist
✅ Check for hallucinated APIs (functions that look right but don't exist)
❌ Don't rely on "it looks about right"

---

## Mistake 6: Over-Prompting

### What happens
Developer writes huge, multi-paragraph prompts with conflicting or excessive instructions. The AI gets confused, and output quality drops.

### How to recognize it
- Prompts that are longer than the expected output
- AI output ignores parts of the prompt
- Developer keeps adding "and also..." to prompts

### How to fix it
✅ Break complex tasks into smaller prompts
✅ Be specific but concise — prioritize the most important requirements
✅ Use instruction files for project-wide rules, prompts for task-specific guidance
❌ Don't put everything in one massive prompt

```
❌ "Write a function that validates emails and also make sure 
it handles international domains and check MX records and 
return detailed errors and also make it async and use our 
custom error type and add logging and make it testable and..."

✅ Prompt 1: "Create an async email validation function using 
our AppError type (see src/types.ts)"
✅ Prompt 2: "Add MX record checking to the validation function"
✅ Prompt 3: "Add structured logging following our Logger pattern"
```

---

## Mistake 7: Skipping Team Instruction Files

### What happens
Developer doesn't read or understand the team instruction files, so they don't configure their tools properly. Their AI output is inconsistent with the rest of the team.

### How to recognize it
- Output uses different naming conventions than the team
- Generated code patterns don't match existing codebase
- Developer says "AI never follows our conventions"

### How to fix it
✅ Read the instruction file on day 1 of onboarding
✅ Verify your tool is loading the instruction file (test it)
✅ When output seems wrong, check instruction file coverage first
❌ Don't assume AI knows your conventions without being told

---

## Mistake 8: Not Using Version Control as Safety Net

### What happens
Developer makes AI-generated changes directly to working code without committing first, then can't undo when the AI changes break something.

### How to recognize it
- Developer asks "how do I undo what AI did?"
- Large uncommitted diffs mixing AI changes with manual work
- Fear of trying AI suggestions because of difficulty reverting

### How to fix it
✅ Commit before starting AI-assisted changes
✅ Use branches for experimental AI-generated code
✅ Commit working AI output incrementally
❌ Don't let AI modify 20 files without any committed checkpoints

```bash
# Safe AI experimentation pattern
git checkout -b feature/ai-experiment
# Let AI generate code
git add -p  # Review and stage selectively
git commit -m "AI: generated user validation (needs review)"
# Continue refining...
```

---

## Mistake 9: Sharing Secrets in Prompts

### What happens
Developer pastes code containing API keys, database credentials, or internal URLs into AI prompts. This data may be sent to external servers and potentially used for model training.

### How to recognize it
- Hard to detect after the fact
- Environment variables or secrets found in AI chat history
- Developer copies entire config files into prompts

### How to fix it
✅ Always redact secrets before sharing code with AI
✅ Use `.copilotignore` to prevent AI from reading sensitive files
✅ Use placeholder values: `API_KEY="your-key-here"`
❌ Never paste `.env` files, credentials, or tokens into AI prompts

```
❌ "Why isn't this working?"
   DATABASE_URL="postgres://admin:p4ssw0rd@prod-db.internal:5432/app"

✅ "Why isn't this database connection working?"
   DATABASE_URL="postgres://user:password@hostname:5432/dbname"
```

---

## Mistake 10: Expecting Perfection

### What happens
Developer gets frustrated when AI output isn't perfect, concluding that AI tools "don't work." They give up on AI-assisted development entirely.

### How to recognize it
- "I tried AI once and it got everything wrong"
- Developer reverts to fully manual coding
- Unrealistic expectations followed by complete rejection

### How to fix it
✅ Expect AI to get you 60-80% of the way, faster
✅ Plan to review and refine — it's a collaboration, not delegation
✅ Focus on time savings, not perfection
❌ Don't judge AI tools by their worst output

**Realistic expectation:**
```
Without AI: 2 hours to write, test, and document a feature
With AI:    30 min to generate + 45 min to review, refine, test = 1h 15min
Savings:    ~35-40% time reduction with equal or better quality
```

---

## Self-Assessment Quiz

Score yourself honestly (0 = Never, 1 = Sometimes, 2 = Often):

| # | Question | Score |
|---|---|---|
| 1 | I accept AI suggestions without reading every line | |
| 2 | I write prompts without specifying project context | |
| 3 | I modify tests to match AI output instead of fixing the code | |
| 4 | I use AI for tasks faster done by hand | |
| 5 | I skim AI output instead of reading thoroughly | |
| 6 | I write prompts longer than a paragraph | |
| 7 | I forget to check if the instruction file is loaded | |
| 8 | I make AI changes without committing first | |
| 9 | I share sensitive data in prompts | |
| 10 | I give up when AI output isn't perfect | |

### Scoring
- **0-5:** Good habits — keep it up
- **6-10:** Some areas to improve — focus on your highest scores
- **11-15:** Significant risk — review this guide and practice with a buddy
- **16-20:** Stop and retrain — these habits will cause real issues
