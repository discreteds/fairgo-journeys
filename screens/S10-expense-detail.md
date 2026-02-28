# S10 — Expense Detail

**Purpose:** View/edit an existing transaction. See line items and per-person breakdown.
**Visible to:** All can view. Admin can edit/approve. Member can propose changes.
**Rails:** R02, R06
**Scenarios:** SC05

## Wireframe — View Mode

```
┌──────────────────────────────┐
│  ← Restaurant        [Edit] │
├──────────────────────────────┤
│  Alice paid · Feb 20        │
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
└──────────────────────────────┘
```

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
     {description?, currency?}
3. For modified line items:
   PUT /events/{eid}/line-items/{lid}
     {description?, amount?}
4. For modified splits:
   (delete old splits + create new ones, or update weights)
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

## Smart Defaults

- "Your share" highlighted per line item — users care about their own portion
- "Your Total" summed across all line items at the bottom
- Breakdown shows per-person net: paid minus consumed
- [Edit] button only visible to admin
- Members see a read-only view (or "Suggest Change" for modification requests — future feature)
- Status badge colour: ✅ green (approved), ⏳ amber (pending), ❌ red (cancelled)

## Error States

| Error | Display |
|-------|---------|
| Transaction not found | "This expense has been deleted" → S05 |
| Transaction cancelled | "This expense has been cancelled" with muted styling, no edit actions |
| Edit conflicts | "This expense was modified by someone else. Refresh to see changes." |
