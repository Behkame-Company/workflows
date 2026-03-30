# Shell and File Tools

> The two most fundamental agent tools — command execution and file manipulation — and the best practices that make them reliable in autonomous workflows.

---

## The Foundation

Every coding agent, regardless of architecture, relies on two core capabilities:

1. **Shell tools** — execute commands, capture output, handle errors
2. **File tools** — read, write, edit, and search files

These are the hands and eyes of a coding agent. Get them right, and the agent can accomplish nearly anything.

```
┌──────────────────────────────────────────────────────────────────┐
│                 FUNDAMENTAL AGENT TOOLS                           │
│                                                                  │
│  ┌──────────────────────┐    ┌──────────────────────┐           │
│  │     SHELL TOOL        │    │     FILE TOOLS        │           │
│  │                       │    │                       │           │
│  │  • Run commands       │    │  • Read files         │           │
│  │  • Capture stdout     │    │  • Write files        │           │
│  │  • Capture stderr     │    │  • Edit (search &     │           │
│  │  • Handle exit codes  │    │    replace)            │           │
│  │  • Manage timeouts    │    │  • Search across       │           │
│  │  • Environment vars   │    │    files               │           │
│  └──────────┬───────────┘    │  • List directories   │           │
│             │                └──────────┬───────────┘           │
│             │                           │                        │
│             └───────────┬───────────────┘                        │
│                         │                                        │
│                         ▼                                        │
│              ┌────────────────────┐                              │
│              │   GIT INTEGRATION  │                              │
│              │                    │                              │
│              │  • Commit changes  │                              │
│              │  • Create branches │                              │
│              │  • Diff/status     │                              │
│              │  • Undo via reset  │                              │
│              └────────────────────┘                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Shell Tool Design

### Capturing All Output

Always capture **both stdout and stderr**. Many important signals (warnings, errors, deprecation notices) only appear on stderr.

```typescript
interface ShellResult {
  stdout: string;
  stderr: string;
  exitCode: number;
  timedOut: boolean;
}

async function shellExecute(command: string, options?: ShellOptions): Promise<ShellResult> {
  const timeout = options?.timeout ?? 30_000; // 30s default
  const cwd = options?.cwd ?? process.cwd();

  try {
    const { stdout, stderr } = await execWithTimeout(command, { cwd, timeout });
    return { 
      stdout, 
      stderr, 
      exitCode: 0, 
      timedOut: false 
    };
  } catch (error) {
    if (error.killed) {
      return { 
        stdout: error.stdout ?? "", 
        stderr: `Command timed out after ${timeout}ms: ${command}`, 
        exitCode: -1, 
        timedOut: true 
      };
    }
    return { 
      stdout: error.stdout ?? "", 
      stderr: error.stderr ?? error.message, 
      exitCode: error.code ?? 1, 
      timedOut: false 
    };
  }
}
```

### Timeout Management

Long-running commands can stall an agent indefinitely. Always set timeouts, with awareness of what reasonable durations look like:

```typescript
const timeoutPresets = {
  quick:    10_000,   // 10s  — git status, ls, cat
  standard: 30_000,   // 30s  — grep, find, simple scripts
  build:   120_000,   // 2min — npm install, build commands
  test:    300_000,   // 5min — full test suites
  long:    600_000,   // 10min — heavy builds, integration tests
};

async function smartShell(command: string): Promise<ShellResult> {
  // Infer appropriate timeout from command
  const timeout = inferTimeout(command);
  return shellExecute(command, { timeout });
}

function inferTimeout(command: string): number {
  if (/\b(install|build|compile)\b/.test(command)) return timeoutPresets.build;
  if (/\b(test|jest|pytest|mocha)\b/.test(command)) return timeoutPresets.test;
  if (/\b(git|ls|cat|echo|pwd)\b/.test(command)) return timeoutPresets.quick;
  return timeoutPresets.standard;
}
```

### Security Considerations

```typescript
// Commands that should require explicit permission
const dangerousPatterns = [
  /\brm\s+(-rf?|--recursive)\b/,     // Recursive delete
  /\bsudo\b/,                          // Privilege escalation
  /\bcurl\b.*\|\s*sh/,                 // Pipe to shell
  /\bchmod\s+777\b/,                   // Open permissions
  /\b(DROP|DELETE\s+FROM|TRUNCATE)\b/i, // Destructive SQL
];

function validateCommand(command: string): ValidationResult {
  for (const pattern of dangerousPatterns) {
    if (pattern.test(command)) {
      return { 
        allowed: false, 
        reason: `Potentially dangerous command matching: ${pattern}`,
        suggestion: "Use a more specific, safer alternative"
      };
    }
  }
  return { allowed: true };
}
```

## File Tool Design

### Read Tool

```typescript
interface ReadFileParams {
  path: string;        // Absolute path
  startLine?: number;  // Optional line range
  endLine?: number;
}

async function readFile(params: ReadFileParams): Promise<ToolResult> {
  // Poka-yoke: require absolute paths
  if (!path.isAbsolute(params.path)) {
    return { 
      success: false, 
      error: `Use absolute paths. Got "${params.path}". Try "/workspace/${params.path}"` 
    };
  }

  try {
    const content = await fs.readFile(params.path, "utf-8");
    const lines = content.split("\n");

    // Apply line range if specified
    if (params.startLine !== undefined) {
      const start = Math.max(0, params.startLine - 1); // 1-indexed input
      const end = params.endLine ? Math.min(lines.length, params.endLine) : start + 100;
      
      const slice = lines.slice(start, end);
      return {
        success: true,
        content: slice.map((line, i) => `${start + i + 1}. ${line}`).join("\n"),
        totalLines: lines.length,
        showing: `lines ${start + 1}-${end} of ${lines.length}`
      };
    }

    // Truncate large files
    if (lines.length > 500) {
      return {
        success: true,
        content: lines.slice(0, 200).map((l, i) => `${i + 1}. ${l}`).join("\n"),
        totalLines: lines.length,
        truncated: true,
        message: `File has ${lines.length} lines. Showing first 200. Use startLine/endLine for specific sections.`
      };
    }

    return {
      success: true,
      content: lines.map((l, i) => `${i + 1}. ${l}`).join("\n"),
      totalLines: lines.length
    };
  } catch (error) {
    return { success: false, error: `Cannot read ${params.path}: ${error.message}` };
  }
}
```

### Edit Tool (Search-and-Replace)

The search-and-replace pattern is **far more reliable** than line-number-based editing:

```typescript
interface EditParams {
  path: string;
  oldText: string;
  newText: string;
}

async function editFile(params: EditParams): Promise<ToolResult> {
  if (!path.isAbsolute(params.path)) {
    return { success: false, error: `Use absolute paths. Got: "${params.path}"` };
  }

  const content = await fs.readFile(params.path, "utf-8");

  // Check: does old text exist?
  if (!content.includes(params.oldText)) {
    // Help the agent recover
    const similar = findMostSimilarSection(content, params.oldText);
    return {
      success: false,
      error: `oldText not found in file.${
        similar 
          ? `\n\nMost similar text (line ${similar.line}):\n"${similar.text}"` 
          : "\nThe file may have changed. Read it again first."
      }`
    };
  }

  // Check: is old text unique?
  const occurrences = content.split(params.oldText).length - 1;
  if (occurrences > 1) {
    return {
      success: false,
      error: `oldText appears ${occurrences} times. Include more context to make it unique.`
    };
  }

  // Apply the edit
  const newContent = content.replace(params.oldText, params.newText);
  await fs.writeFile(params.path, newContent);

  return { 
    success: true, 
    message: `Replaced text in ${params.path}`,
    linesChanged: params.newText.split("\n").length - params.oldText.split("\n").length
  };
}
```

### Write Tool (Full File Creation)

```typescript
async function writeFile(params: { path: string; content: string }): Promise<ToolResult> {
  if (!path.isAbsolute(params.path)) {
    return { success: false, error: `Use absolute paths. Got: "${params.path}"` };
  }

  // Create parent directories if they don't exist
  await fs.mkdir(path.dirname(params.path), { recursive: true });

  // Atomic write: write to temp file then rename
  const tempPath = `${params.path}.tmp`;
  await fs.writeFile(tempPath, params.content);
  await fs.rename(tempPath, params.path);

  return { 
    success: true, 
    message: `Wrote ${params.content.split("\n").length} lines to ${params.path}` 
  };
}
```

## Git as an Undo Mechanism

Git integration transforms file tools from dangerous to safe — every change is reversible:

```typescript
// Before making changes: create a checkpoint
async function createCheckpoint(message: string): Promise<string> {
  await shell("git add -A");
  const result = await shell(`git commit -m "checkpoint: ${message}" --allow-empty`);
  const hash = await shell("git rev-parse HEAD");
  return hash.stdout.trim();
}

// If something goes wrong: revert to checkpoint
async function revertToCheckpoint(hash: string): Promise<void> {
  await shell(`git reset --hard ${hash}`);
}

// Agent workflow with git safety net
async function safeAgentEdit(task: string) {
  const checkpoint = await createCheckpoint("before agent changes");
  
  try {
    const result = await agentLoop(task);
    
    // Verify changes
    const tests = await shell("npm test");
    if (tests.exitCode !== 0) {
      console.log("Tests failed — reverting changes");
      await revertToCheckpoint(checkpoint);
      throw new Error("Agent changes broke tests");
    }
    
    return result;
  } catch (error) {
    await revertToCheckpoint(checkpoint);
    throw error;
  }
}
```

## Shell and File Tools in Practice

### Copilot CLI

Copilot CLI provides these tools to its agent:

```
┌────────────────────────────────────────────────────────────────┐
│              COPILOT CLI TOOL SET                                │
│                                                                │
│  Shell:                                                        │
│  • powershell / bash — full command execution                  │
│  • Captures stdout, stderr, exit codes                         │
│  • Timeout management                                          │
│                                                                │
│  Files:                                                        │
│  • view — read files with line numbers                         │
│  • edit — search-and-replace in files                          │
│  • create — write new files                                    │
│  • glob — find files by pattern                                │
│  • grep — search file contents                                 │
│                                                                │
│  Git (via shell):                                              │
│  • All git commands available through shell tool               │
│  • Atomic commits for each logical change                      │
│                                                                │
│  Permissions (--allow-tool):                                   │
│  • 'shell(git)' — allow all git commands                       │
│  • 'shell(npm test)' — allow specific test commands            │
│  • 'write' — allow all file writes                             │
│  • 'edit' — allow file edits                                   │
└────────────────────────────────────────────────────────────────┘
```

### Permission Model

```
┌────────────────────────────────────────────────────────────────┐
│              TOOL PERMISSION STRATEGIES                          │
│                                                                │
│  CONSERVATIVE (default):                                       │
│    Agent asks for permission before every tool call            │
│    → Safest, but slow and interrupts flow                      │
│                                                                │
│  ALLOW-LIST:                                                   │
│    Pre-approve specific tools                                  │
│    --allow-tool='shell(git)'     ← all git commands            │
│    --allow-tool='shell(npm test)' ← only npm test              │
│    --allow-tool='write(src/**)'  ← writes only in src/         │
│    → Good balance of safety and speed                          │
│                                                                │
│  PERMISSIVE:                                                   │
│    Agent has full access to all tools                           │
│    → Fastest, but requires trust in the agent                  │
│    → Use in sandboxed environments only                        │
└────────────────────────────────────────────────────────────────┘
```

## Best Practices

### 1. Always Capture stderr

```typescript
// ❌ Agent sees: "" (empty) — thinks command worked
const result = await exec("npm install"); // stderr discarded

// ✅ Agent sees: "WARN deprecated package" — can make informed decisions
const result = await shellExecute("npm install"); // captures both streams
```

### 2. Set Timeouts on All Shell Commands

```typescript
// ❌ Agent hangs forever on interactive prompt
await exec("npm init");  // Waits for user input that never comes

// ✅ Agent gets a clear timeout signal
const result = await shellExecute("npm init --yes", { timeout: 10_000 });
```

### 3. Use Atomic File Writes

```typescript
// ❌ Interrupted write = corrupted file
await fs.writeFile(path, content);  // Power loss mid-write = partial file

// ✅ Atomic: write temp file, then rename
await fs.writeFile(`${path}.tmp`, content);
await fs.rename(`${path}.tmp`, path);  // Rename is atomic on most filesystems
```

### 4. Validate File Paths Before Operations

```typescript
// ❌ Silent failure or unexpected behavior
await fs.readFile("../../etc/passwd");  // Path traversal

// ✅ Validate and constrain paths
function validatePath(filePath: string, workspaceRoot: string): boolean {
  const resolved = path.resolve(filePath);
  return resolved.startsWith(workspaceRoot);
}
```

### 5. Use Git for Undo Capability

```typescript
// ❌ No way to undo — agent's bad edit is permanent
await editFile({ path, oldText, newText });

// ✅ Git tracks every change — anything is reversible
await editFile({ path, oldText, newText });
await shell("git add -A && git commit -m 'agent: update auth middleware'");
// To undo: git revert HEAD
```
