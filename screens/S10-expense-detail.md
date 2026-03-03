# S10 — Expense Detail

**Purpose:** View/edit an existing transaction. See line items and per-person breakdown.
**Visible to:** All can view. Admin can edit/approve. Member can propose changes.
**Rails:** R02, R06
**Scenarios:** SC02, SC05, SC08, SC11, SC12, SC19

## Wireframe — View Mode

```
┌──────────────────────────────┐
│  ← Restaurant        [Edit] │
├──────────────────────────────┤
│  Alice paid · Feb 20        │
│  Occurred: Feb 19            │
│  Status: ✅ Approved         │
│                              │
│  Line Items                  │
│  ┌──────────────────────────┐│
│  │ Food              $480   ││
│  │ Split: Everyone · Equal  ││
│  │ Your share: $80.00       ││
│  ├──────────────────────────┤│
│  │ Alcohol            $140  ││
│  │ Split: Drinkers · Equal  ││
│  │ Your share: $35.00       ││
│  └──────────────────────────┘│
│                              │
│  Your Total: $115.00         │
│  Total:      $620.00         │
│                              │
│  Breakdown                   │
│  ┌──────────────────────────┐│
│  │ Alice   paid $620 · -$115││
│  │ Bob              · -$115 ││
│  │ Carol            · -$80  ││
│  │ Dave             · -$115 ││
│  │ Eve              · -$80  ││
│  │ Frank            · -$115 ││
│  └──────────────────────────┘│
│                              │
│  [Suggest Change]            │  ← member only
└──────────────────────────────┘
```

- **`Occurred`** displays the `occurred_at` field — when the expense actually happened. May differ from the creation date (`created_at`). Only shown when `occurred_at` is set and differs from the creation date.

## Wireframe — Multi-Currency Line Item

When a line item's currency differs from the event's base currency, the FX conversion is displayed inline:

```
│  ┌──────────────────────────┐│
│  │ Hotel              €420  ││
│  │ FX: 1 EUR = 1.62 AUD    ││
│  │ Converted: A$680.40      ││
│  │ Split: Everyone · Equal  ││
│  │ Your share: A$136.08     ││
│  └──────────────────────────┘│
```

- **Original amount and currency** shown on line 1
- **FX rate** (`fx_rate_used`) shown on line 2
- **Converted amount** in the event's base currency shown on line 3
- All per-person shares are displayed in the base currency

## Wireframe — Cancelled Expense

When an expense has been cancelled, audit fields are shown:

```
│  Status: ❌ Cancelled         │
│  Cancelled: Mar 1 by Alice   │
│  ┌──────────────────────────┐│
│  │ This expense has been    ││
│  │ cancelled and is excluded││
│  │ from balances.           ││
│  └──────────────────────────┘│
```

- **`Cancelled`** displays `cancelled_at` timestamp and `cancelled_by` user name
- Muted styling applied to the entire expense detail
- No edit or approve actions available on cancelled expenses

## Wireframe — Admin Pending Review

```
│  Status: ⏳ Pending Approval │
│  ┌──────────────────────────┐│
│  │ [Approve] [Request Changes]│
│  └──────────────────────────┘│
```

## Orchestration — Page Load

```
1. GET /events/{eid}/transactions/{tid}
   -> full transaction tree: metadata + line items + computed splits (shares, effective weights)
```

Single call. The transaction endpoint always returns embedded `line_items` with computed `splits` per line item.

## Orchestration — "Edit" (Admin)

Opens S09-style edit form pre-filled with existing data:

```
1. User modifies fields
2. PUT /events/{eid}/transactions/{tid}
     {description?, currency?, occurred_at?}
3. For modified line items:
   PUT /events/{eid}/line-items/{lid}
     {description?, amount?}
4. For modified splits:
   PATCH /events/{eid}/transactions/{tid}/line-items/{lid}/splits
     {splits: [{person_id, weight}, ...]}
   → dedicated split editing endpoint (replaces delete+create pattern)
5. Refresh detail view
```

## Orchestration — "Approve" (Admin)

```
1. POST /events/{eid}/transactions/{tid}/approve
2. Refresh status display → ✅ Approved
```

## Orchestration — "Cancel" (Admin)

```
1. Confirm dialog: "Cancel this expense? It will be excluded from balances but preserved for records."
2. POST /events/{eid}/transactions/{tid}/cancel
   -> status: cancelled
   -> positions recalculate (cancelled transactions excluded)
3. Refresh detail view -> shows cancelled badge
```

Cancelled transactions are soft-deleted (status-based). They are excluded from position calculations and list views by default but preserved for audit. Hard deletion is only available for already-cancelled transactions.

## Orchestration — "Suggest Change" (Member)

Members cannot edit expenses directly but can submit modification requests that enter the admin's moderation queue (S16).

```
1. Member taps [Suggest Change] on expense detail
2. Show modification request form:
   → Pre-filled with current transaction/line-item data
   → Member describes proposed changes and reason
3. POST /events/{eid}/modification-requests
     {
       type: "edit_expense",
       target_type: "transaction",
       target_id: tid,
       proposed_changes: {
         description?: "...",
         line_items?: [{lid, amount?, splits?}]
       },
       message: "Reason for the change"
     }
   → status: pending
   → "Change request sent to admin ✓"
4. Request appears in admin's S16 moderation queue
5. On admin approval: changes are auto-applied (no separate PATCH needed)
   → member is notified of approval
6. On admin rejection: member is notified with optional reason
```

## Orchestration — "Edit Splits" (Admin — Dedicated Endpoint)

For modifying how a line item is split among persons without changing other transaction fields:

```
1. Admin taps a line item's split breakdown → edit mode
2. Adjust weights or split method per person
3. PATCH /events/{eid}/transactions/{tid}/line-items/{lid}/splits
     {splits: [{person_id, weight}, ...]}
   → splits recalculated
   → positions update automatically
4. Refresh detail view
```

This dedicated endpoint replaces the previous pattern of deleting old splits and creating new ones. It atomically replaces all splits on a line item.

## Smart Defaults

- "Your share" highlighted per line item — users care about their own portion
- "Your Total" summed across all line items at the bottom
- Breakdown shows per-person net: paid minus consumed
- [Edit] button only visible to admin
- Members see a read-only view with [Suggest Change] button — submits a modification request to the admin's S16 moderation queue
- Status badge colour: ✅ green (approved), ⏳ amber (pending), ❌ red (cancelled)
- `occurred_at` date shown when it differs from creation date — helps distinguish when an expense happened vs. when it was recorded
- Multi-currency line items show original amount, FX rate (`fx_rate_used`), and converted amount in the event's base currency
- Cancelled expenses display `cancelled_at` and `cancelled_by` audit fields for transparency

## Error States

| Error | Display |
|-------|---------|
| Transaction not found | "This expense has been deleted" → S05 |
| Transaction cancelled | "This expense has been cancelled" with muted styling, no edit actions |
| Edit conflicts | "This expense was modified by someone else. Refresh to see changes." |
