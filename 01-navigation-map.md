# Navigation Map

Master view of how all screens connect and which journey rails traverse them.

## Screen Graph

```
                            S01 Welcome
                                │
                    ┌───────────┼───────────┐
                    │           │           │
                "Get Started" "I have    "Log In"
                    │        a code"       │
                    ▼           │           ▼
                S02 Register ◄──┘    S02 Login
                    │                      │
          ┌─────────┼──────────────────────┘
          │         │
    (has invite)  (no invite)
          │         │
          ▼         ▼
    S05 Dashboard  S03 Home
          ▲         │
          │    ┌────┼────┐
          │    │         │
          │ "+ New"   "Join"
          │    │         │
          │    ▼         │
          │  S04 Create  │
          │    │         │
          └────┴─────────┘

                S05 Event Dashboard (HUB)
                    │
    ┌───────┬───────┼───────┬───────┬───────┬───────┐
    │       │       │       │       │       │       │
    ▼       ▼       ▼       ▼       ▼       ▼       ▼
  S09     S07     S08     S06     S11    S14     S16
  Add     People  Groups  Prepare  Bal.   Fund   Admin
  Expense   │             & Share   │             │
    │       │               │       │             │
    ▼       ▼               │       ▼             ▼
  S05     S05               │     S12           S10
  (back)  (back)            │     Settle        Expense
                            │     Up            Detail
                            ▼
                          S05
                          (back)

    Bottom Nav: [S03 Events] [S17 Activity] [S15 Profile]
                                                │
                                                ▼
                                              S13
                                            Membership
```

## Screen Inventory

| ID | Screen | Purpose | Entry Points |
|----|--------|---------|-------------|
| S01 | Welcome | First-time landing | App launch (unauthenticated) |
| S02 | Register/Login | Authentication | S01, invite deep link |
| S03 | Home | Event list + quick actions | Post-login, bottom nav |
| S04 | Create Event | New event form | S03 "+ New Event" |
| S05 | Event Dashboard | Event hub (role-aware) | S03 event tap, S04 post-create, S02 post-join |
| S06 | Prepare & Share | Person checklist + share invite codes | S05 "Share Event →", S07 "Send Personal Invite" |
| S07 | Manage People | Person list, identity resolution, PFG, inline split preview | S05 "People" |
| S08 | Manage Groups | Consumption + settlement groups | S05 "Groups" |
| S09 | Add Expense | Single-screen expense entry (supports split-pending) | S05 FAB |
| S10 | Expense Detail | View/edit transaction + line items | S05 transaction tap, S16 review |
| S11 | Balances | Who owes whom (positions) | S05, bottom of expense flow |
| S12 | Settle Up | Create/confirm/pay settlements | S11 "Settle Up", S05 "Settle Up" |
| S13 | Membership | Tier management + quotas | S15 "Membership" |
| S14 | Event Funding | Fund event + cost spread | S05 admin "Funding Status" |
| S15 | Profile/Settings | User account management | Bottom nav "Me" |
| S16 | Admin Moderation | Centralized approval queue | S05 "Pending actions" |
| S17 | Activity | Activity feed | Bottom nav "Activity" |

## Journey Rails Overview

| Rail | Path | Primary Persona |
|------|------|----------------|
| R01 Onboarding | S01 → S02 → S03 → S04 → S05 | New user |
| R02 Expense | S05 → S09 → S05 → S10 → S11 | All members |
| R03 Settlement | S11 → S12 → S11 → S05 | All members + admin |
| R04 Invitation | S05 → S06 ··· S01 → S02 → S05 | Admin (prepare & share) + invitee (receive) |
| R05 Membership | S15 → S13 → S05 → S14 | Admin (funding) |
| R06 Admin | S05 → S16 → S07/S08/S10/S14 | Admin |

## SC01 Flow (Capture First, Share Last)

The primary scenario follows this sequence through the screens:

```
S01 → S02 → S03 → S04 → S05 → S09 → S05 → S07 → S05 → S06
                                 │      │      │      │      │
                              event   expense people  review  share
                              created (pending)(add+   splits
                                              split)
```

Expenses and people are equally accessible from S05 in either order. The above is the recommended first-time flow, but returning users may add people before expenses.

## Cross-Validation Matrix

| Screen | Scenarios | Rails | Backend Calls on Load |
|--------|-----------|-------|----------------------|
| S01 Welcome | SC01, SC02 | R01 | 0 |
| S02 Register/Login | SC01, SC02 | R01 | 1 (auth) |
| S03 Home | SC01, SC04 | R01 | 2 (events + positions) |
| S04 Create Event | SC01, SC03, SC04 | R01 | 0 |
| S05 Event Dashboard | ALL | R01-R06 | 6 (parallel) |
| S06 Prepare & Share | SC01, SC02 | R04 | 2 (persons + invite-codes) |
| S07 Manage People | SC01, SC03 | R02, R06 | 3 |
| S08 Manage Groups | SC03 | R06 | 2 |
| S09 Add Expense | SC01, SC02, SC04, SC06 | R02 | 0 (pre-loaded) |
| S10 Expense Detail | SC05 | R02, R06 | 3 |
| S11 Balances | SC03, SC04, SC05 | R02, R03 | 1 |
| S12 Settle Up | SC03, SC04, SC05 | R03 | 1 (positions) |
| S13 Membership | SC06 | R05 | 1 |
| S14 Event Funding | SC06 | R05, R06 | 1 |
| S15 Profile | — | R05 | 2 |
| S16 Admin Moderation | SC02 | R06 | 3 |
| S17 Activity | — | — | derived (no endpoint yet) |

### Observations

- **S05 is the hub** — every rail and scenario passes through it. Its 6 parallel API calls are the most performance-critical path.
- **S06 now serves SC01** — expanded from invite-only to "Prepare & Share" with person checklist, making it part of the primary admin flow.
- **S09 supports split-pending** — no longer requires people before expense entry, enabling the "capture first" principle.
- **S07 shows inline split preview** — person creation response now includes affected transactions, shown inline without extra navigation.
- **S17 (Activity)** isn't referenced by any scenario or rail. Could be deferred or merged into S05 as an activity tab.
- **S15 (Profile)** is utility — low priority for initial implementation.
