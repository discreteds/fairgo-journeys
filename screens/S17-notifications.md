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

The backend activity feed endpoint is implemented:

```
GET /users/me/activity?limit=50&offset=0
-> returns unified, chronological activity feed across all events the user participates in
```

Single call. Activities are recorded server-side when key actions occur (transaction creation, settlement status changes, person merges, event closures, role changes). Pagination via `limit` and `offset` query parameters.

### Superseded: Client-Side Derived (Approach A)

The original MVP approach of aggregating from N x 3 endpoint calls per event is no longer needed. The backend activity endpoint replaces it entirely.

## Activity Types

| Type | Backend `activity_type` | Template |
|------|------------------------|----------|
| Expense added | `transaction_created` | "{summary}" |
| Expense cancelled | `transaction_cancelled` | "{summary}" |
| Settlement confirmed | `settlement_confirmed` | "{summary}" |
| Settlement paid | `settlement_paid` | "{summary}" |
| Settlement voided | `settlement_voided` | "{summary}" |
| Person merged | `person_merged` | "{summary}" |
| Event closed | `event_closed` | "{summary}" |
| Role changed | `role_changed` | "{summary}" |

Each activity record includes: `id`, `user_id`, `event_id`, `activity_type`, `entity_type`, `entity_id`, `summary`, `created_at`.

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

## Implementation Status

Backend endpoint implemented (2026-02-28). `GET /users/me/activity` returns paginated activity feed. Activity records written on all key lifecycle events. Frontend integration pending.
