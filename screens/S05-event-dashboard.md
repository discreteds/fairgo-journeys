# S05 — Event Dashboard

**Purpose:** The hub screen. Everything about one event, role-aware.
**Visible to:** All event members (admin and member views differ).
**Rails:** R01, R02, R03, R04, R05, R06 (all rails pass through here)
**Scenarios:** All

This is the most important screen in the app. Every journey passes through it. Its load-time orchestration (7 parallel API calls) is the most performance-critical path.

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
│  │ Share Event →            ││
│  │ Invite Link          📋  ││
│  └──────────────────────────┘│
│                              │
│         ( + Add Expense )    │  ← FAB
│                              │
│  [Events] [Activity] [Me]   │
└──────────────────────────────┘
```

## Wireframe — Multi-Currency Balances (CR-002)

When an event contains transactions in multiple currencies, the balance section displays per-currency totals instead of a single aggregate number:

```
│  Your Balance                │
│  ┌──────────────────────────┐│
│  │  AUD  You owe $142.50    ││
│  │  USD  You're owed $23.00 ││
│  │  EUR  You owe €18.00     ││
│  │                          ││
│  │  ┌────────────────────┐  ││
│  │  │   Settle Up        │  ││
│  │  └────────────────────┘  ││
│  └──────────────────────────┘│
```

Each currency row follows the same colour conventions: red for "You owe", green for "You're owed". The "Settle Up" button is shown if the user owes in any currency. The summary stats row at the top also groups totals by currency when multiple currencies are present:

```
│  ┌────────┐┌────────┐┌─────┐│
│  │AUD $947││USD $300││  3  ││
│  │ total  ││ total  ││ txn ││
│  └────────┘└────────┘└─────┘│
```

> The positions endpoint (`GET /events/{eid}/positions`) returns positions keyed by currency. The frontend groups and renders one balance row per currency. Single-currency events display as before with no grouping header.

## Wireframe — Event Limits (Free Tier)

When an event is unfunded (free tier), the dashboard displays current usage against limits so users understand remaining capacity:

```
│  ┌──────────────────────────┐│
│  │  Free Tier Limits         ││
│  │  Persons:     4 / 5       ││
│  │  Expenses:    1 / 1       ││
│  │  Groups:      1 / 1       ││
│  │  Settlements: not available││
│  │                           ││
│  │  [Fund This Event →]      ││
│  └──────────────────────────┘│
```

Limits data comes from the event object's `limits` field, which includes `person_limit`, `transaction_limit`, `group_limit` and current usage counts. The limits section is hidden for funded events. For members (non-admin), the "Fund This Event" link is replaced with "Contact your event admin to unlock full features."

## Wireframe — Split-Pending Expenses

When transactions have `splits_status: "pending"`, they display with a pending indicator instead of payer info:

```
│  Recent Expenses             │
│  ┌──────────────────────────┐│
│  │ Pizza            $45.00  ││
│  │ ⏳ splits pending         ││
│  ├──────────────────────────┤│
│  │ Drinks           $30.00  ││
│  │ ⏳ splits pending         ││
│  └──────────────────────────┘│
```

Once people are added and splits are auto-assigned, these revert to the normal display showing payer and per-person amounts.

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
│  │ 📋 Audit Log         ▸   ││
│  └──────────────────────────┘│
```

> **Audit Log (JF-6):** The "Audit Log" link is visible to admin users only. It navigates to a view powered by `GET /events/{eid}/audit-log`, which returns a chronological list of all modifications, settlements, voids, and role changes for the event. Members do not see this link.

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

## Wireframe — Self-Merge Prompt (Member View)

When `GET /events/{eid}/persons/my-matches` returns a match for the current user:

```
|  +----------------------------+|
|  | We found a match!          ||
|  | "Dave" looks like you.     ||
|  | [This is me]  [Not me]     ||
|  +----------------------------+|
```

Tapping "This is me" calls `POST /events/{eid}/persons/{placeholder_id}/merge` with the user's person as target. `authorize_self_merge()` validates the match (email or display name).

## Wireframe — Voided Settlement Display (JF-1B, JF-5)

Settlements that have been voided display with audit fields showing who voided them and when:

```
│  Settlements                 │
│  ┌──────────────────────────┐│
│  │ Bob → Alice    $85.00    ││
│  │ ✅ Confirmed · Jan 15     ││
│  ├──────────────────────────┤│
│  │ Dave → Alice   $42.00    ││
│  │ ❌ Voided · Jan 18        ││
│  │ by Alice · "entered in   ││
│  │ error"                    ││
│  └──────────────────────────┘│
```

The `voided_at` and `voided_by` fields are returned on settlement objects from `GET /events/{eid}/settlements`. Voided settlements are visually distinct (strikethrough or muted styling with the void reason displayed). These fields provide audit transparency so all participants can see when and why a settlement was reversed.

## Wireframe — Closed Event (Admin View)

When `event.status = closed`, admin sees a read-only dashboard with a reopen option:

```
┌──────────────────────────────┐
│  ← Bali Trip           🔒   │
├──────────────────────────────┤
│                              │
│  ┌──────────────────────────┐│
│  │ 🔒 Event Closed           ││
│  │ This event has been       ││
│  │ closed. No new expenses   ││
│  │ or settlements can be     ││
│  │ created.                  ││
│  │                           ││
│  │ ┌────────────────────┐    ││
│  │ │   Reopen Event     │    ││  ← admin only
│  │ └────────────────────┘    ││
│  └──────────────────────────┘│
│                              │
│  (expenses and settlements   │
│   visible but read-only)     │
└──────────────────────────────┘
```

**Reopen Event** calls `POST /events/{eid}/reopen` (or `PUT /events/{eid}` with `{status: "active"}`). After reopening, the FAB returns, settlement creation is re-enabled, and the closed banner disappears. Only admin can reopen.

## Orchestration — Page Load

```
1. GET /events/{eid}                    → event metadata (name, currency, status, limits)
2. GET /events/{eid}/persons            → person list + count
3. GET /events/{eid}/transactions       → transaction list (recent)
4. GET /events/{eid}/positions          → PFG positions + checksum (grouped by currency)
5. GET /events/{eid}/settlements        → settlement list + count (includes voided_at/voided_by)
6. GET /events/{eid}/event-roles        → pending approvals (admin only)
7. GET /events/{eid}/persons/my-matches → match suggestions for current user (member only)

Calls 2-7 parallelised after call 1 returns event metadata.
```

Call 1 returns the event's `limits` field when the event is unfunded, containing `person_limit`, `transaction_limit`, `group_limit`, and current usage counts. This drives the free tier limits display. Call 4 returns positions keyed by currency for multi-currency balance rendering. Call 5 includes `voided_at` and `voided_by` audit fields on voided settlements.

## Actions

| Action | Who | Target | Orchestration |
|--------|-----|--------|--------------|
| + Add Expense (FAB) | All | → S09 | Navigation |
| Settle Up | All | → S12 | Pre-filled with user's PFG |
| Transaction row tap | All | → S10 | Navigation |
| People ▸ | All | → S07 | Navigation |
| Groups ▸ | All | → S08 | Navigation |
| Settlements ▸ | All | → S12 (list mode) | Navigation |
| Share Event → | Admin | → S06 | Navigation — Prepare & Share flow |
| Invite Link 📋 | All | Clipboard | Copy default invite URL (quick access) |
| Approve role (inline) | Admin | Stay on S05 | `POST /events/{eid}/event-roles/{rid}/approve` |
| Reject role (inline) | Admin | Stay on S05 | `POST /events/{eid}/event-roles/{rid}/remove` |
| This is me (self-merge) | Member | Stay on S05 | `POST /events/{eid}/persons/{source_id}/merge` |
| Review transaction | Admin | → S10 | Navigation |
| Event Settings ▸ | Admin | Edit modal | `PUT /events/{eid}` |
| Funding Status ▸ | Admin | → S14 | Navigation |
| Audit Log ▸ | Admin | Audit log view | `GET /events/{eid}/audit-log` |
| Reopen Event | Admin | Stay on S05 | `POST /events/{eid}/reopen` (closed events only) |

## Smart Defaults

- **"Your Balance"** card derived from current user's PFG net position
- **Transaction list** shows payer display_name + relative time ("2 days ago"), not raw timestamps
- **"Settle Up"** button only appears when user's PFG net < 0 (they owe money)
- **"You're owed"** variant appears when PFG net > 0 (green, no settle-up CTA)
- **"All settled"** badge when PFG net = $0.00
- **Split-pending expenses** show "⏳ splits pending" instead of payer info — transitions to normal display once splits are assigned
- **"Share Event →"** leads to S06 Prepare & Share — the recommended path for sharing with people attached
- **Invite link** always one tap to copy — quick access for admins who already know what they're doing
- **Pending actions** banner only shows for admins with pending items
- **Funding status** hidden unless event is unfunded and user is admin
- **Summary stats** (total, people, txn count) computed from loaded data — grouped by currency when the event has multiple currencies
- **Multi-currency balances** show one row per currency with independent owe/owed status and colour
- **Free tier limits** section shown only for unfunded events, with usage counts and a link to fund
- **Voided settlements** shown with strikethrough styling, void reason, and who voided them
- **Audit log** link shown for admins only, hidden for members

## Error States

| Error | Display |
|-------|---------|
| Event not found | "This event doesn't exist or you don't have access" → S03 |
| Pending approval | Read-only view with approval banner |
| Event closed | Read-only view with "Event closed" banner, no FAB, admin sees [Reopen Event] |
| Unfunded limit hit | "This event needs funding to continue" → link to S14 (admin) or message to contact admin (member) |
