# S16 — Admin Moderation

**Purpose:** Centralized queue for pending approvals across events.
**Visible to:** Admin only (per event).
**Rails:** R06 (Admin)
**Scenarios:** SC02

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
│  Identity Matches            │
│  ┌──────────────────────────┐│
│  │ "Dave" may be Dave K.    ││
│  │ who just joined          ││
│  │ Confidence: High (email) ││
│  │ [Merge]  [Dismiss]       ││
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
```

If the user is admin of multiple events, repeat for each event (parallelised).

## Orchestration — "Approve" Role

```
POST /events/{eid}/event-roles/{rid}/approve
→ role status: active
→ remove from queue
```

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

## Smart Defaults

- Items grouped by type (roles, transactions, identities) for easy scanning
- Badge count on S05 dashboard shows total pending items
- Inline actions for simple approvals — no need to navigate away
- Transaction reviews navigate to S10 for full detail
- Empty state is encouraging ("All caught up!")
- This screen is also reachable from the S05 "⚠️ Pending actions" banner

## Relationship to S05

S05 (Event Dashboard) shows a condensed version of this queue as the "⚠️ Pending actions" banner with inline approve/reject for roles. S16 is the full-page version for when there are many pending items or the admin wants to process them in batch.

## Error States

| Error | Display |
|-------|---------|
| Approve fails | "Couldn't approve. The request may have been withdrawn." → refresh |
| Merge fails | "Merge failed. The records may have changed." → refresh |
| No admin events | Screen not reachable (no admin events = no pending items) |
