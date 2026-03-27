# Building Evaluation Suites

> Creating comprehensive evaluation suites for AI-generated code — from defining test cases to tracking results and optimizing prompts through data-driven iteration.

---

## Overview

An evaluation suite is a structured collection of test cases that measures how well an AI model performs on a specific task. Unlike unit tests (which verify your code), eval suites verify your AI's output quality. They're essential for comparing prompts, model versions, and generation strategies with statistical confidence.

## What to Evaluate

AI-generated code should be assessed across multiple dimensions:

| Dimension | What It Measures | Example Check |
|-----------|-----------------|---------------|
| **Correctness** | Does it produce the right output? | Run against test cases with known answers |
| **Performance** | Is it efficient enough? | Measure execution time and memory usage |
| **Style** | Does it follow project conventions? | Lint checks, naming patterns, structure |
| **Edge Cases** | Does it handle unusual inputs? | Empty strings, nulls, huge numbers, Unicode |
| **Security** | Does it introduce vulnerabilities? | SQL injection, XSS, path traversal patterns |
| **Completeness** | Does it address all requirements? | Check each requirement is reflected in output |
| **Robustness** | Does it handle errors gracefully? | Invalid input, network failures, timeouts |

## Eval Suite Structure

Each eval case contains three components:

```
┌──────────────────────────────────────────┐
│              Eval Case                    │
├──────────────┬──────────────┬────────────┤
│   Input      │   Expected   │  Grading   │
│              │   Output     │  Criteria  │
│  - Prompt    │  - Reference │  - Method  │
│  - Context   │    answer    │  - Rubric  │
│  - Params    │  - Test      │  - Weight  │
│              │    cases     │  - Threshold│
└──────────────┴──────────────┴────────────┘
```

### Eval Suite JSON Format

```json
{
  "suite": {
    "name": "password-validator-eval",
    "version": "1.0.0",
    "description": "Evaluates AI-generated password validation functions",
    "model": "claude-sonnet-4-20250514",
    "created": "2025-01-15",
    "tags": ["validation", "security", "unit-function"]
  },
  "config": {
    "grading": {
      "code_based_weight": 0.6,
      "llm_based_weight": 0.4,
      "pass_threshold": 0.8
    },
    "execution": {
      "timeout_seconds": 30,
      "max_retries": 0,
      "language": "typescript"
    }
  },
  "cases": [
    {
      "id": "pw-001",
      "name": "Basic validation requirements",
      "prompt": "Write a TypeScript function `validatePassword(password: string): { valid: boolean; errors: string[] }` that requires: minimum 8 characters, at least one uppercase letter, one lowercase letter, one number, and one special character (!@#$%^&*).",
      "context": "This is for a user registration form. Return specific error messages for each failed requirement.",
      "test_cases": [
        {
          "input": "Str0ng!Pass",
          "expected": { "valid": true, "errors": [] }
        },
        {
          "input": "weak",
          "expected": {
            "valid": false,
            "errors": ["Must be at least 8 characters", "Must contain an uppercase letter", "Must contain a number", "Must contain a special character"]
          }
        },
        {
          "input": "",
          "expected": {
            "valid": false,
            "errors": ["Must be at least 8 characters", "Must contain an uppercase letter", "Must contain a lowercase letter", "Must contain a number", "Must contain a special character"]
          }
        },
        {
          "input": "ALLUPPERCASE1!",
          "expected": {
            "valid": false,
            "errors": ["Must contain a lowercase letter"]
          }
        },
        {
          "input": "NoSpecial1Char",
          "expected": {
            "valid": false,
            "errors": ["Must contain a special character"]
          }
        }
      ],
      "llm_rubric": {
        "correctness": "Does the function correctly validate all five requirements?",
        "error_messages": "Are error messages specific and user-friendly?",
        "type_safety": "Does it use TypeScript types properly?",
        "edge_cases": "Does it handle null, undefined, or non-string inputs?"
      },
      "tags": ["basic", "validation"]
    },
    {
      "id": "pw-002",
      "name": "Unicode and special input handling",
      "prompt": "Write a TypeScript function `validatePassword(password: string): { valid: boolean; errors: string[] }` that requires: minimum 8 characters, at least one uppercase letter, one lowercase letter, one number, and one special character (!@#$%^&*).",
      "context": "Must handle Unicode characters, emoji, and very long strings.",
      "test_cases": [
        {
          "input": "Pässwörd1!",
          "expected": { "valid": true, "errors": [] }
        },
        {
          "input": "🔒Password1!",
          "expected": { "valid": true, "errors": [] }
        },
        {
          "input": "A".repeat(10000) + "a1!",
          "expected": { "valid": true, "errors": [] },
          "note": "Performance: should complete within 100ms"
        }
      ],
      "llm_rubric": {
        "unicode_handling": "Does the function handle non-ASCII characters correctly?",
        "performance": "Does the implementation avoid catastrophic backtracking in regex?"
      },
      "tags": ["edge-case", "unicode", "performance"]
    },
    {
      "id": "pw-003",
      "name": "Security considerations",
      "prompt": "Write a TypeScript function `validatePassword(password: string): { valid: boolean; errors: string[] }` that requires: minimum 8 characters, at least one uppercase letter, one lowercase letter, one number, and one special character (!@#$%^&*). Also check against a list of common passwords.",
      "context": "Security-critical: used in financial application. Must not leak information through timing attacks.",
      "test_cases": [
        {
          "input": "Password1!",
          "expected": { "valid": false, "errors": ["Password is too common"] }
        },
        {
          "input": "Qwerty123!",
          "expected": { "valid": false, "errors": ["Password is too common"] }
        }
      ],
      "llm_rubric": {
        "common_passwords": "Does it check against a meaningful list of common passwords?",
        "timing_safety": "Does validation use constant-time comparison where appropriate?",
        "no_logging": "Does it avoid logging the password value?"
      },
      "tags": ["security", "advanced"]
    }
  ]
}
```

## Scaling Your Eval Suite

### Start Small, Then Expand

```
Phase 1: Foundation (20 cases)
├── 10 happy-path cases
├── 5 edge-case cases  
├── 3 error-handling cases
└── 2 security cases

Phase 2: Expansion (50 cases)
├── Add diversity: different input types, lengths, formats
├── Add adversarial: inputs designed to break the code
├── Add regression: cases from real bugs found in production
└── Add performance: cases with timing requirements

Phase 3: Comprehensive (200+ cases)
├── Cover every documented requirement
├── Include real-world examples from production logs
├── Add cross-cutting concerns (i18n, accessibility, concurrency)
└── Include "trick" cases that test common AI mistakes
```

### Automate Case Generation

Generate eval cases programmatically for broader coverage:

```python
import itertools

def generate_password_eval_cases():
    """Generate diverse password validation test cases."""
    cases = []
    
    # Systematic edge cases
    boundary_lengths = [0, 1, 7, 8, 9, 100, 1000]
    char_classes = {
        "upper": "ABCDEF",
        "lower": "abcdef",
        "digit": "123456",
        "special": "!@#$%^"
    }
    
    # Generate cases missing each character class
    for missing_class in char_classes:
        password_chars = ""
        for cls, chars in char_classes.items():
            if cls != missing_class:
                password_chars += chars[0]
        
        # Pad to minimum length
        password = password_chars.ljust(8, "x")
        
        cases.append({
            "id": f"auto-missing-{missing_class}",
            "input": password,
            "expected_valid": False,
            "expected_error_contains": missing_class
        })
    
    return cases
```

## Tracking Eval Results Over Time

Store results in a structured format for trend analysis:

```json
{
  "run_id": "eval-2025-01-15-001",
  "suite": "password-validator-eval",
  "model": "claude-sonnet-4-20250514",
  "prompt_version": "v3",
  "timestamp": "2025-01-15T14:30:00Z",
  "results": {
    "total_cases": 50,
    "passed": 43,
    "failed": 7,
    "score": 0.86,
    "by_dimension": {
      "correctness": 0.92,
      "edge_cases": 0.75,
      "security": 0.80,
      "style": 0.90,
      "performance": 0.95
    },
    "by_tag": {
      "basic": { "passed": 18, "total": 18, "score": 1.00 },
      "edge-case": { "passed": 12, "total": 16, "score": 0.75 },
      "security": { "passed": 6, "total": 8, "score": 0.75 },
      "unicode": { "passed": 4, "total": 5, "score": 0.80 },
      "performance": { "passed": 3, "total": 3, "score": 1.00 }
    },
    "failures": [
      {
        "case_id": "pw-002",
        "reason": "Did not handle emoji in password string",
        "expected": true,
        "actual": "TypeError: Invalid character"
      }
    ]
  }
}
```

### Visualizing Trends

Track scores across prompt iterations and model versions:

```
Score Trend: password-validator-eval
100% ─────────────────────────────────────
 95% ─                           ┌──●  v5 (prompt refined)
 90% ─                    ●──────┘
 85% ─              ●─────┘      v4 (added examples)
 80% ─        ●─────┘            v3 (added context)
 75% ─  ●─────┘                  v2 (basic prompt)
 70% ───●                        v1 (initial)
      ─────────────────────────────────────
       v1    v2    v3    v4    v5
```

## Using Evals to Compare Models and Prompts

### A/B Comparison

Run the same eval suite against different configurations:

```python
configs = [
    {"model": "claude-sonnet-4-20250514", "prompt": "v3"},
    {"model": "claude-sonnet-4-20250514", "prompt": "v4-with-examples"},
    {"model": "claude-haiku", "prompt": "v4-with-examples"},
]

results = {}
for config in configs:
    key = f"{config['model']}_{config['prompt']}"
    results[key] = run_eval_suite("password-validator-eval", config)

# Compare
for key, result in results.items():
    print(f"{key}: {result['score']:.0%}")
```

**Example output:**

| Configuration | Score | Correctness | Edge Cases | Speed |
|--------------|-------|-------------|------------|-------|
| Sonnet + v3 prompt | 86% | 92% | 75% | 2.1s |
| Sonnet + v4 prompt | 93% | 96% | 88% | 2.3s |
| Haiku + v4 prompt | 81% | 88% | 70% | 0.8s |

## Eval-Driven Prompt Optimization Cycle

Use eval results to systematically improve your prompts:

```
┌────────────────┐
│ 1. Run eval    │
│    suite       │
└───────┬────────┘
        │
        ▼
┌────────────────┐
│ 2. Analyze     │
│    failures    │──── What patterns do failures share?
└───────┬────────┘
        │
        ▼
┌────────────────┐
│ 3. Update      │
│    prompt       │──── Add examples, constraints, context
└───────┬────────┘
        │
        ▼
┌────────────────┐
│ 4. Re-run      │
│    eval suite  │──── Did the score improve?
└───────┬────────┘
        │
   ┌────▼────┐
   │ Better? │
   └────┬────┘
   YES  │  NO
   ▼    ▼
  Keep  Revert and try different approach
```

### Optimization Workflow in Practice

```python
# Step 1: Baseline
baseline = run_eval("suite-v1", prompt="v1")
# Score: 72%  — Failures: edge cases, Unicode, empty input

# Step 2: Analyze failures
# Pattern: Model doesn't handle empty/null inputs
# Pattern: Model ignores Unicode characters

# Step 3: Update prompt
# v2: Added "Handle empty strings and null values explicitly"
# v2: Added "Support Unicode characters in passwords"

# Step 4: Re-run
improved = run_eval("suite-v1", prompt="v2")
# Score: 85%  — Improvement confirmed

# Step 5: Iterate
# v3: Added concrete examples of edge cases
final = run_eval("suite-v1", prompt="v3")
# Score: 93%  — Strong improvement, deploy this prompt
```

## Eval Suite Best Practices

| Practice | Why |
|----------|-----|
| **Version your eval suites** | Track what changed between runs |
| **Tag cases by category** | Identify which areas need improvement |
| **Include regression cases** | Prevent previously-fixed failures from recurring |
| **Keep cases independent** | Each case should be evaluable in isolation |
| **Document expected failures** | Some cases may be intentionally hard — mark them |
| **Review eval cases periodically** | Requirements change — evals should too |
| **Separate generation from grading** | Use different models for generation vs evaluation |
| **Store raw outputs** | Enable re-grading with improved rubrics |

## Next Steps

- 🔗 [AI Output Evaluation Framework](evaluation-framework.md) — Framework design and grading methods
- 🔗 [LLM-Based Grading](llm-based-grading.md) — Rubric design and LLM-as-judge patterns
