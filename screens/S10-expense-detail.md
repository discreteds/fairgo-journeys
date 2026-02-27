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
   → transaction metadata (description, date, status, currency)
2. GET /events/{eid}/transactions/{tid}/line-items
   → line item list
3. For each line item (parallelised):
   GET /events/{eid}/line-items/{lid}/splits
   → per-person splits (expense + consumption sides)
```

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

## Orchestration — "Delete" (Admin)

```
1. Confirm dialog: "Delete this expense? This will update everyone's balances."
2. DELETE /events/{eid}/transactions/{tid}
3. → S05 (event dashboard)
```

## Smart Defaults

- "Your share" highlighted per line item — users care about their own portion
- "Your Total" summed across all line items at the bottom
- Breakdown shows per-person net: paid minus consumed
- [Edit] button only visible to admin
- Members see a read-only view (or "Suggest Change" for modification requests — future feature)
- Status badge colour: ✅ green (approved), ⏳ amber (pending), ❌ red (rejected)

## Error States

| Error | Display |
|-------|---------|
| Transaction not found | "This expense has been deleted" → S05 |
| Edit conflicts | "This expense was modified by someone else. Refresh to see changes." |
