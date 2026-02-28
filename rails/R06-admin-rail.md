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
  └──→ S14 Event Funding
          │
          ├── Fund event
          │     ⚡ POST /events/{eid}/funding
          └── Spread costs
                ⚡ POST /events/{eid}/funding/cost-spread
```

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S05 → S16 | Pending items exist | Full moderation queue |
| S05 → S07 | Inline approve from pending banner | Quick action, stay on S05 |
| S16 → S10 | Review transaction | Expense detail with approve controls |
| S16 → S07 | Identity match action | People management with merge controls |
| S07 → S06 | Send personal invite | Invite generation |

## Admin vs S05 Inline

S05 provides **inline** approve/reject for the most common admin action (role approvals). S16 is the **full queue** for when there are many pending items or the admin wants batch processing. Both call the same backend endpoints.

```
S05 inline:  Quick approve/reject (1-2 items)
S16 full:    Batch processing (many items, mixed types)
```

## Scenarios Using This Rail

- SC02 (Bob Joins via Invite) — admin approves role
- SC03 (Family Holiday) — admin manages people and settlement groups
