# A01 — Orchestration Reference

Every place in the UI where a single user action triggers one or more backend calls.

## Principle

> The UI should feel like a simple app. The backend is complex. The orchestration layer
> bridges the gap — translating simple gestures into correct API call sequences.

## Single-Action Orchestrations

### Event Lifecycle

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Create Event | S04 | 1 form, 1 tap | `POST /events` + `POST /invite-codes` (2 calls) |
| Join via Invite (new user) | S02 | Register + auto-join | `POST /auth/register` + `POST /events/join` (2 calls) |
| Join via Invite (existing user) | S03 | Enter code | `POST /events/join` (1 call) |
| Join via Personal Invite | S02/S03 | Same as above | `POST /events/join` (1 call, auto-claim built in) |
| Close Event | S05 | Confirm + tap | `POST /events/{eid}/close` (1 call) |

### Expense Entry

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Add simple expense | S09 | 2 fields + save | `POST /transactions` with nested line_items + splits (1 call) |
| Add multi-item expense | S09 | Fill rows + save | Same single `POST /transactions` with array of line_items (1 call) |
| Edit expense | S10 | Edit form | `PUT /transactions` + `PUT /line-items` + split updates (3+ calls) |
| Approve expense | S10 | 1 tap | `POST /transactions/{tid}/approve` (1 call) |
| Delete expense | S10 | Confirm + tap | `DELETE /transactions/{tid}` (1 call) |

### People Management

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Add person | S07 | Name + save | `POST /persons` (1 call, PFG auto-created) |
| Merge persons | S07 | 1 tap | `POST /persons/merge` (1 call, transfers all data) |
| Self-merge ("This is me") | S07 | 1 tap | `POST /persons/merge` (1 call, same endpoint) |
| Change settlement group (existing) | S07 | Pick from list | `PUT /persons/{pid}/pfg` (1 call) |
| Change settlement group (new) | S07 | Name + pick | `POST /groups` + `PUT /persons/{pid}/pfg` (2 calls) |
| Send personal invite | S07→S06 | Pick person + create | `POST /invite-codes` with target_person_id (1 call) |

### Group Management

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Create group | S08 | Name + pick members | `POST /groups` + `POST /groups/{gid}/members` × N (1+N calls) |
| Edit group members | S08 | Toggle checkboxes | `POST` or `DELETE /groups/{gid}/members` per change |
| Delete group | S08 | Confirm + tap | `DELETE /groups/{gid}` (1 call) |

### Settlement

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Create settlement | S12 | 1 tap on suggestion | `POST /settlements` (1 call) |
| Confirm settlement | S12 | 1 tap | `POST /settlements/{sid}/confirm` (1 call) |
| Mark settlement paid | S12 | 1 tap | `POST /settlements/{sid}/pay` (1 call) |

### Funding

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Fund with per-event fee | S14 | Select + tap | `POST /events/{eid}/funding` (1 call) |
| Fund with subscription slot | S14 | Select type + tap | `POST /events/{eid}/funding` (1 call) |
| Spread costs | S14 | 1 tap | `POST /events/{eid}/funding/cost-spread` (1 call) |

### Membership

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Upgrade (new subscription) | S13 | 1 tap | `POST /memberships` (1 call) |
| Upgrade (change tier) | S13 | 1 tap | `PUT /memberships/me` (1 call) |
| Cancel subscription | S13 | Confirm + tap | `PUT /memberships/me {tier: "free"}` (1 call) |

### Admin

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Approve role (inline) | S05 | 1 tap | `POST /event-roles/{rid}/approve` (1 call) |
| Reject role (inline) | S05 | 1 tap | `POST /event-roles/{rid}/remove` (1 call) |

## Page Load Orchestrations

| Screen | Calls | Can Parallelise? | Notes |
|--------|-------|-----------------|-------|
| S03 Home | 1 + N (events + positions per event) | Yes (positions parallel) | Most expensive for users with many events |
| S05 Dashboard | 6 | Yes (5 parallel after event load) | Most critical — every journey passes through |
| S07 People | 3 + N (persons + PFG per person + groups) | Yes | Could benefit from enriched persons endpoint |
| S08 Groups | 2 | Yes | Lightweight |
| S10 Expense Detail | 3 + N (txn + line items + splits per item) | Yes | Could benefit from enriched transaction endpoint |
| S11 Balances | 1 | N/A | Positions endpoint returns everything |
| S12 Settle Up | 2 | Yes | Positions + settlements |
| S13 Membership | 1 | N/A | Single endpoint |
| S14 Funding | 1-2 | Yes | Funding + membership (if unfunded) |
| S15 Profile | 2 | Yes | User + persons |
| S16 Admin | 3 × N events | Yes | Roles + txns + persons per event |

## Backend API Gaps for UI Optimisation

These are not bugs but potential future optimisations:

1. **Enriched events list** — `GET /events` with embedded position summaries would eliminate N+1 on S03
2. **Enriched persons** — `GET /persons` with embedded PFG data would eliminate N+1 on S07
3. **Enriched transactions** — `GET /transactions/{tid}` with embedded line items and splits would reduce S10 to 1 call
4. **Activity feed endpoint** — `GET /users/me/activity` would make S17 viable without N×3 calls
