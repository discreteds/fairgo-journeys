# SC15 — Complex Multi-Payer Restaurant (The Big Lunch)

> **Variation of SC11.** Extends the restaurant-bill pattern with multi-payer expense splitting, 13 participants across 5 PFGs, and line-item subgroup splits.

**Persona:** Alice (admin) organises a group lunch; she and Isla split the bill 50/50 as co-payers.
**Screens:** S05 → S07 → S09 → S10 → S11 → S12
**Rails:** R02 (Expense), R03 (Settlement)

## Design Principle

**"Two cards, one bill, fair shares."**

Alice and Isla split the restaurant bill down the middle — but that doesn't mean everyone consumed equally. Kids had cheaper meals, only some adults drank. Fair Go tracks who paid (multi-payer) separately from who consumed (line-item splits), so the final settlement reflects both.

## Participants (13 people, 5 PFGs)

| Person | PFG | Drinks? |
|--------|-----|---------|
| Alice (admin, co-payer) | Family 1 🔵 | No |
| Ben | Family 1 🔵 | Yes |
| Child1 | Family 1 🔵 | No |
| Child2 | Family 1 🔵 | No |
| Child3 | Family 1 🔵 | No |
| Carol | Family 2 🟢 | Yes |
| Dave | Family 2 🟢 | Yes |
| Child4 | Family 2 🟢 | No |
| Eve | Couple 1 🟡 | Yes |
| Frank | Couple 1 🟡 | Yes |
| Grace | Couple 2 🟣 | Yes |
| Henry | Couple 2 🟣 | No |
| Isla (co-payer) | Isla 🔴 | Yes |

## Narrative

### 1. Event and people setup

Alice's event "Big Lunch" already has 13 people and 5 PFGs configured via S07.

### 2. Adding the multi-payer expense with line items

Alice creates a single transaction with 3 line items and 2 co-payers:

```
S05 → taps "+ Add Expense"
S09 Add Expense → "The Big Lunch"
    → taps "Add Line Items" (multi-line mode)
    → Paid by: taps "Multiple..."
```

**Payer selection — the "Multiple..." flow:**

```
S09 → Paid by: "Multiple..."
    ┌─────────────────────────────────────────────────────┐
    │  Who paid?                                          │
    │                                                     │
    │  ☑ Alice    $375.50                                 │
    │  ☑ Isla     $375.50                                 │
    │                                                     │
    │  Split method: Equal (50/50)                        │
    │  Bill total: $751.00                                │
    │                                        [Confirm]    │
    └─────────────────────────────────────────────────────┘
```

Both Alice and Isla are marked as payers with equal weight on every line item (`side: "expense"`, `weight: 1` each). The system calculates each payer's share: $751.00 ÷ 2 = $375.50 each.

**Line Item 1: Kids Meals — $156.00, 4 kids equally**

```
S09 → Line Item: "Kids Meals" / $156.00
    → Split between: Child1, Child2, Child3, Child4
    → Weights: equal (1:1:1:1)
    → Each child: $39.00
```

**Line Item 2: Adult Meals — $378.00, 9 adults equally**

```
S09 → Line Item: "Adult Meals" / $378.00
    → Split between: Alice, Ben, Carol, Dave, Eve, Frank, Grace, Henry, Isla
    → Weights: equal (1:1:1:1:1:1:1:1:1)
    → Each adult: $42.00
```

**Line Item 3: Drinks — $217.00, 7 drinkers equally**

```
S09 → Line Item: "Drinks" / $217.00
    → Split between: Ben, Carol, Dave, Eve, Frank, Grace, Isla
    → Weights: equal (1:1:1:1:1:1:1)
    → Each drinker: $31.00
    → Alice, Child1–4, Henry excluded (non-drinkers)
```

**Saving the transaction:**

```
S09 → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "The Big Lunch",
        line_items: [
          {description: "Kids Meals", amount: 156.00,
           expense_splits: [{person_id: alice, weight: 1, side: "expense"},
                            {person_id: isla, weight: 1, side: "expense"}],
           consumption_splits: [{person_id: child1, weight: 1},
                                {person_id: child2, weight: 1},
                                {person_id: child3, weight: 1},
                                {person_id: child4, weight: 1}]},
          {description: "Adult Meals", amount: 378.00,
           expense_splits: [{person_id: alice, weight: 1, side: "expense"},
                            {person_id: isla, weight: 1, side: "expense"}],
           consumption_splits: [{person_id: alice, weight: 1},
                                {person_id: ben, weight: 1},
                                {person_id: carol, weight: 1},
                                {person_id: dave, weight: 1},
                                {person_id: eve, weight: 1},
                                {person_id: frank, weight: 1},
                                {person_id: grace, weight: 1},
                                {person_id: henry, weight: 1},
                                {person_id: isla, weight: 1}]},
          {description: "Drinks", amount: 217.00,
           expense_splits: [{person_id: alice, weight: 1, side: "expense"},
                            {person_id: isla, weight: 1, side: "expense"}],
           consumption_splits: [{person_id: ben, weight: 1},
                                {person_id: carol, weight: 1},
                                {person_id: dave, weight: 1},
                                {person_id: eve, weight: 1},
                                {person_id: frank, weight: 1},
                                {person_id: grace, weight: 1},
                                {person_id: isla, weight: 1}]}
        ]}
```

Key: `expense_splits` with `side: "expense"` on both Alice and Isla for every line item means the payment is split 50/50 between them, regardless of how consumption is split.

### 3. Expense detail view

```
S10 Expense Detail
    The Big Lunch — Total: $751.00
    Paid by: Alice ($375.50) + Isla ($375.50)

    ┌──────────────────────────────────────────────────────┐
    │ Kids Meals    $156.00   4 kids × $39.00              │
    │ Adult Meals   $378.00   9 adults × $42.00            │
    │ Drinks        $217.00   7 drinkers × $31.00          │
    └──────────────────────────────────────────────────────┘

    Per-person totals:
    Alice:   consumed $42.00   (paid $375.50 → owed $333.50)
    Ben:     consumed $73.00   (owes $73.00)
    Child1:  consumed $39.00   (owes $39.00)
    Child2:  consumed $39.00   (owes $39.00)
    Child3:  consumed $39.00   (owes $39.00)
    Carol:   consumed $73.00   (owes $73.00)
    Dave:    consumed $73.00   (owes $73.00)
    Child4:  consumed $39.00   (owes $39.00)
    Eve:     consumed $73.00   (owes $73.00)
    Frank:   consumed $73.00   (owes $73.00)
    Grace:   consumed $73.00   (owes $73.00)
    Henry:   consumed $42.00   (owes $42.00)
    Isla:    consumed $73.00   (paid $375.50 → owed $302.50)
```

### 4. Balance view — PFG rollup at scale

```
S11 Balances (by Settlement Group)
    ⚡ GET /events/{eid}/positions/pfgs

    🔵 Family 1      is owed $143.50
       ├ Alice   +$333.50
       ├ Ben     −$73.00
       ├ Child1  −$39.00
       ├ Child2  −$39.00
       └ Child3  −$39.00

    🟢 Family 2      owes $185.00
       ├ Carol   −$73.00
       ├ Dave    −$73.00
       └ Child4  −$39.00

    🟡 Couple 1      owes $146.00
       ├ Eve     −$73.00
       └ Frank   −$73.00

    🟣 Couple 2      owes $115.00
       ├ Grace   −$73.00
       └ Henry   −$42.00

    🔴 Isla          is owed $302.50
```

13 individual positions collapse to 5 PFG positions. The complexity is absorbed by the app.

### 5. Settlement suggestions

```
S12 Settle Up
    ⚡ GET /events/{eid}/settlement-suggestions
       → server computes optimal payment paths at PFG level

    Suggested settlements:
    🟢 Family 2   pays $185.00  →  🔴 Isla
    🟡 Couple 1   pays $117.50  →  🔴 Isla
    🟡 Couple 1   pays $28.50   →  🔵 Family 1
    🟣 Couple 2   pays $115.00  →  🔵 Family 1
```

Four transfers settle a 13-person lunch. Without PFG grouping, up to 11 transfers would be needed (every debtor paying every creditor).

## Arithmetic Verification

### Line items and per-person shares

| Line Item | Amount | Split Among | Per-Person |
|-----------|--------|-------------|------------|
| Kids Meals | $156.00 | 4 kids equally | $39.00 |
| Adult Meals | $378.00 | 9 adults equally | $42.00 |
| Drinks | $217.00 | 7 drinkers equally | $31.00 |
| **Total** | **$751.00** | | |

### Per-person consumed

| Person | Kids | Adults | Drinks | **Total** |
|--------|------|--------|--------|-----------|
| Alice | — | $42.00 | — | **$42.00** |
| Ben | — | $42.00 | $31.00 | **$73.00** |
| Child1 | $39.00 | — | — | **$39.00** |
| Child2 | $39.00 | — | — | **$39.00** |
| Child3 | $39.00 | — | — | **$39.00** |
| Carol | — | $42.00 | $31.00 | **$73.00** |
| Dave | — | $42.00 | $31.00 | **$73.00** |
| Child4 | $39.00 | — | — | **$39.00** |
| Eve | — | $42.00 | $31.00 | **$73.00** |
| Frank | — | $42.00 | $31.00 | **$73.00** |
| Grace | — | $42.00 | $31.00 | **$73.00** |
| Henry | — | $42.00 | — | **$42.00** |
| Isla | — | $42.00 | $31.00 | **$73.00** |
| **Column ✓** | $156.00 | $378.00 | $217.00 | **$751.00** ✓ |

### Net positions

| Person | Paid | Consumed | Net | Direction |
|--------|------|----------|-----|-----------|
| Alice | $375.50 | $42.00 | +$333.50 | owed |
| Ben | $0 | $73.00 | −$73.00 | owes |
| Child1 | $0 | $39.00 | −$39.00 | owes |
| Child2 | $0 | $39.00 | −$39.00 | owes |
| Child3 | $0 | $39.00 | −$39.00 | owes |
| Carol | $0 | $73.00 | −$73.00 | owes |
| Dave | $0 | $73.00 | −$73.00 | owes |
| Child4 | $0 | $39.00 | −$39.00 | owes |
| Eve | $0 | $73.00 | −$73.00 | owes |
| Frank | $0 | $73.00 | −$73.00 | owes |
| Grace | $0 | $73.00 | −$73.00 | owes |
| Henry | $0 | $42.00 | −$42.00 | owes |
| Isla | $375.50 | $73.00 | +$302.50 | owed |
| **Checksum** | **$751.00** | **$751.00** | **$0.00** | **✓** |

### PFG rollup

| Settlement Group | Members | Combined Net |
|-----------------|---------|-------------|
| 🔵 Family 1 | Alice (+$333.50) + Ben (−$73) + Child1–3 (−$117) | +$143.50 (owed) |
| 🟢 Family 2 | Carol (−$73) + Dave (−$73) + Child4 (−$39) | −$185.00 (owes) |
| 🟡 Couple 1 | Eve (−$73) + Frank (−$73) | −$146.00 (owes) |
| 🟣 Couple 2 | Grace (−$73) + Henry (−$42) | −$115.00 (owes) |
| 🔴 Isla | Isla (+$302.50) | +$302.50 (owed) |
| **Checksum** | | **$0.00** ✓ |

### Settlement path verification

- Family 2 pays $185.00 → Isla: Isla remaining = $302.50 − $185.00 = $117.50
- Couple 1 pays $117.50 → Isla: Isla fully settled; Couple 1 remaining = $146.00 − $117.50 = $28.50
- Couple 1 pays $28.50 → Family 1: Family 1 remaining = $143.50 − $28.50 = $115.00
- Couple 2 pays $115.00 → Family 1: Family 1 fully settled ✓
- All positions resolved. Transfers needed: **4** (optimal for 2 creditors + 3 debtors)

## Validates

- Multi-payer expense: `side: "expense"` splits with equal weight on Alice and Isla across all line items
- S09 "Multiple..." payer flow with per-payer amount display
- Line-item subgroup splits: kids-only, adults-only, drinkers-only on same transaction
- 13-person event at scale with 5 PFGs
- PFG rollup collapsing 13 individual positions to 5 group positions
- Settlement minimisation: 4 transfers (optimal) for 5 PFGs
- Arithmetic verified: all position checksums = $0.00, column totals = line item totals

## Key Principle Demonstrated

> **"Two cards, one bill, fair shares."**
>
> Alice and Isla split the payment 50/50, but consumption varies wildly —
> kids ate $39, non-drinking adults ate $42, drinking adults consumed $73.
> Multi-payer tracks who paid; line-item splits track who consumed.
> Fair Go keeps both dimensions separate, so everyone's settlement is exact.
