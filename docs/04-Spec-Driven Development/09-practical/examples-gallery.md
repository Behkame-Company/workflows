# Examples Gallery

> Real-world specification examples across different project types and complexity levels.

---

## Example 1: Simple — Dark Mode Toggle

**Complexity**: Small | **Spec Time**: 5 minutes

```markdown
# Dark Mode Toggle

**What**: Users can switch between light and dark color themes.
**Why**: Reduce eye strain and support user preference.
**Who**: All users.

## Success Criteria
- [ ] Toggle button in header switches between light/dark mode
- [ ] Preference persists across page reloads (localStorage)
- [ ] Respects system preference on first visit (prefers-color-scheme)
- [ ] Transition between themes is smooth (200ms CSS transition)

## Edge Cases
- No localStorage available: Default to system preference
- User overrides system preference: User choice takes priority

## Not Included
- Custom color themes (only light and dark)
- Per-page theme settings
```

---

## Example 2: Medium — Search Autocomplete

**Complexity**: Medium | **Spec Time**: 20 minutes

```markdown
# Feature: Search Autocomplete

## Summary
Add real-time search suggestions that appear as users type, helping them
find products faster and discover related items.

## Acceptance Criteria

### Behavior
- AC-1: Given user types 2+ characters in search box
  When 300ms passes without new input (debounce)
  Then show up to 8 suggestions below the search box
- AC-2: Given suggestions are visible
  When user presses Up/Down arrow keys
  Then highlighted suggestion changes (keyboard navigation)
- AC-3: Given user presses Enter on a suggestion
  When suggestion is selected
  Then navigate to that product's page
- AC-4: Given user clicks outside the suggestion box
  When focus leaves
  Then suggestions disappear

### Data
- AC-5: Suggestions include: product name, category, thumbnail image
- AC-6: Suggestions sorted by: exact matches first, then popularity
- AC-7: Maximum 8 suggestions per query

### Performance
- AC-8: Suggestions appear within 200ms of API response
- AC-9: API response time: <100ms at p95

## Edge Cases
| Scenario | Behavior |
|----------|----------|
| No matching results | Show "No results for '[query]'" |
| Query is only spaces | Don't trigger search |
| API is down | Don't show suggestion box; search still works on Enter |
| HTML in query | Sanitize; no XSS via search input |
| Query with special characters | Escape for API, show literal in suggestions |

## Out of Scope
- Search history / recent searches
- Trending searches
- Category-level suggestions
```

---

## Example 3: Large — OAuth Integration

**Complexity**: Large | **Spec Time**: 45 minutes

```markdown
# Feature: Google OAuth Login

## Summary
Enable users to sign in with their Google account as an alternative to
email/password authentication, reducing registration friction.

## User Stories
- US-1: As a new user, I want to sign up with Google so I don't need
  to create another password.
- US-2: As an existing user (email), I want to link my Google account
  so I can use either login method.
- US-3: As a Google-only user, I want to be able to set a password later
  if I choose.

## Acceptance Criteria

### Registration
- AC-1.1: "Continue with Google" button on registration page
- AC-1.2: Google OAuth flow redirects to Google, then back
- AC-1.3: New user created with Google profile (name, email, avatar)
- AC-1.4: User lands on onboarding page after first Google login

### Account Linking
- AC-2.1: If Google email matches existing email account,
  prompt: "An account with this email exists. Link your Google account?"
- AC-2.2: User must verify existing password to link accounts
- AC-2.3: After linking, both login methods work

### Session
- AC-3.1: Google login creates same session as email login (30-day duration)
- AC-3.2: Logging out revokes local session (not Google session)

### Profile
- AC-4.1: Profile shows Google avatar if no custom avatar uploaded
- AC-4.2: Display name defaults to Google display name

## Edge Cases
| Scenario | Behavior |
|----------|----------|
| User denies Google permission | Return to login with "Google login was cancelled" |
| Google returns no email | Block registration; require email scope |
| Google profile has no avatar | Use default avatar |
| User deletes Google account later | Existing sessions continue; can set password in settings |
| Rate limit on OAuth flow | "Too many login attempts. Please try again in 5 minutes." |
| CSRF attack on callback | Validate state parameter; reject mismatched requests |

## Security Requirements
- PKCE (Proof Key for Code Exchange) required for OAuth flow
- State parameter validated on callback
- Google tokens stored encrypted, never in localStorage
- Refresh tokens rotated on each use

## Out of Scope
- GitHub OAuth (separate spec)
- Apple Sign In (separate spec)
- Account unlinking via UI (admin only for v1)
- Google Workspace (organization) login restrictions
```

---

## Example 4: API Spec — Webhook Delivery

**Complexity**: Medium | **Spec Time**: 25 minutes

```markdown
# Feature: Webhook Delivery System

## Summary
Send HTTP POST requests to customer-configured URLs when events occur
in the system. Reliable delivery with retry logic.

## Acceptance Criteria
- AC-1: POST request sent to configured URL within 5 seconds of event
- AC-2: Request includes event type, timestamp, and payload in JSON
- AC-3: Request includes HMAC-SHA256 signature header for verification
- AC-4: Failed deliveries retry: 1min, 5min, 30min, 2hr, 24hr (5 retries)
- AC-5: After 5 failures, webhook is disabled; email sent to owner
- AC-6: Webhook logs show delivery attempts, status codes, response times
- AC-7: Customers can test webhooks with a "Send test event" button

## Webhook Payload
```json
{
  "id": "evt_abc123",
  "type": "order.completed",
  "timestamp": "2026-01-15T10:30:00Z",
  "data": { ... event-specific data ... }
}

Headers:
  X-Webhook-Signature: sha256=<HMAC of body using webhook secret>
  X-Webhook-ID: evt_abc123
  Content-Type: application/json
```
```

---

## Pattern: Scaling Spec Detail with Complexity

```
Trivial:  Skip spec entirely
Simple:   5-line "lightweight spec" (Template 5)
Medium:   30-80 line spec with ACs and edge cases
Large:    Full spec with all sections
Epic:     Decompose into multiple medium specs
```

The examples above demonstrate this scaling in practice.
