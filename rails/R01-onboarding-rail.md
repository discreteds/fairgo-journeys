# R01 — Onboarding Rail

**Purpose:** First-time user from landing to first event.
**Primary persona:** New user (admin or member).

## Rail Path

```
S01 Welcome
  │
  ▼
S02 Register / Login
  │
  ├──── (has invite code) ────────→ S05 Event Dashboard
  │                                  [transfer to R02, R04]
  │                                  (pending approval state)
  ▼
S03 Home
  │
  ├──── (has events) → tap event → S05 Event Dashboard
  │                                  [transfer to R02]
  ▼
S04 Create Event
  │
  ▼
S05 Event Dashboard
  [transfer to R02 Expense, R04 Invitation, R06 Admin]
```

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S02 → S05 | User arrived with invite code | R04 (Invitation Rail — redeem side) |
| S05 → S09 | User taps FAB | R02 (Expense Rail) |
| S05 → S06 | Admin shares invite | R04 (Invitation Rail — generate side) |
| S05 → S16 | Admin has pending actions | R06 (Admin Rail) |

## Key Orchestration Sequence

The "zero to first event" path triggers these backend calls:

```
POST /auth/register              # 1. Create account
POST /events                     # 2. Create event (auto: person + admin role)
POST /events/{eid}/invite-codes  # 3. Auto-generate invite link
```

Three API calls. User filled two forms (register + event name). Everything else was automatic.

## Scenarios Using This Rail

- SC01 (Alice Organizes Dinner) — full path
- SC02 (Bob Joins via Invite) — register → invite code branch
