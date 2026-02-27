# S05 — Event Dashboard

**Purpose:** The hub screen. Everything about one event, role-aware.
**Visible to:** All event members (admin and member views differ).
**Rails:** R01, R02, R03, R04, R05, R06 (all rails pass through here)
**Scenarios:** All

This is the most important screen in the app. Every journey passes through it. Its load-time orchestration (6 parallel API calls) is the most performance-critical path.

## Wireframe — Member View

```
┌──────────────────────────────┐
│  ← Bali Trip            ⚙️  │
├──────────────────────────────┤
│                              │
│  ┌────────┐┌────────┐┌─────┐│
│  │ $1,247 ││ 6      ││  3  ││
│  │ total  ││ people ││ txn ││
│  └────────┘└────────┘└─────┘│
│                              │
│  Your Balance                │
│  ┌──────────────────────────┐│
│  │  You owe $142.50         ││
│  │  ┌────────────────────┐  ││
│  │  │   Settle Up        │  ││
│  │  └────────────────────┘  ││
│  └──────────────────────────┘│
│                              │
│  Recent Expenses             │
│  ┌──────────────────────────┐│
│  │ Restaurant      $620.00  ││
│  │ Alice paid · 2 days ago  ││
│  ├──────────────────────────┤│
│  │ Airport Taxi     $85.00  ││
│  │ Bob paid · 3 days ago    ││
│  ├──────────────────────────┤│
│  │ Hotel           $542.00  ││
│  │ Alice paid · 5 days ago  ││
│  └──────────────────────────┘│
│                              │
│  ┌──────────────────────────┐│
│  │ People (6)           ▸   ││
│  │ Groups (2)           ▸   ││
│  │ Settlements (1)      ▸   ││
│  │ Invite Link          📋  ││
│  └──────────────────────────┘│
│                              │
│         ( + Add Expense )    │  ← FAB
│                              │
│  [Events] [Activity] [Me]   │
└──────────────────────────────┘
```

## Wireframe — Admin Extras

Admin sees everything above, plus:

```
│  ⚠️ 2 pending actions        │
│  ┌──────────────────────────┐│
│  │ Dave awaiting approval   ││
│  │ [Approve]  [Reject]      ││
│  ├──────────────────────────┤│
│  │ Bob's taxi needs review  ││
│  │ [Review]                 ││
│  └──────────────────────────┘│
│                              │
│  ┌──────────────────────────┐│
│  │ ⚙️ Event Settings    ▸   ││
│  │ 💳 Funding Status    ▸   ││
│  └──────────────────────────┘│
```

## Wireframe — Pending Approval State

When a member has joined but not yet been approved:

```
┌──────────────────────────────┐
│  ← Bali Trip                 │
├──────────────────────────────┤
│                              │
│  ┌──────────────────────────┐│
│  │ ⏳ Awaiting Approval      ││
│  │ The event admin needs to ││
│  │ approve your access.     ││
│  └──────────────────────────┘│
│                              │
│  (event details visible but  │
│   actions disabled)          │
└──────────────────────────────┘
```

## Orchestration — Page Load

```
1. GET /events/{eid}                 → event metadata (name, currency, status)
2. GET /events/{eid}/persons         → person list + count
3. GET /events/{eid}/transactions    → transaction list (recent)
4. GET /events/{eid}/positions       → PFG positions + checksum
5. GET /events/{eid}/settlements     → settlement list + count
6. GET /events/{eid}/event-roles     → pending approvals (admin only)

Calls 2-6 parallelised after call 1 returns event metadata.
```

## Actions

| Action | Who | Target | Orchestration |
|--------|-----|--------|--------------|
| + Add Expense (FAB) | All | → S09 | Navigation |
| Settle Up | All | → S12 | Pre-filled with user's PFG |
| Transaction row tap | All | → S10 | Navigation |
| People ▸ | All | → S07 | Navigation |
| Groups ▸ | All | → S08 | Navigation |
| Settlements ▸ | All | → S12 (list mode) | Navigation |
| Invite Link 📋 | All | Clipboard | Copy default invite URL |
| Approve role (inline) | Admin | Stay on S05 | `POST /events/{eid}/event-roles/{rid}/approve` |
| Reject role (inline) | Admin | Stay on S05 | `POST /events/{eid}/event-roles/{rid}/remove` |
| Review transaction | Admin | → S10 | Navigation |
| Event Settings ▸ | Admin | Edit modal | `PUT /events/{eid}` |
| Funding Status ▸ | Admin | → S14 | Navigation |

## Smart Defaults

- **"Your Balance"** card derived from current user's PFG net position
- **Transaction list** shows payer display_name + relative time ("2 days ago"), not raw timestamps
- **"Settle Up"** button only appears when user's PFG net < 0 (they owe money)
- **"You're owed"** variant appears when PFG net > 0 (green, no settle-up CTA)
- **"All settled"** badge when PFG net = $0.00
- **Invite link** always one tap to copy — no need to navigate to a management screen
- **Pending actions** banner only shows for admins with pending items
- **Funding status** hidden unless event is unfunded and user is admin
- **Summary stats** (total, people, txn count) computed from loaded data

## Error States

| Error | Display |
|-------|---------|
| Event not found | "This event doesn't exist or you don't have access" → S03 |
| Pending approval | Read-only view with approval banner |
| Event closed | Read-only view with "Event closed" banner, no FAB |
| Unfunded limit hit | "This event needs funding to continue" → link to S14 (admin) or message to contact admin (member) |
