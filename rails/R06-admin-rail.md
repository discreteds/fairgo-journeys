# R06 — Admin Rail

**Purpose:** Admin moderation, people/group management, event configuration.
**Primary persona:** Event admin.

## Rail Path

```
S05 Event Dashboard
  │ "⚠️ Pending actions"
  │
  ├──→ S16 Admin Moderation (centralized queue)
  │       │
  │       ├── Role approval → inline [Approve] / [Reject]
  │       │     ⚡ POST /event-roles/{rid}/approve or /remove
  │       │
  │       ├── Transaction review → S10 Expense Detail
  │       │     ⚡ POST /transactions/{tid}/approve
  │       │
  │       └── Identity match → S07 Manage People
  │             ⚡ POST /persons/merge
  │
  ├──→ S07 Manage People
  │       │
  │       ├── Add placeholders
  │       │     ⚡ POST /persons
  │       ├── Reassign settlement groups (PFG)
  │       │     ⚡ PUT /persons/{pid}/pfg
  │       ├── Merge identity matches
  │       │     ⚡ POST /persons/merge
  │       └── Send personal invite → S06
  │
  ├──→ S08 Manage Groups
  │       │
  │       ├── Create consumption groups
  │       │     ⚡ POST /groups + POST /groups/{gid}/members
  │       └── Edit/archive groups
  │             ⚡ POST /groups/{gid}/archive
  │
  ├──→ S14 Event Funding
  │       │
  │       ├── Fund event
  │       │     ⚡ POST /events/{eid}/funding
  │       └── Spread costs
  │             ⚡ POST /events/{eid}/funding/cost-spread
  │
  ├──→ S16 Modification Requests (JF-3)
  │       │
  │       ├── View pending modification requests
  │       │     ⚡ GET /events/{eid}/modification-requests
  │       ├── Approve request → auto-apply changes
  │       │     ⚡ POST /events/{eid}/modification-requests/{mrid}/approve
  │       └── Reject request
  │             ⚡ POST /events/{eid}/modification-requests/{mrid}/reject
  │
  └──→ Audit Log (JF-6)
          │
          └── View event audit log
                ⚡ GET /events/{eid}/audit-log
```

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S05 → S16 | Pending items exist | Full moderation queue |
| S05 → S07 | Inline approve from pending banner | Quick action, stay on S05 |
| S16 → S10 | Review transaction | Expense detail with approve controls |
| S16 → S07 | Identity match action | People management with merge controls |
| S07 → S06 | Send personal invite | Invite generation |
| S05 → S16 | Admin views modification requests | Mod request queue (JF-3) |
| S16 → S16 | Approve/reject mod request | Auto-apply changes, refresh queue |
| S05 → Audit Log | Admin views event history | Audit log view (JF-6) |
| S16 → Audit Log | Admin views event history from moderation | Audit log view (JF-6) |

## Admin vs S05 Inline

S05 provides **inline** approve/reject for the most common admin action (role approvals). S16 is the **full queue** for when there are many pending items or the admin wants batch processing. Both call the same backend endpoints.

```
S05 inline:  Quick approve/reject (1-2 items)
S16 full:    Batch processing (many items, mixed types)
```

## Modification Request Handling (JF-3)

Members can submit modification requests (e.g., change an expense amount, update a split). These requests queue for admin review on S16.

```
S05 → S16 (mod requests tab)
  │
  ▼
S16 Admin Moderation → "Modification Requests" section
  │
  ├── View request details (who, what, why)
  │
  ├── [Approve] → auto-apply changes
  │     ⚡ POST /events/{eid}/modification-requests/{mrid}/approve
  │     → original transaction/split updated automatically
  │     → balances recalculated
  │
  └── [Reject] → request closed, no changes
        ⚡ POST /events/{eid}/modification-requests/{mrid}/reject
        → requester notified
```

Approved modifications are applied automatically — the admin does not need to manually edit the transaction.

## Audit Log Access (JF-6)

Admin can view the full event audit log from S05 or S16. The audit log records all significant actions (expense creation, settlements, role changes, modifications, etc.).

```
S05 → Audit Log view
  ⚡ GET /events/{eid}/audit-log

S16 → Audit Log view
  ⚡ GET /events/{eid}/audit-log
```

The audit log provides a chronological record of who did what and when, supporting transparency and dispute resolution.

## Scenarios Using This Rail

- SC02 (Bob Joins via Invite) — admin approves role
- SC03 (Family Holiday) — admin manages people and settlement groups
- SC07 (Charlie Claims Person) — admin reviews identity match
- SC08 (Member Permission Walls) — admin moderation of member requests
- SC12 (Dispute and Modification) — admin reviews and resolves modification requests
