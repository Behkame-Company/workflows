# Vague Instructions

> Why "make it better" fails — and how to be specific instead.

---

## The Problem

Vague instructions force the model to guess your intent. Different guesses produce different (often wrong) results.

```
"Fix this code"

What the developer meant: Fix the null reference on line 42
What the model did: Renamed variables, added comments, reformatted, 
                    and restructured the entire function
```

---

## Hall of Shame: Vaguest Prompts

| Vague Prompt | Why It Fails |
|-------------|-------------|
| "Fix this" | Fix what? There might be nothing broken. |
| "Make it better" | Better in what dimension? Faster? Safer? Shorter? |
| "Clean up this code" | Clean how? What's dirty about it? |
| "Add some tests" | How many? For what? What framework? |
| "Improve the error handling" | Which errors? What's the improvement? |
| "Optimize this" | For what? Speed? Memory? Readability? |
| "Review this PR" | For bugs? Style? Security? Architecture? |

---

## The Fix: Specificity Dimensions

For every vague prompt, add specifics in these dimensions:

### What (the action)
```
❌ "Fix this"
✅ "Fix the null reference error when user.profile is undefined"
```

### Where (the scope)
```
❌ "Update the tests"
✅ "Add a test for the empty array case in calculateTotal()"
```

### How (the approach)
```
❌ "Make it faster"
✅ "Replace the nested loops with a Map-based lookup to reduce 
    from O(n²) to O(n)"
```

### Why (the motivation)
```
❌ "Add error handling"
✅ "Add error handling because unhandled database timeouts are causing 
    500 errors in production (see error log snippet below)"
```

### Done when (success criteria)
```
❌ "Implement pagination"
✅ "Done when GET /users?page=2&limit=10 returns the correct 10 items 
    and the response includes total count"
```

---

## Vagueness Detection Checklist

Before sending a prompt, check for these words:

| Word/Phrase | Replace With |
|-------------|-------------|
| "better" | The specific dimension of improvement |
| "some" | A specific number |
| "good" | Specific quality criteria |
| "appropriate" | The exact value or approach |
| "properly" | The specific correct behavior |
| "etc." | List all the things you mean |
| "things like" | The actual things |

---

## The Golden Rule

> If a colleague with minimal context on the task would be confused by your prompt, the AI will be too.
