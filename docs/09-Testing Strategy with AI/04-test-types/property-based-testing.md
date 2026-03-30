# Property-Based Testing

> Generative testing with AI — instead of writing individual test cases, define properties that must always hold and let the framework generate thousands of random inputs to find violations.

## What Is Property-Based Testing?

Traditional example-based tests check specific inputs:

```typescript
expect(sort([3, 1, 2])).toEqual([1, 2, 3]);
```

Property-based tests check that **invariants hold for all possible inputs**:

```typescript
// For ANY array of numbers, sorting should produce a result where
// every element is ≤ the next element
fc.assert(
  fc.property(fc.array(fc.integer()), (arr) => {
    const sorted = sort(arr);
    return sorted.every((val, i) => i === 0 || sorted[i - 1] <= val);
  })
);
```

The framework generates hundreds of random arrays — including empty arrays, single-element arrays, arrays with duplicates, negative numbers, and `Number.MAX_SAFE_INTEGER` — and checks that the property holds for all of them.

### Why This Matters

Property-based testing finds edge cases that humans (and AI) miss. You don't need to think of every edge case — you just define what "correct" means and let the computer find counterexamples.

## AI + Property-Based Testing

This combination is powerful for two reasons:

1. **AI can identify properties to test** — Given a function, AI can reason about what invariants should hold
2. **Property-based testing catches AI blind spots** — AI-generated code often works for "normal" inputs but fails on edge cases that random generation discovers

### Prompting AI for Properties

```
I have this function:

function parseEmail(input: string): { local: string; domain: string } | null

Identify 5 properties that should always hold for this function,
then write fast-check property tests for each one.

Think about: round-trip properties, invariants, idempotency,
and relationships between inputs and outputs.
```

AI might identify:

1. **Round-trip**: If `parseEmail(s)` returns a result, then `result.local + '@' + result.domain === s`
2. **Null safety**: `parseEmail(null)`, `parseEmail('')`, `parseEmail(undefined)` should not throw
3. **Invariant**: A valid result always has non-empty `local` and `domain`
4. **No-@ rule**: If input contains no `@`, result must be `null`
5. **Idempotency**: Parsing the reconstructed email gives the same result

## Libraries

| Language | Library | Install |
|----------|---------|---------|
| JavaScript/TypeScript | [fast-check](https://github.com/dubzzz/fast-check) | `npm install --save-dev fast-check` |
| Python | [Hypothesis](https://hypothesis.readthedocs.io/) | `pip install hypothesis` |
| Rust | [proptest](https://github.com/proptest-rs/proptest) | `proptest = "1.0"` in `Cargo.toml` |
| Java | [jqwik](https://jqwik.net/) | Maven/Gradle dependency |
| Haskell | [QuickCheck](https://hackage.haskell.org/package/QuickCheck) | The original (1999) |

## fast-check Examples (TypeScript)

### Example 1: Email Validation Property

```typescript
import fc from 'fast-check';
import { validateEmail } from './validators';

describe('validateEmail properties', () => {
  it('should accept any string matching the email pattern', () => {
    // Generate strings that look like emails
    const emailArb = fc
      .tuple(
        fc.stringOf(fc.constantFrom(...'abcdefghijklmnopqrstuvwxyz0123456789'.split('')), { minLength: 1, maxLength: 20 }),
        fc.stringOf(fc.constantFrom(...'abcdefghijklmnopqrstuvwxyz'.split('')), { minLength: 1, maxLength: 10 }),
        fc.constantFrom('com', 'org', 'net', 'io', 'dev')
      )
      .map(([local, domain, tld]) => `${local}@${domain}.${tld}`);

    fc.assert(
      fc.property(emailArb, (email) => {
        return validateEmail(email) === true;
      })
    );
  });

  it('should reject any string without an @ symbol', () => {
    const noAtArb = fc.stringOf(
      fc.constantFrom(...'abcdefghijklmnopqrstuvwxyz0123456789.'.split('')),
      { minLength: 1, maxLength: 50 }
    );

    fc.assert(
      fc.property(noAtArb, (input) => {
        return validateEmail(input) === false;
      })
    );
  });
});
```

### Example 2: Serialization Round-Trip

```typescript
import fc from 'fast-check';
import { serialize, deserialize } from './serializer';

describe('serialization properties', () => {
  it('should round-trip any User object', () => {
    const userArb = fc.record({
      id: fc.uuid(),
      name: fc.string({ minLength: 1, maxLength: 100 }),
      email: fc.emailAddress(),
      age: fc.integer({ min: 0, max: 150 }),
      roles: fc.array(fc.constantFrom('admin', 'user', 'editor'), { maxLength: 3 }),
      createdAt: fc.date(),
    });

    fc.assert(
      fc.property(userArb, (user) => {
        const serialized = serialize(user);
        const deserialized = deserialize(serialized);

        // Deep equality after round-trip
        expect(deserialized).toEqual(user);
      })
    );
  });

  it('should produce valid JSON for any input', () => {
    const userArb = fc.record({
      id: fc.uuid(),
      name: fc.string(),
      email: fc.emailAddress(),
    });

    fc.assert(
      fc.property(userArb, (user) => {
        const serialized = serialize(user);
        // Should not throw
        JSON.parse(serialized);
        return true;
      })
    );
  });
});
```

### Example 3: Mathematical Properties

```typescript
import fc from 'fast-check';
import { clamp } from './math-utils';

describe('clamp properties', () => {
  it('should always return a value within [min, max]', () => {
    fc.assert(
      fc.property(
        fc.integer(),
        fc.integer(),
        fc.integer(),
        (value, a, b) => {
          const min = Math.min(a, b);
          const max = Math.max(a, b);
          const result = clamp(value, min, max);
          return result >= min && result <= max;
        }
      )
    );
  });

  it('should be idempotent: clamp(clamp(x)) === clamp(x)', () => {
    fc.assert(
      fc.property(
        fc.integer(),
        fc.integer({ min: -1000, max: 0 }),
        fc.integer({ min: 0, max: 1000 }),
        (value, min, max) => {
          const once = clamp(value, min, max);
          const twice = clamp(once, min, max);
          return once === twice;
        }
      )
    );
  });

  it('should return the value unchanged if already in range', () => {
    fc.assert(
      fc.property(
        fc.integer({ min: -100, max: 100 }),
        (value) => {
          return clamp(value, -100, 100) === value;
        }
      )
    );
  });
});
```

## Hypothesis Example (Python)

```python
from hypothesis import given, strategies as st, assume
from my_module import sort_records, parse_csv_line

@given(st.lists(st.integers()))
def test_sort_preserves_length(xs):
    """Sorting should never add or remove elements."""
    assert len(sort_records(xs)) == len(xs)

@given(st.lists(st.integers()))
def test_sort_preserves_elements(xs):
    """Sorting should contain exactly the same elements."""
    assert sorted(sort_records(xs)) == sorted(xs)

@given(st.lists(st.integers()))
def test_sort_is_ordered(xs):
    """Every element should be ≤ the next."""
    result = sort_records(xs)
    for i in range(len(result) - 1):
        assert result[i] <= result[i + 1]

@given(st.lists(st.text(min_size=1, alphabet=st.characters(whitelist_categories=('L', 'N'))), min_size=1))
def test_csv_round_trip(fields):
    """Joining fields with commas and parsing should return the original fields."""
    assume(all(',' not in f and '"' not in f for f in fields))
    line = ','.join(fields)
    assert parse_csv_line(line) == fields
```

## When to Use Property-Based Testing

| Great Fit | Poor Fit |
|-----------|----------|
| Pure functions (no side effects) | UI interaction flows |
| Serialization / deserialization | Tests requiring complex setup |
| Parsing and validation | Tests with external dependencies |
| Mathematical computations | Tests where properties are hard to define |
| Data transformations | Business logic with many special cases |
| Codec / encoding / compression | Stateful workflows |

## Common Properties to Test

| Property | Description | Example |
|----------|-------------|---------|
| **Round-trip** | encode → decode returns original | `deserialize(serialize(x)) === x` |
| **Idempotency** | Applying twice equals applying once | `format(format(x)) === format(x)` |
| **Invariant** | Some condition always holds | `sort(x).length === x.length` |
| **Commutativity** | Order doesn't matter | `merge(a, b) === merge(b, a)` |
| **Monotonicity** | Larger input → larger output | `if a ≤ b then f(a) ≤ f(b)` |
| **Hard to compute, easy to verify** | Check the answer not the process | Verify sorted output is ordered |
