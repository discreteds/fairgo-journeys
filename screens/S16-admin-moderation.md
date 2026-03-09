# S16 — Admin Moderation

**Purpose:** Centralized queue for pending approvals across events.
**Visible to:** Admin only (per event).
**Rails:** R06 (Admin)
**Scenarios:** SC02, SC08, SC12

## Wireframe

```
┌──────────────────────────────┐
│  ← Pending Actions           │
├──────────────────────────────┤
│                              │
│  Role Approvals              │
│  ┌──────────────────────────┐│
│  │ Dave wants to join       ││
│  │ Bali Trip                ││
│  │ [Approve]  [Reject]      ││
│  └──────────────────────────┘│
│                              │
│  Transaction Reviews         │
│  ┌──────────────────────────┐│
│  │ Bob's Taxi - $85.00      ││
│  │ Bali Trip                ││
│  │ [Review ▸]               ││
│  └──────────────────────────┘│
│                              │
│  Modification Requests        │
│  ┌──────────────────────────┐│
│  │ ✉ Eve requests:          ││
│  │ Add Person — "Frank"     ││
│  │ "Need to add Frank who   ││
│  │  shared the taxi"         ││
│  │ Submitted: Mar 1, 2:30pm ││
│  │ [Approve]  [Reject]      ││
│  ├──────────────────────────┤│
│  │ ✉ Bob requests:          ││
│  │ Edit Expense — "Dinner"  ││
│  │ "Split should be 7:7:1   ││
│  │  not equal"               ││
│  │ Submitted: Mar 1, 1:15pm ││
│  │ [Review ▸]               ││
│  └──────────────────────────┘│
│                              │
│  Identity Matches            │
│  ┌──────────────────────────┐│
│  │ "Dave" may be Dave K.    ││
│  │ who just joined          ││
│  │ Confidence: High (email) ││
│  │ [Merge]  [Dismiss]       ││
│  └──────────────────────────┘│
│                              │
│  ┌──────────────────────────┐│
│  │ [View Audit Log ▸]       ││
│  └──────────────────────────┘│
│                              │
│  ── No more pending ──      │
│  All caught up! ✓            │
└──────────────────────────────┘
```

## Wireframe — Empty State

```
┌──────────────────────────────┐
│  ← Pending Actions           │
├──────────────────────────────┤
│                              │
│                              │
│    All caught up! ✓          │
│    No pending actions.       │
│                              │
│                              │
└──────────────────────────────┘
```

## Wireframe — Modification Request Detail

When an admin taps "Review" on a modification request with proposed changes:

```
┌──────────────────────────────┐
│  ← Modification Request      │
├──────────────────────────────┤
│                              │
│  Requester: Bob              │
│  Type: Edit Expense          │
│  Submitted: Mar 1, 1:15pm   │
│                              │
│  Target: "Restaurant Dinner" │
│                              │
│  Proposed Changes:           │
│  ┌──────────────────────────┐│
│  │ Wine & Beer split:       ││
│  │ Current: 1:1:1           ││
│  │ Proposed: 7:7:1          ││
│  └──────────────────────────┘│
│                              │
│  Bob's message:              │
│  "I only had one beer (~$8). │
│   Alice and Dave split the   │
│   wine bottles."             │
│                              │
│  [Preview Impact]            │
│                              │
│  ┌──────────────────────────┐│
│  │ [Approve]    [Reject]    ││
│  └──────────────────────────┘│
└──────────────────────────────┘
```

Each modification request shows:
- **Requester** — who submitted the request
- **Type** — `add_person`, `edit_expense`, `edit_split`, etc.
- **Proposed changes** — structured diff of what would change
- **Message** — free-text reason from the requester
- **Timestamp** — when the request was submitted
- **Admin actions** — Approve (auto-applies changes) or Reject (with optional reason)

## Orchestration — Page Load

Draws from multiple API sources per event:

```
1. GET /events/{eid}/event-roles
   → filter where status = pending_approval
2. GET /events/{eid}/transactions
   → filter where status = pending (not yet approved)
3. GET /events/{eid}/persons
   → check for potential_matches in person data
   (or future: GET /events/{eid}/persons/matches)
4. GET /events/{eid}/modification-requests
   → filter where status = pending
   → shows all pending modification requests from members
```

If the user is admin of multiple events, repeat for each event (parallelised).

## Orchestration — "Approve" Role

```
POST /events/{eid}/event-roles/{rid}/approve
→ role status: active
→ remove from queue
```

> **Auto-approve targeted invites (CR-021):** When a member joins via a person-targeted invite (one with `target_person_id`), their EventRole is created with `status: "active"` directly — they skip the pending approval queue entirely. The admin moderation queue only shows members who joined via general invite codes.

## Orchestration — "Reject" Role

```
POST /events/{eid}/event-roles/{rid}/remove
→ role removed
→ remove from queue
```

## Orchestration — "Review" Transaction

```
→ S10 (Expense Detail for that transaction, with approve/reject controls)
```

## Orchestration — "Merge" Identity

```
POST /events/{eid}/persons/merge
  {source_person_id: placeholder_id, target_person_id: matched_person_id}
→ merge completed
→ remove from queue
```

## Orchestration — "Dismiss" Identity Match

Client-side only — mark as dismissed in local state. The match won't be shown again for this session. (Future: backend endpoint to permanently dismiss.)

## Modification Request Queue

Members cannot perform certain actions directly (adding persons, editing others' expenses). Instead they submit modification requests that enter this queue. The full API lifecycle:

### Orchestration — List Modification Requests

```
GET /events/{eid}/modification-requests
→ returns all modification requests for the event
→ filterable by status: pending, approved, rejected, withdrawn
```

#### Status Filter (CR-021)

The modification request list now supports filtering by status via query parameter:

```
GET /events/{eid}/modifications?status=pending&page=1&page_size=20
```

Valid values: `pending`, `approved`, `rejected`, `withdrawn`. Default (no filter) returns all. The UI shows a segmented control or tab bar above the list: **All | Pending | Approved | Rejected**.

### Orchestration — View Modification Request Detail

```
GET /events/{eid}/modification-requests/{id}
→ returns full request detail including proposed_changes and message
```

#### Preview Impact (CR-021)

The modification request detail view includes a "Preview Impact" button that calls the new preview endpoint:

```
GET /events/{eid}/modifications/{mid}/preview
→ 200: {
    current_state: { amount, splits, ... },
    proposed_state: { amount, splits, ... },
    position_deltas: [ { person_id, person_name, delta } ]
  }
```

The preview renders as a side-by-side diff below the proposed changes section. `position_deltas` shows how each person's balance would change (e.g., "Alice: +$12.50, Bob: -$12.50"). This helps the admin understand the financial impact before approving.

### Orchestration — Create Modification Request (Member)

Initiated from other screens (S07, S10) — not directly from S16:

```
POST /events/{eid}/modification-requests
  {
    type: "add_person" | "edit_expense" | "edit_split" | ...,
    target_type?: "transaction" | "line_item_split" | ...,
    target_id?: resource_id,
    proposed_changes: { ... },
    message?: "Reason for the request"
  }
→ status: pending
→ appears in admin's S16 queue
```

### Orchestration — "Approve" Modification Request

Approval auto-applies the proposed changes. No separate PATCH call needed.

```
POST /events/{eid}/modification-requests/{id}/approve
→ status: approved
→ proposed changes are automatically applied to the target resource
→ requester is notified: "Your request was approved ✓"
→ remove from pending queue
```

### Orchestration — "Reject" Modification Request

```
POST /events/{eid}/modification-requests/{id}/reject
  {reason?: "Explanation for the rejection"}
→ status: rejected
→ requester is notified with optional reason
→ no changes applied
→ remove from pending queue
```

### Orchestration — "Withdraw" Modification Request (Requester)

The original requester can withdraw their own pending request:

```
POST /events/{eid}/modification-requests/{id}/withdraw
→ status: withdrawn
→ removed from pending queue
→ no changes applied
```

## Orchestration — "View Audit Log" (Admin)

Admin-only access to the full event audit log, showing all significant actions:

```
GET /events/{eid}/audit-log
→ returns chronological list of all significant event actions
→ includes: who performed the action, what changed, when
→ covers: expense creation/editing/cancellation, person changes,
   role changes, settlement actions, modification request resolutions
```

Accessible from the `[View Audit Log ▸]` link at the bottom of the moderation queue.

## Smart Defaults

- Items grouped by type (roles, transactions, modification requests, identities) for easy scanning
- Badge count on S05 dashboard shows total pending items (includes modification requests)
- Inline actions for simple approvals — no need to navigate away
- Transaction reviews navigate to S10 for full detail
- Modification requests with complex proposed changes open a detail view with impact preview
- Approval auto-applies changes — admin doesn't need to manually PATCH the target resource
- Requesters can withdraw pending requests before admin action
- Empty state is encouraging ("All caught up!")
- This screen is also reachable from the S05 "⚠️ Pending actions" banner
- Audit log provides full transparency into all event actions (admin-only)

## Relationship to S05

S05 (Event Dashboard) shows a condensed version of this queue as the "⚠️ Pending actions" banner with inline approve/reject for roles. S16 is the full-page version for when there are many pending items or the admin wants to process them in batch.

## Error States

| Error | Display |
|-------|---------|
| Approve fails | "Couldn't approve. The request may have been withdrawn." → refresh |
| Merge fails | "Merge failed. The records may have changed." → refresh |
| No admin events | Screen not reachable (no admin events = no pending items) |
