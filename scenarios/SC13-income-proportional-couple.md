# SC13 — Mel & Jake: Income-Proportional Household

**Persona:** Mel (admin) — higher earner in an established couple. She and Jake split shared costs 60/40 based on income.
**Screens:** S05 → S09 (×3) → S11 → S12
**Rails:** R02 (Expense), R03 (Settlement)

## Design Principle

**Fair doesn't mean equal — it means correct.** Mel earns more than Jake. They've agreed that shared expenses should be proportional to income: Mel covers 60%, Jake covers 40%. This isn't charity — it's their agreed definition of fair. Fair Go makes this expressible as a weight ratio (3:2), not a per-expense manual calculation.

## The Situation

Mel and Jake are an established couple sharing household costs. They're not first-time users — the event ("March Household") is already set up with both people added. Over the month, different expenses arrive and different people pay them, but the 3:2 ratio stays constant.

| Expense | Amount | Paid By | Ratio |
|---------|--------|---------|-------|
| Rent | $2,400.00 | Mel (direct debit from her account) | 3:2 (60/40) |
| Electricity | $185.00 | Jake (his name on the bill) | 3:2 (60/40) |
| Groceries | $320.00 | Mel (her card at the supermarket) | 3:2 (60/40) |

Total shared costs: **$2,905.00**

## People and Settlement Groups

```
S05 Event Dashboard — "March Household"
    People:
    🔴 Mel (you, admin)  — singleton PFG
    🔴 Jake              — singleton PFG

    Note: Mel and Jake are a couple but settle INDEPENDENTLY.
    They don't share a bank account — Mel pays from hers, Jake from his.
    Two people, two credit cards → two singleton PFGs.
    This is NOT the SC03 shared-PFG pattern.
```

## Entering Expenses with 3:2 Weights

### Expense 1: Rent — Mel pays, split 3:2

```
S05 → taps "+ Add Expense"
S09 Add Expense
    Description: "Rent"
    Amount: $2,400.00
    Paid by: You (Mel)

    Line item: "Rent" $2,400.00
    → Split: Mel & Jake (both selected by default)
    → Mel taps the split row to adjust weights
    → Weight picker:
        Mel:  3
        Jake: 2

    Split preview:
    ┌────────────────────────────────────────────────┐
    │  Mel:   $1,440.00  (3/5 = 60%)                 │
    │  Jake:    $960.00  (2/5 = 40%)                 │
    │  Total: $2,400.00  ✓                           │
    └────────────────────────────────────────────────┘

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Rent",
        line_items: [{
          description: "Rent", amount: "2400.00",
          payer_person_id: mel_id,
          splits: [
            {person_id: mel_id,  side: "expense",      weight: 1},
            {person_id: mel_id,  side: "consumption",  weight: 3},
            {person_id: jake_id, side: "consumption",  weight: 2}
          ]
        }]}

S05 → "Rent $2,400.00 — Mel paid"
```

### Expense 2: Electricity — Jake pays, split 3:2

Jake's name is on the electricity bill, so he is the payer this month. Mel must still re-enter the 3:2 weights — there is no automatic carry-forward from the previous expense.

```
S05 → taps "+ Add Expense"
S09 Add Expense
    Description: "Electricity"
    Amount: $185.00
    Paid by: Jake  ← Mel changes the payer from the default (herself)

    Line item: "Electricity" $185.00
    → Split: Mel & Jake
    → Mel taps the split row to adjust weights
    → Weight picker:
        Mel:  3
        Jake: 2

    Split preview:
    ┌────────────────────────────────────────────────┐
    │  Mel:   $111.00  (3/5 = 60%)                   │
    │  Jake:   $74.00  (2/5 = 40%)                   │
    │  Total:  $185.00  ✓                            │
    └────────────────────────────────────────────────┘

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Electricity",
        line_items: [{
          description: "Electricity", amount: "185.00",
          payer_person_id: jake_id,
          splits: [
            {person_id: jake_id, side: "expense",      weight: 1},
            {person_id: mel_id,  side: "consumption",  weight: 3},
            {person_id: jake_id, side: "consumption",  weight: 2}
          ]
        }]}

S05 → "Electricity $185.00 — Jake paid"
```

### Expense 3: Groceries — Mel pays, split 3:2

```
S05 → taps "+ Add Expense"
S09 Add Expense
    Description: "Groceries"
    Amount: $320.00
    Paid by: You (Mel)

    Line item: "Groceries" $320.00
    → Split: Mel & Jake
    → Mel taps the split row to adjust weights
    → Weight picker:
        Mel:  3
        Jake: 2

    Split preview:
    ┌────────────────────────────────────────────────┐
    │  Mel:   $192.00  (3/5 = 60%)                   │
    │  Jake:  $128.00  (2/5 = 40%)                   │
    │  Total:  $320.00  ✓                            │
    └────────────────────────────────────────────────┘

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Groceries",
        line_items: [{
          description: "Groceries", amount: "320.00",
          payer_person_id: mel_id,
          splits: [
            {person_id: mel_id,  side: "expense",      weight: 1},
            {person_id: mel_id,  side: "consumption",  weight: 3},
            {person_id: jake_id, side: "consumption",  weight: 2}
          ]
        }]}

S05 → "Groceries $320.00 — Mel paid"
```

## Balances

```
S11 Balances
    ⚡ GET /events/{eid}/positions/persons

    Expense breakdown:
    ┌────────────────────────────────────────────────────────────────┐
    │  Rent $2,400        Mel paid    Mel: $1,440.00  Jake: $960.00  │
    │  Electricity $185   Jake paid   Mel:   $111.00  Jake:  $74.00  │
    │  Groceries $320     Mel paid    Mel:   $192.00  Jake: $128.00  │
    └────────────────────────────────────────────────────────────────┘

    Net positions:
    ┌────────────────────────────────────────────────────────────────┐
    │  🔴 Mel     is owed $977.00                                    │
    │     paid $2,720.00, consumed $1,743.00                         │
    │     (rent $1,440 + electricity $111 + groceries $192)          │
    │                                                                │
    │  🔴 Jake    owes $977.00                                       │
    │     paid $185.00, consumed $1,162.00                           │
    │     (rent $960 + electricity $74 + groceries $128)             │
    │                                                                │
    │  Checksum: $0.00 ✓                                             │
    └────────────────────────────────────────────────────────────────┘
```

## Arithmetic Verification

| Expense | Amount | Mel consumed (60%) | Jake consumed (40%) | Mel paid | Jake paid |
|---------|--------|--------------------|---------------------|----------|-----------|
| Rent | $2,400.00 | $1,440.00 | $960.00 | $2,400.00 | $0.00 |
| Electricity | $185.00 | $111.00 | $74.00 | $0.00 | $185.00 |
| Groceries | $320.00 | $192.00 | $128.00 | $320.00 | $0.00 |
| **Totals** | **$2,905.00** | **$1,743.00** | **$1,162.00** | **$2,720.00** | **$185.00** |

| Person | Consumed | Paid | Net |
|--------|----------|------|-----|
| Mel | $1,743.00 | $2,720.00 | **−$977.00** (is owed) |
| Jake | $1,162.00 | $185.00 | **+$977.00** (owes) |
| **Σ** | **$2,905.00** | **$2,905.00** | **$0.00 ✓** |

## Settlement

```
S12 Settle Up
    ⚡ GET /events/{eid}/settlements/suggested

    Suggested:
    ┌────────────────────────────────────────────────┐
    │  🔴 Jake pays $977.00 → 🔴 Mel                 │
    └────────────────────────────────────────────────┘

    One payment. End of month. Done.

    → Jake confirms → taps "Mark as Paid"
    ⚡ POST /events/{eid}/settlements
       {from_pfg_id: jake_pfg, to_pfg_id: mel_pfg,
        amount: "977.00", status: "pending"}
    ⚡ PATCH /events/{eid}/settlements/{sid}
       {status: "paid"}

    S11 Balances → all positions return to $0.00
```

## The Differentiator Moment

With 3:2 weights applied consistently, Jake pays exactly 40% of every shared cost regardless of who paid that month. The system nets his $185 electricity payment against his $1,162 total consumption share and produces a single transfer amount — no spreadsheet, no manual reconciliation.

In a spreadsheet, Mel would recalculate 60/40 on every line. In Fair Go, the weights are structural — a property of the split, not a per-entry calculation.

## Design Consideration: Per-Expense Weight Re-Entry

This scenario exposes a UX friction point: Mel must re-enter the 3:2 weights on every expense. For a couple with a fixed ongoing arrangement, this is repetitive.

Potential future improvements:

- **Event-level split template** — "All expenses in this event default to 60/40 between Mel and Jake" — the most aligned option with the event model
- **"Same as last split" shortcut** — copy weights from the previous transaction
- **Group-level default weights** — if a group exists for Mel and Jake, the group stores their default ratio

None of these are in the current spec. This scenario makes the need visible and provides a concrete use case for the event-level template feature noted in `02.principles/i.futures/`.

## Contrast with SC03 (Shared PFG)

| | SC03: Couple with shared account | SC13: Mel & Jake |
|---|---|---|
| **Relationship** | Couple | Couple |
| **Bank accounts** | Shared (one card) | Separate (two cards) |
| **PFG setup** | Shared PFG | Singleton PFGs (default) |
| **Settlement** | Combined — one payment for the pair | Independent — Jake pays Mel directly |
| **Split ratio** | Equal | 60/40 (income-proportional) |
| **"How many credit cards?"** | One | Two |

Same relationship type, completely different financial configuration. Fair Go does not assume how couples manage money — it supports both patterns without special-casing either.

## Validates

- **Weighted splits (non-equal)** — 3:2 ratio applied consistently across all three expenses
- **Integer weights mapping to percentages** — user enters "3" and "2", system computes 60% / 40%; no decimal input required
- **Mixed payers with asymmetric weights** — Mel pays most, Jake pays one; weights apply regardless of who is payer
- **Net settlement across multiple transactions** — three expenses collapse to one payment
- **Settlement minimization** — single transfer clears all positions
- **Singleton PFGs for a couple** — deliberately contrasts SC03 (shared PFG); same relationship, different financial arrangement
- **Checksum verified** — $0.00 across all net positions
