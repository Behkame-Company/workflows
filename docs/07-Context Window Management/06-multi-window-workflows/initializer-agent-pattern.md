# Initializer Agent Pattern

> How the first context window sets up everything for success in subsequent windows.

---

## Purpose

The initializer agent runs in the very first context window. Its job is NOT to build features — it's to create the scaffolding that makes all future windows productive.

---

## What the Initializer Creates

### 1. Feature List (feature_list.json)
A comprehensive list of all features needed, all initially marked as failing:

```json
[
  {
    "category": "authentication",
    "description": "User can register with email and password",
    "steps": [
      "Navigate to registration page",
      "Enter valid email and password",
      "Submit form",
      "Verify account created"
    ],
    "passes": false
  },
  {
    "category": "authentication",
    "description": "User can log in with valid credentials",
    "steps": [
      "Navigate to login page",
      "Enter valid credentials",
      "Verify redirect to dashboard"
    ],
    "passes": false
  }
]
```

**Why JSON**: The model is less likely to inappropriately modify or delete JSON entries compared to Markdown checkboxes.

### 2. Init Script (init.sh)
```bash
#!/bin/bash
# Start the development server
cd /project
npm install
npm run dev &
sleep 5
echo "Server running on http://localhost:3000"
```

This prevents each future window from having to figure out how to run the app.

### 3. Progress File (progress.txt)
The initializer creates the first progress entry noting scaffolding setup, feature count, and next steps.

### 4. Initial Git Commit
A clean initial commit captures the full scaffolding as a recoverable baseline.

For detailed descriptions of these state artifacts (feature_list.json, progress.txt, init.sh) and how they interrelate, see [State Handoff Strategies](state-handoff-strategies.md).

---

## Initializer Prompt Structure

```markdown
You are setting up a new project. Your job is NOT to implement features.
Your job is to create the foundation for future work sessions.

## Steps
1. Analyze the user's requirements
2. Create a comprehensive feature_list.json with ALL required features
   - Each feature should have clear test steps
   - Mark all features as "passes": false
   - Organize by category
3. Set up the project structure
4. Create init.sh to start the development server
5. Create progress.txt with session notes
6. Make an initial git commit

## Critical Rules
- Do NOT implement any features
- Do NOT mark any features as passing
- Be comprehensive in the feature list — list over 50 features
- Create init.sh that works without manual intervention
```

---

## Key Takeaways

1. **First window is setup, not implementation** — Scaffolding only
2. **Feature list is the contract** — Comprehensive, all failing
3. **Init script saves future time** — Don't rediscover how to run the app
4. **Progress file is the handoff** — What was done, what's next
5. **Git commit preserves the baseline** — Clean starting point
