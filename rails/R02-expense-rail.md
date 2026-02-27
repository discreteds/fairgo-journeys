# R02 — Expense Rail

**Purpose:** Adding, viewing, and reviewing expenses.
**Primary persona:** All event members.

## Rail Path

```
S05 Event Dashboard
  │
  ▼
S09 Add Expense
  │
  ├──── (≥2 people) ──→ saved with splits (splits_status: "assigned")
  │
  ├──── (≤1 person) ──→ saved without splits (splits_status: "pending")
  │                      → splits auto-assigned when people added on S07
  ▼
S05 Event Dashboard (updated balances or "splits pending" indicator)
  │
  ▼ (tap transaction)
S10 Expense Detail
  │
  ▼
S11 Balances
  [transfer to R03 Settlement]
```

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S09 → S05 | Expense saved (with or without splits) | Return to dashboard |
| S11 → S12 | User taps "Settle Up" | R03 (Settlement Rail) |
| S10 → S09 | Admin taps "Edit" | Edit mode (same screen flow) |

## Key Orchestration Sequence

The "add a simple expense" path (with people):

```
POST /events/{eid}/transactions    # Single call creates everything:
  {                                # - transaction envelope
    line_items: [{                 # - line item
      splits: [...]                # - all splits (expense + consumption)
    }]
  }
```

One API call. User filled two fields (description + amount). Everything else was aggressive defaults.

The "add expense before people" path (split-pending):

```
POST /events/{eid}/transactions    # Single call, no splits:
  {
    splits_status: "pending",
    line_items: [{
      amount: 45.00,
      splits: []                   # Empty — auto-assigned when people added
    }]
  }
```

Still one API call. Same two fields. Splits deferred until people exist.

## Scenarios Using This Rail

- SC01 (Alice Organizes Dinner) — add expense before people (split-pending), then auto-assigned
- SC02 (Bob Joins via Invite) — member adds expense (people already exist)
- SC04 (Housemates Monthly Bills) — multiple expenses by different payers
- SC06 (Free User Hits Limits) — expense blocked, then succeeds after funding
