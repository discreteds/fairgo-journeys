# SC04 — Housemates: Monthly Bills

> **Rework:** Replaces original SC04. Adds verified arithmetic, unequal rent splits, and dietary exclusion (Sam weight=0 on meat expenses).

**Persona:** Alex (admin) — master bedroom housemate running recurring household expenses. Everyone settles independently.
**Screens:** S04 → S05 → S07 → S09 (×4) → S11 → S12
**Rails:** R01 (Onboarding), R02 (Expense), R03 (Settlement)

## Design Principle

**Fair isn't always equal.** Alex has the master bedroom — more space, more rent. Sam is vegetarian — she shouldn't subsidise meat she doesn't eat. Casey is the baseline: standard room, eats everything. Three people, three different situations. Fair Go lets each expense reflect the actual arrangement, not an assumed equal share.

## The Situation

Alex, Sam, and Casey share a house. They track monthly costs in a single ongoing event. This month's expenses land differently:

| Expense | Amount | Paid By | Split Pattern |
|---------|--------|---------|---------------|
| Rent | $1,500.00 | Alex (pays landlord directly) | Unequal: master room premium |
| Electricity | $180.00 | Casey | 3-way equal |
| Meat groceries | $90.00 | Casey | Alex + Casey only (Sam excluded) |
| Veggie/shared groceries | $120.00 | Sam | 3-way equal |

Total tracked: **$1,890.00**

## People and Settlement Groups

```
S04 Create Event → "House Bills Feb"
    Type: Ongoing
    ⚡ POST /events {name: "House Bills Feb", event_type: "ongoing"}

S05 → taps "People ▸"
S07 Manage People → adds Sam, Casey
    ⚡ POST /events/{eid}/persons × 2

    People:
    🔴 Alex  (you, admin) — singleton PFG
    🔴 Sam               — singleton PFG
    🔴 Casey             — singleton PFG

    Three housemates = 3 mouths AND 3 credit cards.
    Each person settles their own debts independently.
    No settlement group setup needed — singleton PFGs are the default.
```

## Entering Expenses

### Expense 1: Rent — Alex pays, unequal split

Alex has the master bedroom. The three housemates have agreed on custom amounts: Alex pays $650 (master room premium), Sam and Casey each pay $425 (standard rooms).

```
S05 → taps "+ Add Expense"
S09 Add Expense
    Description: "Rent"
    Amount: $1,500.00
    Paid by: You (Alex)

    Line item: "Rent" $1,500.00
    → Split: Alex, Sam, Casey (all selected by default)
    → Alex taps the split row → switches to "Custom amounts"
    → Custom amount entry:
        Alex:  $650.00
        Sam:   $425.00
        Casey: $425.00

    Split preview:
    ┌────────────────────────────────────────────────┐
    │  Alex:   $650.00  (43.3%)                       │
    │  Sam:    $425.00  (28.3%)                       │
    │  Casey:  $425.00  (28.3%)                       │
    │  Total: $1,500.00  ✓                            │
    └────────────────────────────────────────────────┘

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Rent",
        line_items: [{
          description: "Rent", amount: "1500.00",
          payer_person_id: alex_id,
          splits: [
            {person_id: alex_id,  side: "expense",      weight: 1},
            {person_id: alex_id,  side: "consumption",  weight: 650},
            {person_id: sam_id,   side: "consumption",  weight: 425},
            {person_id: casey_id, side: "consumption",  weight: 425}
          ]
        }]}

S05 → "Rent $1,500.00 — Alex paid"
```

### Expense 2: Electricity — Casey pays, 3-way equal

```
S05 → taps "+ Add Expense"
S09 Add Expense
    Description: "Electricity"
    Amount: $180.00
    Paid by: Casey  ← Alex changes payer from the default (herself)

    Line item: "Electricity" $180.00
    → Split: Everyone (3-way equal, default)

    Split preview:
    ┌────────────────────────────────────────────────┐
    │  Alex:   $60.00  (33.3%)                        │
    │  Sam:    $60.00  (33.3%)                        │
    │  Casey:  $60.00  (33.3%)                        │
    │  Total: $180.00  ✓                              │
    └────────────────────────────────────────────────┘

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Electricity",
        line_items: [{
          description: "Electricity", amount: "180.00",
          payer_person_id: casey_id,
          splits: [
            {person_id: casey_id, side: "expense",      weight: 1},
            {person_id: alex_id,  side: "consumption",  weight: 1},
            {person_id: sam_id,   side: "consumption",  weight: 1},
            {person_id: casey_id, side: "consumption",  weight: 1}
          ]
        }]}

S05 → "Electricity $180.00 — Casey paid"
```

### Expense 3: Meat groceries — Casey pays, Sam excluded

Sam is vegetarian. She shouldn't pay for the meat Casey bought. Alex sets Sam's weight to 0 on this line item — Sam is excluded entirely. Alex and Casey split the cost equally.

```
S05 → taps "+ Add Expense"
S09 Add Expense
    Description: "Groceries – meat"
    Amount: $90.00
    Paid by: Casey

    Line item: "Groceries – meat" $90.00
    → Split: All three selected by default
    → Alex taps the split row → switches to "Custom weights"
    → Weight entry:
        Alex:  1
        Sam:   0   ← Sam excluded (dietary — she didn't consume this)
        Casey: 1

    Split preview:
    ┌────────────────────────────────────────────────┐
    │  Alex:   $45.00  (1/2 = 50%)                    │
    │  Sam:     $0.00  (0/2 = 0%)                     │
    │  Casey:  $45.00  (1/2 = 50%)                    │
    │  Total:  $90.00  ✓                              │
    └────────────────────────────────────────────────┘

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Groceries – meat",
        line_items: [{
          description: "Groceries – meat", amount: "90.00",
          payer_person_id: casey_id,
          splits: [
            {person_id: casey_id, side: "expense",      weight: 1},
            {person_id: alex_id,  side: "consumption",  weight: 1},
            {person_id: sam_id,   side: "consumption",  weight: 0},
            {person_id: casey_id, side: "consumption",  weight: 1}
          ]
        }]}

S05 → "Groceries – meat $90.00 — Casey paid"
```

### Expense 4: Veggie/shared groceries — Sam pays, 3-way equal

Sam bought the shared vegetables and pantry items everyone uses. All three split equally.

```
S05 → taps "+ Add Expense"
S09 Add Expense
    Description: "Groceries – veg & pantry"
    Amount: $120.00
    Paid by: Sam

    Line item: "Groceries – veg & pantry" $120.00
    → Split: Everyone (3-way equal, default)

    Split preview:
    ┌────────────────────────────────────────────────┐
    │  Alex:   $40.00  (33.3%)                        │
    │  Sam:    $40.00  (33.3%)                        │
    │  Casey:  $40.00  (33.3%)                        │
    │  Total: $120.00  ✓                              │
    └────────────────────────────────────────────────┘

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Groceries – veg & pantry",
        line_items: [{
          description: "Groceries – veg & pantry", amount: "120.00",
          payer_person_id: sam_id,
          splits: [
            {person_id: sam_id,   side: "expense",      weight: 1},
            {person_id: alex_id,  side: "consumption",  weight: 1},
            {person_id: sam_id,   side: "consumption",  weight: 1},
            {person_id: casey_id, side: "consumption",  weight: 1}
          ]
        }]}

S05 → "Groceries – veg & pantry $120.00 — Sam paid"
```

## Balances

```
S11 Balances
    ⚡ GET /events/{eid}/positions/persons

    ┌─────────────────────────────────────────────────────────────────────┐
    │  Expense               Amount   Alex       Sam       Casey          │
    │  Rent                $1,500    $650.00   $425.00   $425.00         │
    │  Electricity           $180     $60.00    $60.00    $60.00         │
    │  Groceries – meat       $90     $45.00     $0.00    $45.00         │
    │  Groceries – veg       $120     $40.00    $40.00    $40.00         │
    │  ─────────────────────────────────────────────────────────────────  │
    │  Total consumed                $795.00   $525.00   $570.00         │
    │  Total paid                  $1,500.00   $120.00   $270.00         │
    └─────────────────────────────────────────────────────────────────────┘

    Net positions:
    ┌────────────────────────────────────────────────┐
    │  🔴 Alex    is owed $705.00                     │
    │     paid $1,500.00, consumed $795.00            │
    │                                                 │
    │  🔴 Sam     owes $405.00                        │
    │     paid $120.00, consumed $525.00              │
    │                                                 │
    │  🔴 Casey   owes $300.00                        │
    │     paid $270.00, consumed $570.00              │
    │                                                 │
    │  Checksum: $0.00 ✓                              │
    └────────────────────────────────────────────────┘
```

## Arithmetic Verification

| Expense | Amount | Alex consumed | Sam consumed | Casey consumed | Paid by |
|---------|--------|---------------|--------------|----------------|---------|
| Rent | $1,500.00 | $650.00 | $425.00 | $425.00 | Alex $1,500 |
| Electricity | $180.00 | $60.00 | $60.00 | $60.00 | Casey $180 |
| Groceries – meat | $90.00 | $45.00 | $0.00 | $45.00 | Casey $90 |
| Groceries – veg | $120.00 | $40.00 | $40.00 | $40.00 | Sam $120 |
| **Totals** | **$1,890.00** | **$795.00** | **$525.00** | **$570.00** | |

Consumed totals sum: $795 + $525 + $570 = **$1,890.00 ✓**

| Person | Consumed | Paid | Net |
|--------|----------|------|-----|
| Alex | $795.00 | $1,500.00 | **−$705.00** (is owed) |
| Sam | $525.00 | $120.00 | **+$405.00** (owes) |
| Casey | $570.00 | $270.00 | **+$300.00** (owes) |
| **Σ** | **$1,890.00** | **$1,890.00** | **$0.00 ✓** |

## Settlement

```
S12 Settle Up
    ⚡ GET /events/{eid}/settlements/suggested

    Suggested (2 payments — minimum to clear all positions):
    ┌────────────────────────────────────────────────┐
    │  🔴 Sam   pays $405.00 → 🔴 Alex               │
    │  🔴 Casey pays $300.00 → 🔴 Alex               │
    └────────────────────────────────────────────────┘

    → Sam confirms → taps "Mark as Paid"
    ⚡ POST /events/{eid}/settlements
       {from_pfg_id: sam_pfg, to_pfg_id: alex_pfg,
        amount: "405.00", status: "pending"}
    ⚡ PATCH /events/{eid}/settlements/{sid} {status: "paid"}

    → Casey confirms → taps "Mark as Paid"
    ⚡ POST /events/{eid}/settlements
       {from_pfg_id: casey_pfg, to_pfg_id: alex_pfg,
        amount: "300.00", status: "pending"}
    ⚡ PATCH /events/{eid}/settlements/{sid} {status: "paid"}

    S11 Balances → all positions return to $0.00
```

## The Differentiator Moment

Sam never touches the meat expense. She didn't eat it; she doesn't pay for it. In a shared spreadsheet, someone would have to remember to manually exclude Sam from that row — or get into an argument about it. In Fair Go, weight=0 is a structural fact recorded once per expense. No argument needed. The numbers are simply correct.

Conversely, Sam pays for vegetables that all three consume equally. Her vegetarianism doesn't exempt her from shared costs — it exempts her from costs that don't apply to her.

> **"Fair isn't always equal."**
> Equal splits are the default. But sometimes the right answer is $0.00 — and Fair Go can express that without a workaround.

## Split Patterns Used in This Scenario

| Expense | Pattern | Mechanism |
|---------|---------|-----------|
| Rent | Unequal — custom amounts | Custom amount entry ($650 / $425 / $425) |
| Electricity | Equal — 3-way | Default equal split |
| Meat groceries | Partial — 2 of 3 | Weight=0 exclusion for Sam |
| Veggie groceries | Equal — 3-way | Default equal split |

Two distinct split patterns across four expenses, all tracked in one event.

## Key Principle Demonstrated

> **"How many mouths, how many credit cards?"**
>
> Three housemates = 3 mouths AND 3 credit cards.
> Each person settles their own debts individually.
> → Singleton PFGs (the default). No configuration needed.
>
> **Anti-pattern:** If someone assigned all 3 to a shared settlement group,
> all individual debts would be erased — Sam and Casey wouldn't owe anything.
> The UI prevents this by making singleton PFGs the default.

## Ongoing Event Behaviour

Because this event is `event_type: ongoing`, it does not close after settlement. Next month, Alex (or any housemate with access) creates new expenses in the same event. Balances carry forward from any unsettled amounts. The event remains open indefinitely, with periodic settlement cycles.

## Validates

- **Default singleton PFGs** — no settlement group setup needed for housemates
- **Custom amount splits** — rent entered as specific dollar amounts, not percentage ratios
- **Weight=0 dietary exclusion** — Sam excluded from meat groceries; her position is exactly $0.00 on that line item
- **Two distinct split patterns** — unequal (rent) and exclusion (meat) within the same event
- **Multiple payers** — Alex, Sam, and Casey each pay at least one expense
- **Verified arithmetic** — all figures checked; checksum $0.00
- **Settlement minimisation** — 2 payments clear 3 positions (all flow to Alex)
- **Anti-pattern avoided** — singleton PFGs prevent accidental debt erasure
