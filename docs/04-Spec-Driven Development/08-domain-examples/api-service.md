# Example: API Service Spec

> A complete SDD example for a REST API notification service.

---

## Feature Spec: Notification Service API

```markdown
# Feature: Notification Service API

## Summary
Internal API service that receives notification events from other services
and delivers them to users via in-app, email, and webhook channels based
on user preferences. Designed as a standalone microservice.

## User Stories

### US-1: Send Notification
As a backend service, I want to send a notification to a user via API
so that users are informed of important events without direct coupling.

### US-2: User Preferences
As a user, I want to configure which notification types I receive and
through which channels so that I'm not overwhelmed by irrelevant alerts.

### US-3: Notification History
As a user, I want to retrieve my notification history so that I can
review past events I may have missed.

### US-4: Mark as Read
As a user, I want to mark notifications as read individually or in bulk
so that I can track which notifications I've already seen.

## API Contract

### POST /api/v1/notifications
Create and send a notification.

**Request:**
```json
{
  "userId": "uuid",
  "type": "info | warning | error | success",
  "title": "string (max 100 chars)",
  "body": "string (max 500 chars, markdown supported)",
  "link": "string (optional, valid URL)",
  "metadata": { "key": "value" },
  "channels": ["in_app", "email", "webhook"]
}
```

**Response (201):**
```json
{
  "id": "uuid",
  "status": "queued",
  "channels": {
    "in_app": "sent",
    "email": "queued",
    "webhook": "skipped_by_preference"
  },
  "createdAt": "2026-01-15T10:30:00Z"
}
```

**Errors:**
- 400: Invalid payload (missing required fields, invalid type)
- 401: Missing or invalid API key
- 404: User not found
- 429: Rate limit exceeded (100 notifications/min per API key)

### GET /api/v1/notifications
Retrieve user's notification history.

**Query Parameters:**
- `userId` (required): UUID
- `type` (optional): Filter by notification type
- `read` (optional): true/false
- `since` (optional): ISO 8601 timestamp
- `page` (optional, default: 1): Page number
- `limit` (optional, default: 20, max: 100): Items per page

**Response (200):**
```json
{
  "data": [
    {
      "id": "uuid",
      "type": "info",
      "title": "New comment on your post",
      "body": "John replied to your post...",
      "link": "/posts/123#comment-456",
      "read": false,
      "createdAt": "2026-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 142,
    "hasMore": true
  }
}
```

### PATCH /api/v1/notifications/:id/read
Mark a single notification as read.

### PATCH /api/v1/notifications/read
Mark multiple notifications as read.

**Request:**
```json
{ "ids": ["uuid1", "uuid2"] }
```
Max 100 IDs per request.

### GET /api/v1/preferences/:userId
Retrieve user notification preferences.

### PUT /api/v1/preferences/:userId
Update user notification preferences.

**Request:**
```json
{
  "channels": {
    "in_app": { "enabled": true },
    "email": { "enabled": true, "frequency": "immediate | daily_digest" },
    "webhook": { "enabled": false, "url": null }
  },
  "muted_types": ["info"],
  "quiet_hours": { "start": "22:00", "end": "08:00", "timezone": "UTC" }
}
```

## Acceptance Criteria

- AC-1: POST /notifications with valid payload creates notification and returns 201
- AC-2: Notification is delivered to channels based on user preferences
- AC-3: If a channel is disabled in preferences, notification is skipped for that channel
- AC-4: Rate limit: 100 notifications/min per API key; 429 returned when exceeded
- AC-5: GET /notifications returns paginated results sorted by createdAt DESC
- AC-6: Quiet hours are respected — notifications queued, not delivered
- AC-7: PATCH /read marks notification as read; subsequent GET shows read=true
- AC-8: Invalid payloads return 400 with descriptive error messages

## Edge Cases
| Scenario | Expected Behavior |
|----------|------------------|
| Notification body exceeds 500 chars | Return 400: "Body must be 500 characters or fewer" |
| Email delivery fails | Retry 3 times with exponential backoff; mark as "failed" after |
| Webhook URL returns 5xx | Retry 3 times; mark as "failed"; log for monitoring |
| User has no preferences set | Use defaults: in_app=true, email=true, webhook=false |
| Bulk read with 101+ IDs | Return 400: "Maximum 100 IDs per request" |
| Notification for deleted user | Return 404: "User not found" |
| Concurrent read/unread toggle | Last write wins; no error |

## Non-Functional Requirements
- Throughput: 1,000 notifications/second sustained
- Latency: p95 < 200ms for POST, p95 < 100ms for GET
- Availability: 99.9% uptime SLA
- Data retention: 90 days, then archived to cold storage
- API versioning: URL-based (v1, v2, etc.)
```

---

## Why This Spec Works

1. **Complete API contract** — Developers can implement without guessing
2. **Specific error codes** — AI agents know exactly what to return
3. **Concrete limits** — Rate limits, pagination, character counts all specified
4. **Edge cases** — Every boundary condition has defined behavior
5. **NFRs with numbers** — "1,000/second" not "must be scalable"
