# S17 — Notifications / Activity

**Purpose:** Activity feed and pending action indicators.
**Visible to:** All authenticated users.
**Rails:** — (passive screen, accessible via bottom nav)
**Scenarios:** —

## Wireframe

```
┌──────────────────────────────┐
│  Activity                    │
├──────────────────────────────┤
│                              │
│  Today                       │
│  ┌──────────────────────────┐│
│  │ Alice added Restaurant   ││
│  │ $620 · Bali Trip        ││
│  │ 2 hours ago              ││
│  ├──────────────────────────┤│
│  │ You were approved for    ││
│  │ Bali Trip                ││
│  │ 3 hours ago              ││
│  └──────────────────────────┘│
│                              │
│  Yesterday                   │
│  ┌──────────────────────────┐│
│  │ Bob settled $85.00 with  ││
│  │ Frank · Bali Trip        ││
│  │ Yesterday                ││
│  └──────────────────────────┘│
│                              │
│  This Week                   │
│  ┌──────────────────────────┐│
│  │ You joined Bali Trip     ││
│  │ 3 days ago               ││
│  ├──────────────────────────┤│
│  │ Alice created Bali Trip  ││
│  │ 5 days ago               ││
│  └──────────────────────────┘│
│                              │
│  [Events] [Activity] [Me]   │
└──────────────────────────────┘
```

## Wireframe — Empty State

```
┌──────────────────────────────┐
│  Activity                    │
├──────────────────────────────┤
│                              │
│                              │
│    No activity yet.          │
│    Join or create an event   │
│    to get started.           │
│                              │
│                              │
│  [Events] [Activity] [Me]   │
└──────────────────────────────┘
```

## Data Source

**The backend does not currently have a notifications/activity endpoint.** This screen must be implemented using one of two approaches:

### Approach A: Client-Side Derived (MVP)

Aggregate activity from existing endpoints and sort by timestamp:

```
For each event the user belongs to:
  GET /events/{eid}/transactions    → new expenses (created_at)
  GET /events/{eid}/settlements     → settlement activity (created_at, updated_at)
  GET /events/{eid}/event-roles     → role changes (created_at, updated_at)
```

**Pros:** No backend changes needed.
**Cons:** Expensive (N events × 3 calls), no push notifications, may miss some activity types.

### Approach B: Backend Activity Endpoint (Future)

```
GET /users/me/activity?since={timestamp}&limit=50
→ returns unified activity feed across all events
```

**Pros:** Single call, efficient, supports push notifications.
**Cons:** Requires new backend endpoint + activity logging infrastructure.

**Recommendation:** Start with Approach A for MVP. Add backend endpoint when activity feed becomes a priority.

## Activity Types

| Type | Icon | Template |
|------|------|----------|
| Expense added | 💰 | "{person} added {description} ${amount}" |
| Expense approved | ✅ | "{description} was approved" |
| Settlement created | 📤 | "{from} settling ${amount} with {to}" |
| Settlement confirmed | ✅ | "Settlement confirmed: {from} → {to}" |
| Settlement paid | 💰 | "{from} paid ${amount} to {to}" |
| Role approved | 🎉 | "You were approved for {event}" |
| Person joined | 👋 | "{person} joined {event}" |
| Event created | 🆕 | "{person} created {event}" |

## Actions

| Action | Target |
|--------|--------|
| Activity item tap | → S05 (relevant event dashboard) |

## Smart Defaults

- Grouped by time period: Today, Yesterday, This Week, Earlier
- Relative timestamps ("2 hours ago" not "14:32")
- Events shown as context on each item
- Tapping any item navigates to the relevant event
- Badge count on bottom nav tab shows unread items (since last visit)

## Implementation Priority

This is a **low-priority screen** for MVP. The S05 Event Dashboard already surfaces the most important activity (recent expenses, pending actions) per event. S17 adds cross-event visibility but isn't critical for core workflows. Consider deferring to post-MVP.
