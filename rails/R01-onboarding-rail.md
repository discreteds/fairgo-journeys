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

## SC01 Extended Path (Capture First, Share Last)

For the primary scenario (Alice Organizes Dinner), the onboarding rail extends through expense entry, people management, and sharing:

```
S05 Event Dashboard
  │
  ├──── "+ Add Expense" ──→ S09 Add Expense (split-pending)
  │                          │
  │                          ▼
  │                        S05 (expense saved, splits pending)
  │
  ├──── "People" ─────────→ S07 Manage People (add + auto-assign splits)
  │                          │
  │                          ▼
  │                        S05 (balances visible)
  │
  └──── "Share Event →" ──→ S06 Prepare & Share
                             [transfer to R04 Invitation Rail]
```

Expenses and people are equally available from S05. The above order is recommended for first-time users but not enforced.

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S02 → S05 | User arrived with invite code | R04 (Invitation Rail — redeem side) |
| S05 → S09 | User taps FAB | R02 (Expense Rail) |
| S05 → S06 | Admin taps "Share Event →" | R04 (Invitation Rail — prepare & share side) |
| S05 → S07 | Admin taps "People" | People management (inline split preview) |
| S05 → S16 | Admin has pending actions | R06 (Admin Rail) |

## Key Orchestration Sequence

The "zero to first event" path triggers these backend calls:

```
POST /auth/register              # 1. Create account + free tier membership (auto)
POST /events                     # 2. Create event (auto: person + admin role)
POST /events/{eid}/invite-codes  # 3. Auto-generate invite link (NOT shared yet)
```

Three API calls. User filled two forms (register + event name). Everything else was automatic. The invite code is generated but not shared until the user reaches S06.

### Free Tier Auto-Creation (JF-4A)

Registration automatically creates a **free tier membership** for the new user. There is no tier selection step — free is the default. This means:

- The user immediately has access to create events within free tier limits
- No onboarding friction from tier/plan choices
- Upgrade paths are available later from account settings
- If the user joins an event where the admin has sponsorship enabled, they may receive paid-tier benefits without upgrading (see R04 Invitation Rail)

## Scenarios Using This Rail

- SC01 (Alice Organizes Dinner) — full path including expense → people → share
- SC02 (Bob Joins via Invite) — register → invite code branch
- SC03 (Family Holiday) — create event → add people → PFG setup
- SC04 (Housemates Monthly Bills) — create event → add people → multiple expenses
- SC07 (Charlie Claims Person) — register → join → claim identity
- SC09 (Multi-Event Power User) — create multiple events from S03
- SC18 (Weekend Away Multi-Payer) — create event → add people → multi-payer expenses
