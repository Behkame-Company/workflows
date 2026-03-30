# Multi-Window Development with Claude Code

> Running parallel Claude Code sessions for speed — the initializer-worker pattern that lets you implement features concurrently with focused context per window.

---

## Why Multi-Window?

Single-session development hits two bottlenecks: **context limits** and **sequential execution**. Multi-window development solves both.

```
┌──────────────────────────────────────────────────────────────┐
│            Single vs Multi-Window                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Single Window                Multi-Window                  │
│   ─────────────                ────────────                  │
│   Task A ──▶ Task B ──▶ C     Task A ──▶ Done               │
│   (sequential, shared          Task B ──▶ Done               │
│    context fills up)           Task C ──▶ Done               │
│                                (parallel, fresh              │
│   Time: ████████████████        context each)                │
│                                                              │
│                                Time: ██████                  │
│                                                              │
│   Benefits:                                                  │
│   • Each window has focused context                          │
│   • Parallel implementation                                  │
│   • Clear contracts between features                         │
│   • No context starvation                                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## The Initializer-Worker Pattern

The core multi-window pattern: one window designs the architecture, workers implement against it.

```
┌──────────────────────────────────────────────────────────────┐
│           Initializer-Worker Pattern                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Window 1 (Initializer)                                     │
│   ──────────────────────                                     │
│   1. Create architecture spec                                │
│   2. Define interfaces / contracts                           │
│   3. Write shared types                                      │
│   4. Set up integration test stubs                           │
│          │                                                   │
│          ├──── spec.md, types.ts, interfaces ────┐           │
│          │                                       │           │
│          ▼                                       ▼           │
│   Window 2 (Worker)               Window 3 (Worker)          │
│   ─────────────────               ─────────────────          │
│   Implements Feature A            Implements Feature B       │
│   against the interface           against the interface      │
│   + unit tests                    + unit tests               │
│          │                                       │           │
│          └──────────────┬────────────────────────┘           │
│                         ▼                                    │
│   Window 1 (Integration)                                     │
│   ──────────────────────                                     │
│   1. Integrate features                                      │
│   2. Run integration tests                                   │
│   3. Fix cross-feature issues                                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Implementation Guide

### Step 1: Initializer Creates Architecture

In your first terminal window:

```bash
# Window 1: Architecture
$ claude
> I'm building a notification system with three components:
> 1. NotificationService (core logic)
> 2. EmailProvider (sends emails)
> 3. WebhookProvider (sends webhooks)
>
> Create:
> - src/types/notification.ts with all shared interfaces
> - src/services/notification.interface.ts with the service contract
> - src/providers/provider.interface.ts with the provider contract
> - A brief spec in docs/notification-spec.md
>
> Don't implement — just define the interfaces and contracts.
> Other developers will implement against these.
```

The initializer produces files like:

```typescript
// src/types/notification.ts
export interface Notification {
  id: string;
  type: "email" | "webhook";
  recipient: string;
  payload: Record<string, unknown>;
  status: "pending" | "sent" | "failed";
  createdAt: Date;
}

export interface SendResult {
  success: boolean;
  messageId?: string;
  error?: string;
}
```

```typescript
// src/providers/provider.interface.ts
import { Notification, SendResult } from "../types/notification";

export interface NotificationProvider {
  readonly type: string;
  send(notification: Notification): Promise<SendResult>;
  validateRecipient(recipient: string): boolean;
}
```

### Step 2: Launch Worker Windows

Open new terminal windows and assign focused tasks:

```bash
# Window 2: Email Provider
$ claude
> Implement the EmailProvider in src/providers/email.provider.ts
> It must implement the NotificationProvider interface in
> src/providers/provider.interface.ts
>
> Use the types from src/types/notification.ts
> Use nodemailer for sending (already in package.json)
> Write tests in src/providers/__tests__/email.provider.test.ts
> Follow patterns from existing services in src/services/
```

```bash
# Window 3: Webhook Provider
$ claude
> Implement the WebhookProvider in src/providers/webhook.provider.ts
> It must implement the NotificationProvider interface in
> src/providers/provider.interface.ts
>
> Use the types from src/types/notification.ts
> Include HMAC signature generation for webhook security
> Write tests in src/providers/__tests__/webhook.provider.test.ts
> Follow patterns from existing services in src/services/
```

### Step 3: Integration in Main Window

After workers finish, return to Window 1:

```bash
# Window 1: Integration
$ claude
> The EmailProvider and WebhookProvider are now implemented.
> 1. Implement the NotificationService in src/services/notification.service.ts
>    that uses these providers
> 2. Wire everything together in src/index.ts
> 3. Write integration tests that test the full flow
> 4. Run all tests and fix any issues
```

## Shared Context Through Files

Workers coordinate through the file system, not through shared context:

```
┌──────────────────────────────────────────────────┐
│          Coordination Mechanism                   │
├──────────────────────────────────────────────────┤
│                                                   │
│   Shared Files (source of truth)                  │
│   ──────────────────────────────                  │
│   • Interface definitions (*.interface.ts)        │
│   • Type definitions (types/*.ts)                 │
│   • Spec documents (docs/*.md)                    │
│   • CLAUDE.md (project conventions)               │
│                                                   │
│   Each Worker Reads                               │
│   ─────────────────                               │
│   • Interfaces they must implement                │
│   • Types they must use                           │
│   • Spec for requirements                         │
│   • CLAUDE.md for conventions                     │
│                                                   │
│   Each Worker Writes                              │
│   ──────────────────                              │
│   • Implementation files                          │
│   • Test files                                    │
│   • Nothing that another worker also writes       │
│                                                   │
└──────────────────────────────────────────────────┘
```

**Critical rule:** Workers must not edit the same files. Partition work so each worker has exclusive ownership of its files.

## Cross-Session Coordination with Tracking Files

For larger multi-window efforts, use a tracking file:

```json
// .dev/tasks.json
{
  "spec_version": "1.0",
  "tasks": [
    {
      "id": "email-provider",
      "status": "in-progress",
      "assignee": "window-2",
      "files": ["src/providers/email.provider.ts"],
      "tests": ["src/providers/__tests__/email.provider.test.ts"],
      "depends_on": [],
      "notes": ""
    },
    {
      "id": "webhook-provider",
      "status": "in-progress",
      "assignee": "window-3",
      "files": ["src/providers/webhook.provider.ts"],
      "tests": ["src/providers/__tests__/webhook.provider.test.ts"],
      "depends_on": [],
      "notes": ""
    },
    {
      "id": "notification-service",
      "status": "blocked",
      "assignee": "window-1",
      "files": ["src/services/notification.service.ts"],
      "tests": ["src/services/__tests__/notification.service.test.ts"],
      "depends_on": ["email-provider", "webhook-provider"],
      "notes": "Wait for providers to be implemented"
    }
  ]
}
```

Workers update their task status when done, and the integrator can check when dependencies are complete.

## Scaling Guidelines

```
┌──────────────────────────────────────────────────┐
│        How Many Windows?                         │
├──────────────────────────────────────────────────┤
│                                                   │
│  Tasks    Recommended Windows    Coordination     │
│  ─────    ───────────────────    ────────────     │
│  1-2      1 window               None needed      │
│  3-5      2-3 windows            Interfaces       │
│  6-10     3-5 windows            Spec + tracking  │
│  10+      5+ windows             Full spec doc    │
│                                                   │
│  Rule of thumb:                                   │
│  • Each window should have 1-3 related tasks     │
│  • Tasks in a window should not depend on        │
│    tasks in another window                        │
│  • The initializer window handles integration    │
│                                                   │
└──────────────────────────────────────────────────┘
```

## Common Pitfalls

| Pitfall | Prevention |
|---|---|
| Workers edit the same file | Partition files exclusively per worker |
| Interface changes mid-flight | Freeze interfaces before starting workers |
| Merge conflicts | Each worker gets its own file set |
| Context bleed | Start fresh sessions; don't reuse |
| Missing integration | Always have an integration step after workers finish |
