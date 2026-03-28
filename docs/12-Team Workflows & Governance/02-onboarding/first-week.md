# First Week with AI Tools

> A structured, day-by-day learning path for your first week with AI development tools. Each day builds on the previous one, progressing from basic setup to independent AI-assisted development.

---

## Key Lessons for Week 1

Before diving in, internalize these principles — they'll save you hours of frustration:

1. **AI is probabilistic** — the same prompt can produce different results each time
2. **Always verify** — never merge AI-generated code you haven't reviewed and tested
3. **Tests are your safety net** — AI code that passes tests is trustworthy; AI code without tests is a gamble
4. **Context matters enormously** — a well-configured instruction file transforms AI output quality

---

## Day 1: Install, Configure, Explore

**Theme:** Get everything working and have your first AI conversations.

### Morning: Setup

Follow the [Tool Setup Checklist](tool-setup-checklist.md) completely. Don't skip the verification steps.

### Afternoon: Explore Autocomplete

Open a project file and start typing. Observe how AI autocomplete works:

**Exercise 1.1: Autocomplete Exploration**
```
1. Create a new file (e.g., utils.ts or utils.py)
2. Type a function signature: function validateEmail(email: string)
3. Observe the AI autocomplete suggestion
4. Accept it with Tab
5. Review what was generated — does it make sense?
6. Try again with a different function name
```

**Exercise 1.2: Simple Code Generation**
```
Prompt: "Create a function that converts a temperature 
from Celsius to Fahrenheit and vice versa"

Evaluate:
- Does it handle both directions?
- Are there edge cases (absolute zero)?
- Does it follow your team's naming conventions?
```

**Exercise 1.3: Ask a Question**
```
Prompt: "Explain how error handling works in this project"

Evaluate:
- Does the AI reference your project's actual patterns?
- Is the instruction file influencing the response?
```

### Day 1 Checklist
- [ ] All tools installed and verified
- [ ] Tried autocomplete on 5+ functions
- [ ] Generated 2+ complete functions via prompts
- [ ] Asked AI a question about the project
- [ ] Read the team instruction file completely

---

## Day 2: Instruction Files and Context

**Theme:** Understand how context shapes AI output and why instruction files matter.

### Morning: The Context Experiment

**Exercise 2.1: With vs. Without Context**
```
Step 1: Create a temporary file OUTSIDE your project directory
Step 2: Prompt: "Create a REST API endpoint for user registration"
Step 3: Note the output style, patterns, error handling
Step 4: Now do the same prompt INSIDE your project (with instruction file)
Step 5: Compare the two outputs

What differences do you see?
- Naming conventions?
- Error handling patterns?
- Response format?
- Validation approach?
```

**Exercise 2.2: Instruction File Deep Dive**
```
1. Open your team's copilot-instructions.md
2. For each rule, test whether AI follows it:
   - If the rule says "use camelCase" → generate code and check
   - If the rule says "use Result pattern" → generate error handling and check
   - If the rule says "never use eval" → ask AI to solve a problem that could use eval
3. Note any rules that aren't being followed
```

### Afternoon: Prompt Patterns

**Exercise 2.3: Prompt Library Practice**
```
Pick 5 prompts from your team's prompt library:
1. Run each prompt as-is
2. Evaluate the output quality
3. Try modifying one prompt to be more specific
4. Compare the original vs. modified output
```

**Exercise 2.4: Context Window Management**
```
1. Start a fresh AI conversation
2. Ask it to implement a feature
3. Notice quality of first response
4. Continue the conversation with refinements
5. After 10+ messages, notice if quality changes
6. Start fresh and give better initial context
7. Compare the results
```

### Day 2 Checklist
- [ ] Compared AI output with/without instruction file
- [ ] Tested each instruction file rule
- [ ] Used 5 prompts from team library
- [ ] Experimented with prompt modification
- [ ] Understand how context affects output

---

## Day 3: Pair Programming with AI

**Theme:** Learn the collaboration pattern — AI generates, you review and refine.

### Morning: Shadow Your Buddy

Watch your onboarding buddy work on a real task with AI tools. Notice:
- How they structure prompts
- How they evaluate suggestions
- What they accept vs. reject vs. modify
- How they break complex tasks into steps

### Afternoon: You Drive

**Exercise 3.1: The Review-and-Refine Cycle**
```
Task: Implement a small feature (buddy provides the task)

Step 1: Write a clear prompt describing what you need
Step 2: Review the AI output — don't accept yet
Step 3: Identify what's good, what needs changes
Step 4: Refine your prompt or manually edit the output
Step 5: Repeat until satisfied
Step 6: Run tests to verify
```

**Exercise 3.2: When AI Helps vs. Hinders**

Try using AI for each of these tasks and rate helpfulness (1-5):

| Task | Helpfulness | Notes |
|---|---|---|
| Write a utility function | ? | |
| Generate test cases | ? | |
| Write a complex algorithm | ? | |
| Create boilerplate code | ? | |
| Debug a subtle issue | ? | |
| Write documentation | ? | |
| Refactor existing code | ? | |
| Design an architecture | ? | |

### Day 3 Checklist
- [ ] Shadowed buddy for at least 1 hour
- [ ] Completed a task with AI in "driver" seat
- [ ] Practiced the review-and-refine cycle
- [ ] Identified where AI helps most and least

---

## Day 4: Testing with AI

**Theme:** AI-generated tests are powerful but require critical evaluation.

### Morning: TDD with AI

**Exercise 4.1: Test-First Development**
```
Step 1: Write a function specification (what it should do)
Step 2: Prompt AI: "Write unit tests for a function that [spec]"
Step 3: Review generated tests — are they meaningful?
Step 4: Now prompt AI: "Implement the function to pass these tests"
Step 5: Run the tests
Step 6: Evaluate: Did AI write tests that catch real issues?
```

**Exercise 4.2: Test Quality Assessment**
```
Take an existing function in your project.

Step 1: Prompt: "Generate comprehensive tests for [function]"
Step 2: Evaluate each test:
  ✅ Does it test a meaningful behavior?
  ✅ Does it cover an edge case?
  ✅ Would this test catch a real bug?
  ❌ Is it just testing that the function returns what it returns? (tautological)
  ❌ Is it testing implementation details rather than behavior?
  ❌ Is it missing important edge cases?

Step 3: Add the missing edge cases manually
```

### Afternoon: Verification Patterns

**Exercise 4.3: Boundary Testing**
```
Prompt: "Generate tests for [function] focusing on:
- Null and undefined inputs
- Empty strings and arrays  
- Maximum and minimum values
- Concurrent access (if applicable)
- Malicious input (SQL injection, XSS)"

Evaluate: Does AI generate meaningful boundary tests
or just placeholders?
```

### Day 4 Checklist
- [ ] Completed a TDD cycle with AI
- [ ] Evaluated AI-generated test quality critically
- [ ] Identified gaps in AI-generated test coverage
- [ ] Added manual tests for missed edge cases
- [ ] Understand the difference between "has tests" and "well tested"

---

## Day 5: Independent Task and Retrospective

**Theme:** Put it all together with an independent task, then reflect on the week.

### Morning: Independent Task

Your buddy assigns a small, well-defined task from the backlog.

**Rules:**
1. Use AI tools as you've learned this week
2. Follow the team review process
3. Write tests (AI-assisted is fine)
4. Submit a PR with appropriate AI disclosure
5. Focus on process quality, not speed

### Afternoon: Review and Retrospective

**PR Review with Buddy:**
- Walk through the PR together
- Discuss AI-generated portions
- Identify what could be improved
- Get feedback on process, not just code

**Week 1 Retrospective Questions:**
```markdown
## Week 1 Retrospective

### Most valuable thing I learned:

### Biggest surprise about AI tools:

### Where AI helped most:

### Where AI was frustrating or unhelpful:

### Team standards I want to understand better:

### What I'd tell the next new developer:

### My confidence level (1-5): 

### Questions I still have:
```

### Day 5 Checklist
- [ ] Completed independent task with AI assistance
- [ ] Submitted PR following team process
- [ ] Received and discussed PR feedback
- [ ] Completed retrospective
- [ ] Have a plan for continued learning in Week 2

---

## Daily Exercise Summary

| Day | Focus | Key Exercise | Deliverable |
|---|---|---|---|
| 1 | Setup & Explore | Autocomplete + simple generation | Tools working, 2+ functions generated |
| 2 | Context & Prompts | With/without instruction file comparison | Understanding of context impact |
| 3 | Pair Programming | Review-and-refine cycle | Feature implemented with buddy |
| 4 | Testing | TDD with AI, test quality assessment | Test suite with manual edge cases |
| 5 | Independence | Solo task + PR | Merged PR, completed retrospective |

## Next Steps

- [Learn common beginner mistakes to avoid →](beginner-mistakes.md)
- [Review the full onboarding program →](developer-onboarding.md)
- [Explore the team prompt library →](../03-standards-and-conventions/prompt-library.md)
- [Understand code quality standards →](../03-standards-and-conventions/code-quality-standards.md)
