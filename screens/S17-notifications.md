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
GET /users/me/activity?limit=50&offset=0&since=2026-01-15T00:00:00Z
→ returns unified, chronological activity feed across all events the user participates in
```

Single call. Activities are recorded server-side when key actions occur (transaction creation, settlement status changes, person merges, event closures, role changes, modification requests). Pagination via `limit` and `offset` query parameters.

**`since` parameter (JF-4B):** Optional ISO 8601 timestamp. When provided, only activities created after the given timestamp are returned. This is used to fetch new activity since the user's last visit, powering the unread badge count on the Activity tab. Example: `?since=2026-01-15T00:00:00Z` returns only activities from January 15, 2026 onwards.

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
| Mod request created | `modification_request_created` | "{summary}" |
| Mod request approved | `modification_request_approved` | "{summary}" |
| Mod request rejected | `modification_request_rejected` | "{summary}" |
| Event funded | `event_funded` | "{summary}" |
| Person added | `person_added` | "{summary}" |
| Person removed | `person_removed` | "{summary}" |

> **Phase 4B additions (JF-4B):** The `modification_request_created`, `modification_request_approved`, `modification_request_rejected`, `event_funded`, `person_added`, and `person_removed` types were added to cover the full lifecycle of event modifications and membership changes.

Each activity record includes: `id`, `user_id`, `event_id`, `event_name`, `activity_type`, `entity_type`, `entity_id`, `summary`, `created_at`.

> **Event name field (JF-4B):** Each activity item now includes `event_name` directly in the response. This allows the frontend to display event context (e.g. "Bali Trip") without needing a separate lookup. Activities can be grouped or filtered by event name in the UI.

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
