# Debugging Prompts

> Prompts for finding and fixing bugs effectively with AI assistance.

---

## The Debugging Prompt Formula

```
[Symptom] + [Expected vs Actual] + [Context] + [What You've Tried]
```

---

## Prompt Patterns by Bug Type

### Runtime Error
```
This function throws "TypeError: Cannot read property 'name' of undefined"
when called with a user who has no profile.

Function: getUserDisplayName() in src/utils/user.ts
Input that triggers: { id: 1, email: "test@test.com" }  (no profile field)
Expected: Return "Anonymous" for users without profiles
Actual: Throws TypeError

Fix the function to handle missing profile gracefully.
```

### Logic Error
```
The calculateDiscount() function returns wrong values.

Input: { price: 100, quantity: 3, couponCode: "SAVE20" }
Expected: 60 (20% off 100 * 3 = 300, so 300 - 60 = 240, discount is 60)
Actual: 20 (seems to apply 20% to price only, ignoring quantity)

Walk through the function line by line with this input 
to find where the logic diverges from expected behavior.
```

### Test Failure
```
This test is failing:

test('should paginate results', () => {
  const result = paginate(items, { page: 2, pageSize: 10 });
  expect(result.data).toHaveLength(10);
  expect(result.total).toBe(25);
});

Error: Expected length 10, received 0

The function works for page 1 but returns empty for page 2.
Trace through the pagination logic for page=2, pageSize=10, total=25 items.
```

### Performance Issue
```
This database query takes 8 seconds for 10K rows.
Expected: < 500ms

Query: [the SQL or ORM query]
Table sizes: users (10K rows), orders (500K rows), products (5K rows)

Analyze:
1. Is there a missing index?
2. Is there an N+1 query problem?
3. Are we fetching more data than needed?
4. Can this be optimized with a different query strategy?
```

---

## The Execution Trace Technique

The most powerful debugging prompt asks the model to trace execution:

```
Trace through this function step by step with input: [specific input]

At each line, show:
- Current variable values
- Condition evaluation results (true/false)
- Function call arguments and returns

Mark the EXACT line where actual behavior diverges from expected.
```

---

## Information to Always Include

| Info | Why |
|------|-----|
| Error message (exact) | Pinpoints the failure type |
| Input that triggers it | Enables reproduction |
| Expected behavior | Defines "correct" |
| Actual behavior | Shows the gap |
| Relevant code | The model needs to see it |
| What you've tried | Avoids suggesting failed fixes |

---

## Debugging Anti-Patterns

| Anti-Pattern | Problem |
|-------------|---------|
| "This doesn't work" | No symptoms to analyze |
| Pasting entire codebase | Model can't find the relevant part |
| "Fix all the bugs" | Unfocused, model invents problems |
| Not including error messages | Forces model to guess |
| Not sharing what you tried | Model suggests your failed attempts |

---

## Next Steps

- 🔗 [Test Generation Prompts](test-generation-prompts.md) — Write tests to prevent regressions
- 🔗 [Chain-of-Thought](../02-core-techniques/chain-of-thought.md) — Step-by-step reasoning for debugging
