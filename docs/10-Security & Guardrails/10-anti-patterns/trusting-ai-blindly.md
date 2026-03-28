# Anti-Pattern: Trusting AI Blindly

> **OWASP LLM09 — Overreliance:** Blindly depending on AI-generated outputs without
> verification leads to security vulnerabilities, logic errors, and technical debt that
> compounds silently. AI code looks professional, compiles cleanly, and reads convincingly —
> but that surface-level polish masks plausible, subtle mistakes that humans routinely miss
> when they stop reviewing critically.

---

## Why This Matters

Large Language Models are trained to produce *plausible* output, not *correct* output.
There is no internal "Am I sure about this?" check — the model generates the most
statistically likely next token. This means:

- Flawed code can look **identical** to correct code
- Security vulnerabilities are introduced with the same confidence as best practices
- Architectural decisions may be subtly wrong for your specific context
- Edge cases are routinely missed because training data favors happy paths

The result is a new class of risk: **high-confidence, low-accuracy code** that passes
superficial review because it *looks right*.

### The "It Compiles So It Must Be Correct" Trap

Developers have always fought the temptation to ship code that "works on my machine."
AI amplifies this by producing code that:

1. **Compiles without warnings** — syntactically perfect, semantically broken
2. **Passes basic tests** — because the AI wrote the tests to match its own flawed logic
3. **Reads like documentation** — well-commented, well-structured, subtly wrong
4. **Follows conventions** — uses the right patterns but applies them incorrectly

The trap is that AI code bypasses the usual "smell test." When a junior developer writes
bad code, it often *looks* rough — odd variable names, inconsistent formatting, missing
comments. AI code has none of these warning signs. It arrives polished and professional,
which disarms the critical review instinct.

---

## Signs You're Trusting AI Blindly

If any of these describe your workflow, you have an overreliance problem:

### 1. Copy-Pasting AI Code Without Reading It

You accept AI suggestions by hitting Tab or clicking "Accept" without reading the
generated code line by line. You treat the AI like an autocomplete tool for boilerplate,
but you're also accepting its logic, security assumptions, and architectural choices.

**The risk:** You inherit vulnerabilities, deprecated API usage, and logic errors that
you never knew existed in your codebase.

### 2. Approving AI-Generated PRs Without Review

Pull requests that were "written by AI" get a lighter review because the team assumes
the AI did a reasonable job. Reviewers skim instead of scrutinize. The implicit bias is:
"A machine wrote this, so it's probably more consistent than human code."

**The risk:** AI-generated PRs are *more* dangerous than human PRs because they contain
errors that are harder to spot — they don't have the usual human tells of fatigue,
inconsistency, or laziness.

### 3. Skipping Tests Because "The AI Wrote It"

You reduce test coverage or skip writing tests for AI-generated code because it "looks
correct" or because the AI also generated tests. AI-generated tests frequently test the
implementation rather than the requirements — they verify that the code does what it does,
not that it does what it *should*.

**The risk:** Circular validation. The AI writes code with a subtle bug, then writes
tests that validate the buggy behavior. Everything is green. Nothing is correct.

### 4. Not Questioning AI Architectural Decisions

You ask the AI to design a system, and it produces a reasonable-looking architecture.
You adopt it without evaluating alternatives, checking for your specific constraints,
or questioning why it chose that approach. The AI defaults to the most *common*
architecture, not the most *appropriate* one.

**The risk:** You end up with an architecture optimized for a generic tutorial project,
not for your actual scale, compliance, and operational requirements.

### 5. Assuming AI Follows Security Best Practices

You assume the AI is up to date on the latest CVEs, follows OWASP guidelines, uses
secure defaults, and avoids deprecated cryptographic functions. In reality, AI models
are trained on a snapshot of the internet that includes vast amounts of insecure
example code — Stack Overflow answers from 2015, outdated tutorials, and blog posts
that prioritize simplicity over security.

**The risk:** The AI confidently generates code using MD5 for password hashing, `eval()`
for JSON parsing, or SQL string concatenation — and it does so with perfect formatting
and helpful comments.

---

## Real-World Examples of Plausible but Flawed AI Code

### Example 1: SQL Injection Hidden in Clean Code

The AI generates a database query helper that looks professional:

```python
# ❌ AI-generated code — looks clean, contains SQL injection
def get_user_by_email(connection, email):
    """Retrieve a user record by email address."""
    query = f"SELECT id, name, email, role FROM users WHERE email = '{email}'"
    cursor = connection.cursor()
    cursor.execute(query)
    return cursor.fetchone()
```

This code has proper docstrings, good formatting, clear variable names — and a textbook
SQL injection vulnerability. The AI used f-string interpolation instead of parameterized
queries. A reviewer scanning quickly would see "database helper function" and move on.

**The fix:**

```python
# ✅ Corrected — parameterized query prevents injection
def get_user_by_email(connection, email):
    """Retrieve a user record by email address."""
    query = "SELECT id, name, email, role FROM users WHERE email = ?"
    cursor = connection.cursor()
    cursor.execute(query, (email,))
    return cursor.fetchone()
```

### Example 2: JWT Verification That Doesn't Verify

```javascript
// ❌ AI-generated code — accepts 'none' algorithm
const jwt = require('jsonwebtoken');

function verifyToken(token) {
  try {
    const decoded = jwt.decode(token, { complete: true });
    if (decoded.payload.exp < Date.now() / 1000) {
      throw new Error('Token expired');
    }
    return decoded.payload;
  } catch (err) {
    throw new Error('Invalid token');
  }
}
```

This looks like JWT verification. It checks expiration. It has error handling. But it
calls `jwt.decode()` instead of `jwt.verify()` — meaning it **never validates the
signature**. An attacker can forge any token with any payload and it will be accepted.

**The fix:**

```javascript
// ✅ Corrected — actually verifies the signature
const jwt = require('jsonwebtoken');

function verifyToken(token, secret) {
  try {
    return jwt.verify(token, secret, { algorithms: ['HS256'] });
  } catch (err) {
    throw new Error('Invalid token');
  }
}
```

### Example 3: Path Traversal in File Download

```python
# ❌ AI-generated code — path traversal vulnerability
from flask import Flask, send_file

app = Flask(__name__)

@app.route('/download/<filename>')
def download_file(filename):
    """Allow users to download files from the uploads directory."""
    filepath = f"./uploads/{filename}"
    return send_file(filepath, as_attachment=True)
```

Clean route, clear docstring, uses `send_file` correctly — but `filename` is never
sanitized. An attacker can request `/download/../../etc/passwd` and read arbitrary
files from the server.

**The fix:**

```python
# ✅ Corrected — validates the resolved path stays within uploads/
import os
from flask import Flask, send_file, abort

app = Flask(__name__)

UPLOAD_DIR = os.path.abspath('./uploads')

@app.route('/download/<filename>')
def download_file(filename):
    """Allow users to download files from the uploads directory."""
    safe_path = os.path.abspath(os.path.join(UPLOAD_DIR, filename))
    if not safe_path.startswith(UPLOAD_DIR):
        abort(403)
    if not os.path.isfile(safe_path):
        abort(404)
    return send_file(safe_path, as_attachment=True)
```

### Example 4: Insecure Random Token Generation

```python
# ❌ AI-generated code — uses insecure random for security token
import random
import string

def generate_reset_token(length=32):
    """Generate a password reset token."""
    chars = string.ascii_letters + string.digits
    return ''.join(random.choice(chars) for _ in range(length))
```

This uses `random.choice()` (which is **not** cryptographically secure) to generate
password reset tokens. The output looks random to a human but is predictable to an
attacker who can determine the PRNG state.

**The fix:**

```python
# ✅ Corrected — uses cryptographically secure random
import secrets

def generate_reset_token(length=32):
    """Generate a password reset token."""
    return secrets.token_urlsafe(length)
```

### Example 5: Mass Assignment in an API Endpoint

```javascript
// ❌ AI-generated code — mass assignment vulnerability
app.put('/api/users/:id', authenticate, async (req, res) => {
  try {
    const user = await User.findByIdAndUpdate(
      req.params.id,
      req.body,       // Passes entire request body to the database
      { new: true }
    );
    res.json(user);
  } catch (err) {
    res.status(500).json({ error: 'Update failed' });
  }
});
```

The AI generated a concise, well-structured endpoint. But it passes `req.body` directly
to the database update — meaning an attacker can set `{ "role": "admin" }` in the
request body and escalate their privileges.

**The fix:**

```javascript
// ✅ Corrected — explicit field allowlist
app.put('/api/users/:id', authenticate, async (req, res) => {
  try {
    const { name, email, avatar } = req.body;
    const user = await User.findByIdAndUpdate(
      req.params.id,
      { name, email, avatar },
      { new: true }
    );
    res.json(user);
  } catch (err) {
    res.status(500).json({ error: 'Update failed' });
  }
});
```

---

## The Fix: Treat AI Like a Talented New Hire

The correct mental model for AI-generated code is: **a talented but brand-new team
member who doesn't know your codebase, your threat model, or your constraints.**

A new hire might:

- Write clean, idiomatic code that misses project-specific patterns
- Follow general best practices but miss domain-specific requirements
- Produce working code that doesn't account for edge cases you've already discovered
- Make reasonable architectural choices that conflict with existing decisions
- Introduce subtle security issues because they haven't read your threat model

You would **never** merge a new hire's PR without review. Apply the same standard to AI.

### The Review Protocol

| Step | What to Check | Why |
|------|--------------|-----|
| **1. Read every line** | Understand what the code actually does | AI code hides bugs in plain sight |
| **2. Check security** | Injection, auth, crypto, input validation | AI defaults to insecure patterns |
| **3. Verify edge cases** | Nulls, empty inputs, boundary values, concurrency | AI optimizes for the happy path |
| **4. Run independent tests** | Write your own tests, don't just use AI-generated ones | AI tests validate its own bugs |
| **5. Check dependencies** | Verify packages exist, are maintained, are not typosquats | AI hallucinate package names |
| **6. Validate architecture** | Does this fit your system's actual constraints? | AI gives generic solutions |

---

## Self-Assessment Checklist

Use this checklist honestly. If you check three or more items, you need to change your
workflow.

- [ ] I regularly accept AI code suggestions without reading them fully
- [ ] I review AI-generated PRs less carefully than human-written PRs
- [ ] I have shipped AI-generated code without writing additional tests for it
- [ ] I have adopted an AI-suggested architecture without evaluating alternatives
- [ ] I assume AI-generated code handles security correctly by default
- [ ] I use AI-generated tests as the *only* tests for AI-generated code
- [ ] I have never found a bug in AI-generated code (this means you're not looking)
- [ ] I feel that questioning AI output slows me down
- [ ] I have used AI-generated code in production without a security review
- [ ] I trust AI more when the code has good comments and formatting

### Scoring

| Checked | Assessment |
|---------|-----------|
| **0–1** | Healthy skepticism — keep it up |
| **2–3** | Caution — review your process and add verification steps |
| **4–6** | Warning — you are likely shipping unreviewed vulnerabilities |
| **7+**  | Critical — stop and restructure your AI workflow immediately |

---

## Tips for Healthy AI Usage

### Do This

- ✅ **Read every line** of AI-generated code before accepting it
- ✅ **Write your own tests** that validate requirements, not just implementation
- ✅ **Run SAST tools** (static analysis) on AI-generated code the same as any other code
- ✅ **Question architectural suggestions** — ask "why this approach over alternatives?"
- ✅ **Verify dependency names** — AI frequently hallucinate package names that may be typosquats
- ✅ **Check for deprecated APIs** — AI training data includes outdated patterns
- ✅ **Review crypto choices** — AI commonly suggests weak algorithms (MD5, SHA1, DES)
- ✅ **Validate input handling** — ensure all user input is sanitized and validated
- ✅ **Maintain threat models** — update them to include AI-assisted development as an attack vector
- ✅ **Pair AI speed with human judgment** — use AI to draft, use your brain to verify

### Don't Do This

- ❌ **Don't copy-paste without reading** — even "simple" utility functions can hide bugs
- ❌ **Don't skip code review** for AI-generated PRs — review them *more* carefully
- ❌ **Don't trust AI-generated tests alone** — they often test the implementation, not the spec
- ❌ **Don't assume security** — AI does not run a threat model before generating code
- ❌ **Don't blindly trust AI dependency suggestions** — verify packages exist and are legitimate
- ❌ **Don't let AI choose your architecture** without evaluating it against your constraints
- ❌ **Don't disable linters or SAST** to make AI-generated code pass — fix the code instead
- ❌ **Don't assume AI is up to date** — models have training cutoffs and miss recent CVEs
- ❌ **Don't confuse fluency with accuracy** — well-written code is not necessarily correct code
- ❌ **Don't use AI-generated secrets, keys, or salts** — they are not random, they are patterns

---

## Building a Verification Culture

### For Individual Developers

1. **Adopt a "trust but verify" mindset.** Use AI to accelerate drafting, but always
   verify before committing.
2. **Keep a bug journal.** Track every bug you find in AI-generated code. Patterns will
   emerge that help you review more efficiently.
3. **Learn the common failure modes.** AI tends to fail in predictable ways — injection,
   missing auth checks, insecure defaults, race conditions, and missing error handling.
4. **Use AI to review AI.** Ask one AI session to critique the output of another. This
   isn't foolproof, but it catches surface-level issues.

### For Teams

1. **Don't relax review standards for AI code.** If anything, tighten them. Add
   "AI-generated code" as a label on PRs so reviewers know to be extra careful.
2. **Require independent tests.** AI-generated code must have tests written by a human
   (or at least human-reviewed tests written with different prompts).
3. **Run automated security scans.** SAST, DAST, dependency scanning, and secret
   detection should run on every PR regardless of who — or what — wrote the code.
4. **Track AI-related bugs.** If your team finds a pattern of AI-introduced issues,
   document it and add it to your review checklist.

### For Organizations

1. **Include AI in your threat model.** AI-assisted development is a new attack surface.
   Treat it as such in your security documentation.
2. **Set policy on AI code review.** Define minimum review standards for AI-generated
   code — this should be part of your SDLC.
3. **Train developers on AI failure modes.** Security training should cover how AI
   introduces vulnerabilities, not just how humans do.
4. **Audit AI-generated code in production.** If you can trace which code was
   AI-generated, prioritize it for security audits.

---

## The Paradox of AI Productivity

AI dramatically increases the speed at which code is produced. This is genuinely
valuable — but only if the code is correct. If AI-generated code has a higher defect
rate per line (which current research suggests it does for security-sensitive code),
then the productivity gain is partially offset by:

- **Increased review burden** — more code to review per unit time
- **Subtle bug density** — harder-to-find bugs that take longer to diagnose
- **False confidence** — the team believes they are moving faster, but they are
  accumulating technical and security debt
- **Delayed discovery** — AI bugs tend to surface in production, not in development,
  because they pass superficial testing

The net effect is that **unreviewed AI code is a productivity illusion**. You ship
faster, but you fix more — and the fixes are more expensive because the bugs are
more subtle.

---

## Common Objections (and Why They're Wrong)

### "AI saves us so much time — adding review overhead defeats the purpose."

The time saved generating code is only real if the code is correct. Debugging a subtle
AI-introduced vulnerability in production costs orders of magnitude more than a 10-minute
code review. The review *is* the productivity — it's what converts AI-generated drafts
into shippable code.

### "Our AI tool is better than the others — it doesn't make these mistakes."

Every LLM, regardless of vendor or version, is a statistical text generator. Some are
better calibrated than others, but none have a formal verification step. The failure
modes described in this document apply to all current-generation AI coding tools.

### "We have CI/CD — it catches everything."

CI/CD catches what your tests cover. If the AI wrote the tests, and the tests validate
the buggy behavior, CI will report green on broken code. Automated pipelines are
necessary but not sufficient.

---

## Key Takeaways

1. **AI generates plausible code, not correct code.** Plausibility is not a proxy for
   correctness.
2. **Polished code is not safe code.** Good formatting and helpful comments do not
   indicate the absence of vulnerabilities.
3. **Review AI code more carefully, not less.** The errors are harder to spot because
   they lack the usual human tells.
4. **Write independent tests.** Never rely solely on AI-generated tests for
   AI-generated code.
5. **Treat AI like a new hire.** Talented, fast, eager — but unfamiliar with your
   system and unaware of your threat model.
6. **Update your threat model.** AI-assisted development is an attack vector. Document
   it and mitigate it like any other risk.

---

## Next Steps

- [Security by Obscurity](security-by-obscurity.md) — Another dangerous assumption:
  that hiding implementation details is a substitute for real security controls.
- [Overpermissioned Agents](overpermissioned-agents.md) — When AI agents operate with
  more access than they need, blind trust becomes an even greater liability.
- [Ignoring AI in Threat Models](ignoring-ai-in-threat-models.md) — If your threat
  model doesn't account for AI-assisted development, you have a blind spot that
  attackers will find before you do.
