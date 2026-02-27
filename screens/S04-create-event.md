# S04 — Create Event

**Purpose:** Single-screen event creation with aggressive defaults.
**Visible to:** All authenticated users. Creator becomes admin.
**Rails:** R01 (Onboarding)
**Scenarios:** SC01, SC03, SC04

## Wireframe

```
┌─────────────────────────┐
│  ← New Event            │
├─────────────────────────┤
│                         │
│  Event Name    [______] │
│  Description   [______] │
│  Currency      [AUD ▾]  │
│                         │
│  ┌───────────────────┐  │
│  │   Create Event    │  │
│  └───────────────────┘  │
└─────────────────────────┘
```

## Orchestration — "Create Event"

```
1. POST /events {name, description, currency}
   → event created
   → user gets admin event_role (auto)
   → user's person auto-created with display_name from profile
2. POST /events/{eid}/invite-codes {max_uses: null}
   → default shareable invite code generated
3. → S05 (event dashboard)
```

The backend auto-creates the user's person on event creation. The frontend auto-generates a shareable invite code so the admin can immediately share it. Two calls, but the user only filled one form.

## Smart Defaults

| Field | Default | Override |
|-------|---------|----------|
| Event Name | (required, focused) | — |
| Description | (optional, empty) | Free text |
| Currency | AUD (locale-detected) | Dropdown of supported currencies |

- Single required field: event name
- Description is optional — most casual events don't need one
- Currency defaults to AUD based on locale; can be overridden but rarely is
- Creator's person uses their profile display_name automatically

## Error States

| Error | Display |
|-------|---------|
| Empty event name | Inline: "Give your event a name" |
| Network error | "Couldn't create event. Try again." with retry button |
