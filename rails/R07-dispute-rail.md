# R07 — Dispute Rail

**Purpose:** Raise, review, and resolve modification requests (disputes).
**Primary persona:** Member (raise), admin (review and resolve).

## Rail Path

```
S10 Expense Detail (member view)
  │ "Dispute This →"
  ▼
S16 Admin Moderation
  │ Member submits modification request
  │ ⚡ POST /events/{eid}/modification-requests
  │     {type: "modify_split", suggested_weights: [...]}
  │
  ▼
S16 Admin Moderation (admin view)
  │ Admin reviews pending request
  │ ⚡ GET /events/{eid}/modification-requests
  │
  ├──── [Approve] → auto-apply changes
  │     ⚡ POST /events/{eid}/modification-requests/{mrid}/approve
  │     → original transaction/split updated automatically
  │     → balances recalculated
  │
  └──── [Reject] → request closed, no changes
        ⚡ POST /events/{eid}/modification-requests/{mrid}/reject
        → requester notified

  ▼
S10 Expense Detail (updated splits) / S05 Event Dashboard (updated balances)
```

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S10 → S16 | Member taps "Dispute This →" | Modification request form |
| S16 → S16 | Admin approves/rejects request | Queue refreshes, changes auto-applied |
| S16 → S10 | Admin taps to review transaction detail | Expense detail with context |
| S16 → S05 | Admin returns to dashboard | Updated balances reflect approved changes |

## Key Orchestration Sequence

```
POST /events/{eid}/modification-requests     # 1. Member raises dispute
GET  /events/{eid}/modification-requests     # 2. Admin views queue
POST /events/{eid}/modification-requests/{mrid}/approve  # 3. Approve → auto-apply
  OR
POST /events/{eid}/modification-requests/{mrid}/reject   # 3. Reject → notify
```

Approved modifications are applied automatically — the admin does not need to manually edit the transaction. The full 6-endpoint modification request API also supports `GET /{mrid}` (detail), and `POST /{mrid}/withdraw` (member self-cancels).

## Cross-References

- **S10 Expense Detail** — "Dispute This →" button visible to members
- **S16 Admin Moderation** — modification request queue (JF-3)
- **R06 Admin Rail** — S16 is also the entry point for R06; dispute handling is a sub-flow within the moderation queue

## Scenarios Using This Rail

- SC02 (Bob Joins via Invite) — Bob disputes wine split, Alice approves with 2:1 weight
- SC12 (Dispute and Modification) — full dispute lifecycle with approve and reject paths
