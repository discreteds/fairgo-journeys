# R02 — Expense Rail

**Purpose:** Adding, viewing, and reviewing expenses.
**Primary persona:** All event members.

## Rail Path

```
S05 Event Dashboard
  │
  ├──── (no people yet) ───→ S07 Manage People
  │                            │
  │                            └──→ return to S05
  ▼
S09 Add Expense
  │
  ▼
S05 Event Dashboard (updated balances)
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
| S09 → S07 | Event has ≤1 person | Detour to add people, return |
| S11 → S12 | User taps "Settle Up" | R03 (Settlement Rail) |
| S10 → S09 | Admin taps "Edit" | Edit mode (same screen flow) |

## Key Orchestration Sequence

The "add a simple expense" path:

```
POST /events/{eid}/transactions    # Single call creates everything:
  {                                # - transaction envelope
    line_items: [{                 # - line item
      splits: [...]                # - all splits (expense + consumption)
    }]
  }
```

One API call. User filled two fields (description + amount). Everything else was aggressive defaults.

## Scenarios Using This Rail

- SC01 (Alice Organizes Dinner) — add expense after adding people
- SC02 (Bob Joins via Invite) — member adds expense
- SC04 (Housemates Monthly Bills) — multiple expenses by different payers
- SC06 (Free User Hits Limits) — expense blocked, then succeeds after funding
