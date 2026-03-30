# Prompt Routing

> Directing tasks to the right specialized prompt based on the task type.

---

## What Is Prompt Routing?

Prompt routing analyzes an incoming request and directs it to the most appropriate specialized prompt. Instead of one generic prompt handling everything, you have focused prompts for different task types.

```
User Request
     │
     ▼
┌──────────┐
│  Router   │ "What type of task is this?"
└──────────┘
     │
     ├──► Bug fix?      → Debug prompt (with trace instructions)
     ├──► New feature?   → Generation prompt (with patterns)
     ├──► Refactor?      → Refactoring prompt (with scope limits)
     ├──► Test?          → Test generation prompt (with framework)
     └──► Review?        → Review prompt (with severity focus)
```

---

## Routing with .prompt.md Files

GitHub Copilot's prompt file system is natural routing:

```
.github/prompts/
├── generate-endpoint.prompt.md    # New API endpoint
├── review-security.prompt.md      # Security review
├── debug-error.prompt.md          # Bug investigation
├── write-tests.prompt.md          # Test generation
└── refactor-function.prompt.md    # Refactoring tasks
```

Each file contains the specialized prompt for its task type. The developer picks the right one.

---

## Automated Routing (Programmatic)

```python
def route_task(user_request: str) -> str:
    # Step 1: Classify the task
    classification = call_model(
        f"""Classify this development task into exactly one category:
        - generation: Creating new code
        - debugging: Finding/fixing bugs
        - refactoring: Improving existing code
        - testing: Writing tests
        - review: Reviewing code quality
        
        Task: {user_request}
        Category:""",
        temperature=0
    )
    
    # Step 2: Load specialized prompt
    prompts = {
        "generation": load_prompt("generate.md"),
        "debugging": load_prompt("debug.md"),
        "refactoring": load_prompt("refactor.md"),
        "testing": load_prompt("test.md"),
        "review": load_prompt("review.md"),
    }
    
    return prompts[classification.strip()]
```

---

## Benefits of Routing

| Benefit | How |
|---------|-----|
| Higher quality | Specialized prompts outperform generic ones |
| Consistent output | Each prompt defines its own output format |
| Easier maintenance | Update one prompt, not a monolith |
| Testable | Test each prompt independently |
| Team scalability | Different team members own different prompts |

---

## Routing Patterns

### 1. Task-Type Routing
Route based on what the developer wants to do (see above).

### 2. File-Type Routing
Route based on the file being worked on:
```
.tsx files → React component prompt (with hooks, a11y focus)
.test.ts  → Test writing prompt (with Jest patterns)
.prisma   → Schema prompt (with migration awareness)
.md       → Documentation prompt (with format rules)
```

### 3. Complexity Routing
Route based on task complexity:
```
Simple (1 file, clear task)  → Direct generation prompt
Medium (2-5 files, some decisions) → Generation + review chain
Complex (many files, architecture) → Plan → generate → review → test chain
```
