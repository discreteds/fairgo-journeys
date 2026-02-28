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
| Add split-pending expense | S09 | 2 fields + save | `POST /transactions` with `splits_status: "pending"`, empty splits (1 call) |
| Add multi-item expense | S09 | Fill rows + save | Same single `POST /transactions` with array of line_items (1 call) |
| Edit expense | S10 | Edit form | `PUT /transactions` + `PUT /line-items` + split updates (3+ calls) |
| Approve expense | S10 | 1 tap | `POST /transactions/{tid}/approve` (1 call) |
| Cancel expense | S10 | Confirm + tap | `POST /transactions/{tid}/cancel` (1 call) |

### People Management

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Add person | S07 | Name + save | `POST /persons` (1 call, PFG auto-created, split-pending txns auto-assigned) |
| Add person (with pending txns) | S07 | Name + save + split preview | `POST /persons` → returns `PersonCreateResponse` with `affected_transactions` (1 call) |
| Merge persons | S07 | 1 tap | `POST /persons/merge` (1 call, transfers all data) |
| Self-merge ("This is me") | S07 | 1 tap | `POST /persons/merge` (1 call, same endpoint) |
| Change settlement group (existing) | S07 | Pick from list | `PUT /persons/{pid}/pfg` (1 call) |
| Change settlement group (new) | S07 | Name + pick | `POST /groups` + `PUT /persons/{pid}/pfg` (2 calls) |
| Adjust split (inline) | S07 | Tap expense row + adjust | `PUT /transactions/{tid}` with updated splits (1 call) |
| Send personal invite | S07→S06 | Pick person + create | Navigation to S06 Prepare & Share |

### Group Management

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Create group | S08 | Name + pick members | `POST /groups` + `POST /groups/{gid}/members` × N (1+N calls) |
| Edit group members | S08 | Toggle checkboxes | `POST` or `DELETE /groups/{gid}/members` per change |
| Archive group | S08 | Confirm + tap | `POST /groups/{gid}/archive` (1 call) |

### Prepare & Share (Invitation)

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Copy group link | S06 | 1 tap | 0 calls (code already exists from event creation) |
| Share via native sheet | S06 | 1 tap | 0 calls (uses existing code + Web Share API) |
| Create personal link | S06 | 1 tap per person | `POST /invite-codes` with target_person_id (1 call) |
| Create all personal links | S06 | 1 tap | `POST /invite-codes` × N (N calls, parallelised) |
| Add phone number | S06 | Inline field | `PUT /persons/{pid}` with phone_hint (1 call) |

### Settlement

| UI Action | Screen | Feels Like | Backend Calls |
|-----------|--------|-----------|---------------|
| Create settlement | S12 | 1 tap on suggestion | `POST /settlements` (1 call) |
| Confirm settlement | S12 | 1 tap | `POST /settlements/{sid}/confirm` (1 call) |
| Mark settlement paid | S12 | 1 tap | `POST /settlements/{sid}/pay` (1 call) |
| Void settlement | S12 | Confirm + tap | `POST /settlements/{sid}/void` (1 call) |

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
| S03 Home | 1 (`GET /events?include=my_position`) | N/A | Position embedded in event response (P1) |
| S05 Dashboard | 6 | Yes (5 parallel after event load) | Most critical — every journey passes through |
| S06 Prepare & Share | 2 | Yes | Persons + invite-codes |
| S07 People | 2 (persons with embedded PFG + groups) | Yes | PFG embedded in person response; `my-matches` for member view |
| S08 Groups | 2 | Yes | Lightweight |
| S10 Expense Detail | 1 (`GET /transactions/{tid}` with embedded line items + splits) | N/A | Full tree in single response (P2) |
| S11 Balances | 1 | N/A | Positions endpoint returns everything |
| S12 Settle Up | 2 | Yes | Positions + settlements |
| S13 Membership | 1 | N/A | Single endpoint |
| S14 Funding | 1-2 | Yes | Funding + membership (if unfunded) |
| S15 Profile | 2 | Yes | User + persons |
| S16 Admin | 3 × N events | Yes | Roles + txns + persons per event |

## Split-Pending Transaction Lifecycle

Orchestration for the split-pending feature introduced in the SC01 reflow:

```
1. CREATE (split-pending)
   POST /events/{eid}/transactions
     {splits_status: "pending", line_items: [{amount: 45, splits: []}]}
   → Transaction saved with no splits, splits_status = "pending"
   → No position impact (excluded from calculation)

2. AUTO-ASSIGN (on person creation)
   POST /events/{eid}/persons {display_name: "Bob"}
   → Backend queries all transactions where splits_status = "pending"
   → For each pending transaction:
     a. Get all active persons (including new person)
     b. Set event creator as expense-side (weight=1)
     c. Set all persons as consumption-side (weight=1 each)
     d. Calculate shares using largest remainder method
     e. Update splits_status to "assigned"
   → Response includes affected_transactions with new splits

3. MANUAL ADJUST (optional, from S07 inline preview)
   PUT /events/{eid}/transactions/{tid}
     {line_items: [{splits: [adjusted splits]}]}
   → Splits updated with custom weights
   → shares recalculated

4. POSITION UPDATE
   → After splits_status changes from "pending" to "assigned",
     positions endpoint includes the transaction in calculations
```

## Person Creation with Auto-Assign — Detailed Flow

```
POST /events/{eid}/persons {display_name: "Bob"}

Backend steps:
  1. Create person record (existing)
  2. Create singleton group (existing)
  3. Create PFG (existing)
  4. Query pending transactions:
     SELECT * FROM transactions
     WHERE event_id = {eid} AND splits_status = 'pending'
  5. For each pending transaction:
     a. Get all active persons in event
     b. For each line item:
        - Create expense split: event_creator, weight=1, modifier=1
        - Create consumption splits: all persons, weight=1, modifier=1
        - Calculate shares
     c. Set splits_status = "assigned"
  6. Return PersonCreateResponse:
     {person: PersonOut, affected_transactions: [TransactionOut, ...]}
```

## Backend API Gaps for UI Optimisation

1. ~~**Enriched events list**~~ — **RESOLVED (P1):** `GET /events?include=my_position` embeds position summaries. S03 loads with 1 call.
2. ~~**Enriched persons**~~ — **RESOLVED:** `_person_to_dict()` embeds PFG data. S07 loads with 2 calls.
3. ~~**Enriched transactions**~~ — **RESOLVED (P2):** `GET /transactions/{tid}` returns full tree (line items + computed splits). S10 loads with 1 call.
4. ~~**Activity feed endpoint**~~ — **RESOLVED (P3):** `GET /users/me/activity` returns paginated cross-event feed. S17 loads with 1 call.
5. **Bulk invite codes** — `POST /invite-codes/bulk` would reduce S06 "Create All Links" from N calls to 1. Deferred to post-MVP.
