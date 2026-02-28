# S04 — Create Event

**Purpose:** Single-screen event creation with aggressive defaults.
**Visible to:** All authenticated users. Creator becomes admin.
**Rails:** R01 (Onboarding)
**Scenarios:** SC01, SC03, SC04

## Wireframe

```
+-----------------------------+
|  <- New Event               |
+-----------------------------+
|                             |
|  Event Name    [______]     |
|  Description   [______]     |
|  Currency      [AUD v]      |
|  Type    (*) One-off        |
|          ( ) Ongoing        |
|                             |
|  +---------------------+   |
|  |   Create Event       |   |
|  +---------------------+   |
+-----------------------------+
```

## Orchestration — "Create Event"

```
1. POST /events {name, description, currency, event_type, visibility}
   -> event created (event_type defaults to "singular", visibility defaults to "public")
   -> user gets admin event_role (auto)
   -> user's person auto-created with display_name from profile
   -> default shareable invite code auto-generated
2. -> S05 (event dashboard)
```

The backend auto-creates the user's person and invite code on event creation. The frontend sends one call; the user filled one form.

## Smart Defaults

| Field | Default | Override |
|-------|---------|----------|
| Event Name | (required, focused) | — |
| Description | (optional, empty) | Free text |
| Currency | AUD (locale-detected) | Dropdown of supported currencies |
| Event Type | singular (One-off) | Toggle: One-off / Ongoing |
| Visibility | public | Not shown on creation — available in event settings |

- Single required field: event name
- Description is optional — most casual events don't need one
- Currency defaults to AUD based on locale; can be overridden but rarely is
- Creator's person uses their profile display_name automatically

## Error States

| Error | Display |
|-------|---------|
| Empty event name | Inline: "Give your event a name" |
| Network error | "Couldn't create event. Try again." with retry button |
