# Edge Case Discovery

> Using AI to find untested edge cases — because AI excels at brainstorming the scenarios humans forget.

---

## Why AI Is Excellent at Edge Case Discovery

Humans tend to test for **expected behavior**. AI, having been trained on millions of bug reports and test suites, can systematically enumerate edge cases across categories that developers often overlook:

```
Human testing instinct:              AI edge case discovery:
"Does it work with normal input?"    "What about null?"
"Does the happy path pass?"          "What about empty string?"
                                     "What about MAX_INT + 1?"
                                     "What about unicode?"
                                     "What about concurrent access?"
                                     "What about leap years?"
                                     "What about timezone boundaries?"
```

---

## The Discovery Prompt Pattern

### Basic Pattern

```
Given this function, list all edge cases that could cause failures,
unexpected behavior, or incorrect results. Organize by category.

```typescript
{pasteFunction}
```

For each edge case:
1. Describe the scenario
2. Explain why it could fail
3. Provide a specific test input
```

### Enhanced Pattern (with categories)

```
Analyze this function for edge cases across ALL of these categories:

1. Null/undefined/missing values
2. Empty collections and strings
3. Boundary values (0, -1, MAX_INT, etc.)
4. Type coercion issues
5. Unicode and special characters
6. Concurrent/async race conditions
7. Large inputs (performance/memory)
8. Floating point precision
9. Timezone and locale issues
10. State-dependent behavior

```typescript
{pasteFunction}
```

For each discovered edge case, provide a failing test.
```

---

## Complete Example: Date Parser

### The Function

```typescript
function parseDate(input: string): {
  year: number;
  month: number;
  day: number;
  isValid: boolean;
} {
  const parts = input.split('-');
  const year = parseInt(parts[0], 10);
  const month = parseInt(parts[1], 10);
  const day = parseInt(parts[2], 10);

  const isValid =
    !isNaN(year) && !isNaN(month) && !isNaN(day) &&
    month >= 1 && month <= 12 &&
    day >= 1 && day <= 31;

  return { year, month, day, isValid };
}
```

### AI-Discovered Edge Cases

**Prompt:** "List all edge cases that could cause failures in this `parseDate` function."

**AI Response:**

| # | Category | Edge Case | Input | Expected Problem |
|---|----------|-----------|-------|-----------------|
| 1 | Empty input | Empty string | `""` | `parts` has one empty element, all NaN |
| 2 | Null/undefined | Null input | `null` | `split` throws TypeError |
| 3 | Missing parts | Only year | `"2024"` | `parts[1]` is undefined → NaN |
| 4 | Missing parts | Year and month only | `"2024-01"` | `parts[2]` is undefined → NaN |
| 5 | Boundary | Day 0 | `"2024-01-00"` | Incorrectly validates (day >= 1 catches this) |
| 6 | Boundary | Day 31 in Feb | `"2024-02-31"` | Validates as true but Feb never has 31 days |
| 7 | Boundary | Day 29 Feb non-leap | `"2023-02-29"` | Validates as true but 2023 isn't a leap year |
| 8 | Boundary | Month 0 | `"2024-00-15"` | Correctly invalid |
| 9 | Boundary | Month 13 | `"2024-13-15"` | Correctly invalid |
| 10 | Negative numbers | Negative year | `"-2024-01-15"` | Split produces `["", "2024", "01", "15"]` |
| 11 | Floating point | Decimal day | `"2024-01-15.5"` | `parseInt` ignores `.5`, passes as 15 |
| 12 | Whitespace | Leading spaces | `" 2024-01-15"` | `parseInt(" 2024")` returns 2024 (works by accident) |
| 13 | Unicode | Full-width digits | `"２０２４-０１-１５"` | `parseInt` returns NaN for full-width |
| 14 | Extra parts | Too many segments | `"2024-01-15-extra"` | Ignores extra parts (silent data loss) |
| 15 | Wrong separator | Slashes | `"2024/01/15"` | `split('-')` produces one element |
| 16 | Large values | Year 99999 | `"99999-01-01"` | Validates but may cause Date issues downstream |
| 17 | Hex strings | Hex values | `"0x10-0x01-0x0F"` | `parseInt` with radix 10 returns 0 for `0x10` |

### Generated Tests for Each Edge Case

```typescript
describe('parseDate - edge cases', () => {
  // 1. Empty input
  it('should handle empty string', () => {
    const result = parseDate('');
    expect(result.isValid).toBe(false);
  });

  // 3-4. Missing parts
  it('should handle input with only year', () => {
    const result = parseDate('2024');
    expect(result.isValid).toBe(false);
  });

  it('should handle input missing day', () => {
    const result = parseDate('2024-01');
    expect(result.isValid).toBe(false);
  });

  // 6. February 31st
  it('should reject February 31st', () => {
    const result = parseDate('2024-02-31');
    // BUG: This currently returns isValid: true
    // because the function only checks day <= 31
    expect(result.isValid).toBe(false);
  });

  // 7. Feb 29 on non-leap year
  it('should reject Feb 29 on a non-leap year', () => {
    const result = parseDate('2023-02-29');
    expect(result.isValid).toBe(false);
  });

  // 10. Negative year (tricky split behavior)
  it('should handle negative year input', () => {
    const result = parseDate('-2024-01-15');
    // split('-') produces ["", "2024", "01", "15"]
    // parts[0] is "" → NaN
    expect(result.isValid).toBe(false);
  });

  // 11. Decimal values
  it('should not silently truncate decimal days', () => {
    const result = parseDate('2024-01-15.5');
    // parseInt silently drops .5 — is this desired?
    expect(result.day).toBe(15); // Documents current behavior
  });

  // 14. Extra segments
  it('should reject input with extra segments', () => {
    const result = parseDate('2024-01-15-extra');
    // Current behavior: ignores extra parts
    expect(result.isValid).toBe(true); // Documents current behavior
  });

  // 15. Wrong separator
  it('should reject dates with slash separators', () => {
    const result = parseDate('2024/01/15');
    expect(result.isValid).toBe(false);
  });

  // 17. Hex strings
  it('should reject hex-formatted numbers', () => {
    const result = parseDate('0x10-0x01-0x0F');
    expect(result.isValid).toBe(false);
  });
});
```

---

## Common Edge Case Categories by Data Type

### Strings

| Category | Edge Cases |
|----------|-----------|
| **Empty/null** | `""`, `null`, `undefined` |
| **Whitespace** | `" "`, `"\t"`, `"\n"`, leading/trailing spaces |
| **Unicode** | Emoji (`"😀"`), CJK characters, RTL text, combining characters |
| **Special chars** | `"<script>"`, SQL injection (`"'; DROP TABLE--"`), null bytes (`"\0"`) |
| **Length** | Single char, max length, exceeds max length |
| **Encoding** | UTF-8 multibyte, Latin-1, Windows-1252 |

### Numbers

| Category | Edge Cases |
|----------|-----------|
| **Boundaries** | `0`, `-0`, `1`, `-1`, `MAX_SAFE_INTEGER`, `MIN_SAFE_INTEGER` |
| **Special values** | `NaN`, `Infinity`, `-Infinity` |
| **Precision** | `0.1 + 0.2`, very small decimals (`0.000001`) |
| **Types** | String numbers (`"42"`), boolean-as-number (`true → 1`) |

### Arrays/Collections

| Category | Edge Cases |
|----------|-----------|
| **Empty** | `[]`, empty after filter |
| **Single element** | `[1]` — off-by-one errors |
| **Duplicates** | `[1, 1, 1]` |
| **Order** | Already sorted, reverse sorted, random |
| **Large** | 10,000+ elements, nested deeply |
| **Types** | Mixed types (`[1, "two", null]`), sparse arrays |

### Dates/Times

| Category | Edge Cases |
|----------|-----------|
| **Boundaries** | Midnight, end of day (`23:59:59.999`) |
| **Calendar** | Leap years, Feb 29, Dec 31 → Jan 1 transition |
| **Timezones** | UTC, DST transitions, half-hour offsets (IST +5:30) |
| **Formats** | ISO 8601, Unix timestamps, locale-specific |
| **Special** | Epoch (1970-01-01), Y2K, dates before epoch |

### Objects/Maps

| Category | Edge Cases |
|----------|-----------|
| **Empty** | `{}` |
| **Missing keys** | Optional properties absent |
| **Extra keys** | Unexpected properties present |
| **Nested nulls** | `{ user: { address: null } }` |
| **Circular refs** | Object referencing itself |
| **Prototype** | Properties on prototype chain |

---

## Iterative Discovery Workflow

```
Step 1: Ask AI for edge cases → get initial list
Step 2: Generate tests for each edge case
Step 3: Run tests → discover actual bugs
Step 4: Ask AI: "Given these bugs, what other edge cases should I test?"
Step 5: Repeat until coverage is satisfactory
```

### Follow-Up Prompt

```
These edge case tests revealed that the parseDate function doesn't
validate days-per-month correctly. Given this bug, what additional
edge cases should I test? Consider:
- All months with different day counts (28, 29, 30, 31)
- Leap year rules (divisible by 4, except 100, except 400)
- Boundary dates for each month
```

---

## Prompt Patterns for Specific Discovery Goals

### Security Edge Cases

```
Analyze this function for security-related edge cases:
- Injection attacks (SQL, XSS, command injection)
- Path traversal
- Buffer overflow / large inputs
- Authentication bypass
- Information leakage in error messages
```

### Performance Edge Cases

```
Analyze this function for performance edge cases:
- Inputs that trigger worst-case time complexity
- Inputs that cause excessive memory allocation
- Inputs that trigger many database queries (N+1)
- Recursive inputs that could cause stack overflow
```

### Concurrency Edge Cases

```
Analyze this function for concurrency edge cases:
- Two users calling simultaneously with the same data
- Read-modify-write race conditions
- Partial failure in multi-step operations
- Stale cache reads
```
