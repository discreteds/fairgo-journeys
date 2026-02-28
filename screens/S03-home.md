# S03 — Home (Event List)

**Purpose:** Hub for all events. Entry point after login.
**Visible to:** All authenticated users.
**Rails:** R01 (Onboarding)
**Scenarios:** SC01, SC04

## Wireframe

```
┌─────────────────────────┐
│  Fair Go          ⚙️ 👤 │
├─────────────────────────┤
│                         │
│  Your Events            │
│  ┌─────────────────────┐│
│  │ Bali Trip           ││
│  │ You owe $142.50     ││
│  │ 3 unsettled         ││
│  └─────────────────────┘│
│  ┌─────────────────────┐│
│  │ Friday Dinner       ││
│  │ You're owed $23.00  ││
│  │ All settled ✓       ││
│  └─────────────────────┘│
│                         │
│  ┌───────────────────┐  │
│  │   + New Event      │  │
│  └───────────────────┘  │
│                         │
│  ┌───────────────────┐  │
│  │   Join with Code   │  │
│  └───────────────────┘  │
│                         │
│  [Events] [Activity] [Me]│
└─────────────────────────┘
```

## Wireframe — Empty State

```
┌─────────────────────────┐
│  Fair Go          ⚙️ 👤 │
├─────────────────────────┤
│                         │
│                         │
│   No events yet!        │
│                         │
│   Create an event to    │
│   start splitting       │
│   expenses with friends │
│                         │
│  ┌───────────────────┐  │
│  │   + New Event      │  │
│  └───────────────────┘  │
│                         │
│   Got an invite?        │
│  ┌───────────────────┐  │
│  │   Join with Code   │  │
│  └───────────────────┘  │
│                         │
│  [Events] [Activity] [Me]│
└─────────────────────────┘
```

## Orchestration — Page Load

```
1. GET /events?include=my_position   → event list with embedded position summaries
```

Each event includes a `my_position` object with `person_id`, `total_paid`, `total_consumed`, and `net`. This eliminates the per-event position call. For events where the user has no person, `my_position` is null.

## Orchestration — "+ New Event"

```
→ S04 (Create Event)
```

## Orchestration — "Join with Code"

```
1. Show inline input or modal for invite code
2. POST /events/join {invite_code: "..."}
3. If target_person_id on invite → auto-claim (no new person created)
4. → S05 (event dashboard, pending approval banner if applicable)
```

## Orchestration — Event Card Tap

```
→ S05 (Event Dashboard for tapped event)
```

## Smart Defaults

- Events sorted by most recent activity (updated_at)
- Net position shown as "You owe $X" (red) or "You're owed $X" (green) — derived from user's PFG net position
- "All settled ✓" badge when user's PFG net = $0.00
- Closed events shown at bottom with muted styling
- Ongoing events (`event_type: ongoing`) show running balance without "All settled" badge — they don't have a natural end point
- Singular events (`event_type: singular`) show completion badge when all settled
- Events with pending approval show "⏳ Awaiting approval" badge

## Error States

| Error | Display |
|-------|---------|
| Invalid invite code | "This code isn't valid or has expired" |
| Already a member | "You're already in this event" — navigate to S05 |
| Network error on load | Cached event list if available, "Pull to refresh" |
