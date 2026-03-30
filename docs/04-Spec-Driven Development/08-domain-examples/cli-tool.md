# Example: CLI Tool Spec

> A complete SDD example for a command-line application.

---

## Feature Spec: Task CLI

```markdown
# Feature: Task CLI (tsk)

## Summary
A command-line task management tool that integrates with the Task Manager Pro
API. Users can create, list, update, and complete tasks from the terminal,
enabling developer workflows without leaving the command line.

## User Stories

### US-1: List Tasks
As a developer, I want to list my tasks from the terminal so that I can
see my work without opening a browser.

### US-2: Create Task
As a developer, I want to create tasks from the CLI so that I can capture
ideas immediately while coding.

### US-3: Update Task Status
As a developer, I want to move tasks between statuses from the CLI so that
I can update progress without context-switching.

### US-4: Configure Connection
As a developer, I want to authenticate and configure the CLI once so that
subsequent commands work without re-entering credentials.

## Commands

### `tsk list`
List tasks with optional filters.

```
tsk list                           # All tasks (default: non-done)
tsk list --status todo             # Only "To Do" tasks
tsk list --status in-progress      # Only "In Progress"
tsk list --all                     # Include completed tasks
tsk list --assignee @me            # My tasks only
tsk list --label bug               # Filter by label
tsk list --format json             # Output as JSON (for piping)
tsk list --format table            # Output as table (default)
```

**Output (table format):**
```
ID     STATUS        PRIORITY  TITLE
#142   In Progress   High      Fix login redirect
#138   To Do         Medium    Update API docs
#135   To Do         Low       Add dark mode toggle
```

### `tsk create`
Create a new task.

```
tsk create "Fix login redirect"              # Title only (status: todo)
tsk create "Fix login" --priority high       # With priority
tsk create "Fix login" --label bug --label auth  # With labels
tsk create --interactive                      # Interactive mode (prompts)
```

### `tsk move`
Change task status.

```
tsk move 142 in-progress     # Move task #142 to In Progress
tsk move 142 done            # Complete task #142
tsk move 142 todo            # Move back to To Do
```

### `tsk show`
Show task details.

```
tsk show 142                 # Full task details
```

**Output:**
```
Task #142: Fix login redirect
Status:    In Progress
Priority:  High
Labels:    bug, auth
Created:   2026-01-15 by @jane
Updated:   2026-01-18

Description:
  After OAuth login, users are redirected to /home instead of
  the page they were on before login.
```

### `tsk config`
Configure CLI authentication.

```
tsk config                   # Interactive setup
tsk config --token <token>   # Set API token
tsk config --url <api-url>   # Set API base URL
tsk config --show            # Show current config (token masked)
```

## Acceptance Criteria

### Authentication
- AC-1: `tsk config --token <token>` stores token in ~/.tsk/config.json
- AC-2: Token is stored with file permissions 0600 (owner read/write only)
- AC-3: All commands without valid config show "Run `tsk config` to set up"

### Listing
- AC-4: `tsk list` shows non-completed tasks sorted by priority
- AC-5: `tsk list --format json` outputs valid JSON parseable by `jq`
- AC-6: `tsk list` with no tasks shows "No tasks found" (not empty output)

### Creating
- AC-7: `tsk create "Title"` creates task and prints "Created task #<id>"
- AC-8: `tsk create ""` shows error "Task title cannot be empty"

### Moving
- AC-9: `tsk move <id> <status>` updates status and prints confirmation
- AC-10: `tsk move <invalid-id> done` shows "Task not found"
- AC-11: Valid statuses: todo, in-progress, in-review, done

### General
- AC-12: `tsk --help` shows usage for all commands
- AC-13: `tsk --version` shows current version
- AC-14: All errors exit with non-zero code and write to stderr
- AC-15: Network errors show "Connection failed. Check your network and API URL."

## Edge Cases

| Scenario | Expected Behavior |
|----------|------------------|
| No network connection | "Connection failed..." error, exit code 1 |
| API returns 401 | "Authentication failed. Run `tsk config` to update token." |
| API returns 429 | "Rate limited. Please wait and try again." |
| Very long task title (500+ chars) | Truncate in list view, show full in `tsk show` |
| Config file corrupted | "Config file corrupted. Run `tsk config` to reset." |
| Piped output (non-TTY) | Skip colors and formatting, plain text |
| Windows vs Unix paths | Use OS-appropriate config directory |

## Non-Functional Requirements
- Startup time: <200ms for any command
- Binary size: <10MB
- Dependencies: Zero runtime dependencies (single binary)
- Supported OS: macOS, Linux, Windows (x64 and ARM64)
- Config location: ~/.tsk/ (XDG_CONFIG_HOME on Linux)

## Out of Scope (v1)
- Interactive TUI mode (full terminal UI)
- Offline mode with sync
- Task comments via CLI
- Bulk operations (move multiple tasks)
- Shell completions (Bash, Zsh, Fish)
```

---

## Why This Spec Works for CLI Tools

1. **Every command fully documented** — Input, output, and errors specified
2. **Exact output format** — AI knows precisely what to print
3. **Exit codes** — Essential for scripting and CI/CD integration
4. **Cross-platform** — OS differences explicitly addressed
5. **Non-TTY behavior** — Piping behavior defined (critical for CLI tools)
6. **Configuration security** — File permissions specified (0600)

---

## Key Differences from Web App Specs

| Dimension | Web App Spec | CLI Spec |
|-----------|-------------|---------|
| UI description | Visual layout, components | Command syntax, output format |
| User interaction | Form inputs, clicks | Flags, arguments, stdin |
| Output | Rendered HTML/React | Stdout text, exit codes |
| Error display | Toast/modal | Stderr + exit code |
| Configuration | .env, UI settings | Config file, env vars |
| Testing | E2E with browser | Integration with process spawning |
