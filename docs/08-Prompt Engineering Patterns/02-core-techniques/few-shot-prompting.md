# Few-Shot Prompting

> Teaching the model through in-context examples — the most reliable way to control output.

---

## What Is Few-Shot?

Few-shot prompting provides examples of input/output pairs in the prompt, letting the model learn the pattern in-context before applying it to new input.

```
Classify the sentiment:

Text: "The deployment was smooth and fast"
Sentiment: Positive

Text: "The build keeps failing on CI"
Sentiment: Negative

Text: "We updated the README with the new API endpoints"
Sentiment: Neutral

Text: "The refactoring broke three existing tests"
Sentiment:
```

The model sees the pattern and responds: **Negative**

---

## Why Examples Beat Descriptions

| Approach | Prompt | Result |
|----------|--------|--------|
| Description only | "Convert to our API format" | Model guesses your format |
| Description + example | "Convert to our API format: <example>...</example>" | Model matches your format exactly |

> Examples are worth more than descriptions. 3 examples consistently outperform a detailed paragraph of instructions.

---

## How Many Examples?

| Count | Name | When to Use |
|-------|------|-------------|
| 0 | Zero-shot | Simple, well-known tasks |
| 1 | One-shot | Simple format demonstration |
| 3-5 | Few-shot | **Sweet spot for most tasks** |
| 5-10 | Many-shot | Complex patterns with edge cases |
| 10+ | Many-shot | Classification with many categories |

**Diminishing returns**: Going from 0→3 examples is huge. Going from 5→10 is marginal for most tasks.

---

## Crafting Effective Examples

### 1. Diversity Over Quantity

❌ Three examples that all show the same pattern:
```
Input: function add(a, b) { return a + b }
Output: function add(a: number, b: number): number { return a + b }

Input: function subtract(a, b) { return a - b }
Output: function subtract(a: number, b: number): number { return a - b }

Input: function multiply(a, b) { return a * b }
Output: function multiply(a: number, b: number): number { return a * b }
```

✅ Three examples showing different patterns:
```
Input: function add(a, b) { return a + b }
Output: function add(a: number, b: number): number { return a + b }

Input: const getName = (user) => user.name || 'Anonymous'
Output: const getName = (user: User): string => user.name || 'Anonymous'

Input: async function fetchData(url) { const res = await fetch(url); return res.json() }
Output: async function fetchData(url: string): Promise<unknown> { const res = await fetch(url); return res.json() }
```

### 2. Include Edge Cases

Show the model how to handle unusual inputs:
```
Input: "" (empty string)
Output: "invalid_input"

Input: "Hello, World! 123"
Output: "hello_world_123"

Input: "---already-kebab---"  
Output: "already-kebab"
```

### 3. Use XML Tags for Structure

```xml
<examples>
<example>
  <input>getUserProfile(userId)</input>
  <output>
  /**
   * Retrieves a user's profile by their unique identifier.
   * @param userId - The unique identifier of the user
   * @returns The user's profile data
   * @throws {NotFoundError} If no user exists with the given ID
   */
  </output>
</example>
</examples>
```

Anthropic recommends `<example>` tags to separate examples from instructions.

---

## Few-Shot for Code Tasks

### Format: Before → After

```
Convert class components to functional components with hooks:

Before:
class Counter extends React.Component {
  state = { count: 0 }
  increment = () => this.setState({ count: this.state.count + 1 })
  render() { return <button onClick={this.increment}>{this.state.count}</button> }
}

After:
function Counter() {
  const [count, setCount] = useState(0)
  const increment = () => setCount(c => c + 1)
  return <button onClick={increment}>{count}</button>
}

Now convert this class component:
[actual input]
```

---

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| All examples look the same | Model only learns one pattern | Diversify examples |
| Examples don't match real inputs | Model can't transfer the pattern | Use realistic data |
| Too many examples | Wastes context budget | 3-5 is usually enough |
| Examples contradict instructions | Model gets confused | Ensure consistency |
| No edge cases shown | Model fails on unusual inputs | Include 1-2 edge cases |
