# SC11 — Complex Group Splits (The Restaurant Bill)

**Persona:** Alice hosts a dinner; food split 5 ways, alcohol split 3 ways, dessert split 2 ways.
**Screens:** S05 → S09 → S08 → S10 → S11
**Rails:** R02 (Expense)

## Design Principle

**"Fair ≠ equal."** This scenario demonstrates the core Fair Go thesis: a complex restaurant bill where different items are consumed by different people at different rates. Simple equal splitting would be unfair — line-item-level splits with weights make it fair.

## Narrative

Alice's event has 5 people: Alice, Bob, Carol, Dave, Eve.

Alice creates a transaction with multiple line items:

```
S05 → taps "+ Add Expense"
S09 Add Expense → "Restaurant Dinner"
    → Total will be sum of line items
    → taps "Add Line Items" (multi-line mode)
```

**Line Item 1: Food ($200, split 5 ways equally)**

```
S09 → Line Item: "Food" / $200.00
    → Paid by: Alice
    → Split between: Everyone (5 people)
    → Weights: all equal (1:1:1:1:1)
    → Each person: $40.00
```

**Line Item 2: Alcohol ($150, split 3 ways — only Alice, Bob, Dave drank)**

```
S09 → Line Item: "Wine & Beer" / $150.00
    → Paid by: Alice
    → Split between: Alice, Bob, Dave (custom selection)
    → Weights: all equal (1:1:1)
    → Each drinker: $50.00
    → Carol and Eve excluded — they don't drink
```

**Line Item 3: Dessert ($60, split 2 ways — Alice and Carol shared)**

```
S09 → Line Item: "Dessert Platter" / $60.00
    → Paid by: Alice
    → Split between: Alice, Carol
    → Weights: 2:1 (Alice had more)
    → Alice: $40.00 (weight 2 of 3)
    → Carol: $20.00 (weight 1 of 3)
```

**Line Item 4: Service charge ($41, split 5 ways with modifier)**

```
S09 → Line Item: "Service Charge" / $41.00
    → Paid by: Alice
    → Split between: Everyone
    → Weights: equal
    → Modifier on Eve: 0.5 (she arrived late, agreed to pay half service)
    → Effective: Alice(1×1.0), Bob(1×1.0), Carol(1×1.0), Dave(1×1.0), Eve(1×0.5)
    → Sum of effective weights: 4.5
    → Alice, Bob, Carol, Dave: $41 × 1.0/4.5 = $9.11 each
    → Eve: $41 × 0.5/4.5 = $4.56
```

Alice saves the transaction:

```
S09 → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Restaurant Dinner",
        line_items: [
          {description: "Food", amount: 200.00,
           expense_splits: [{person_id: alice, weight: 1, modifier: 1.0}],
           consumption_splits: [{person_id: alice, weight: 1}, {person_id: bob, weight: 1},
                                {person_id: carol, weight: 1}, {person_id: dave, weight: 1},
                                {person_id: eve, weight: 1}]},
          {description: "Wine & Beer", amount: 150.00,
           expense_splits: [{person_id: alice, weight: 1, modifier: 1.0}],
           consumption_splits: [{person_id: alice, weight: 1}, {person_id: bob, weight: 1},
                                {person_id: dave, weight: 1}]},
          {description: "Dessert Platter", amount: 60.00,
           expense_splits: [{person_id: alice, weight: 1, modifier: 1.0}],
           consumption_splits: [{person_id: alice, weight: 2}, {person_id: carol, weight: 1}]},
          {description: "Service Charge", amount: 41.00,
           expense_splits: [{person_id: alice, weight: 1, modifier: 1.0}],
           consumption_splits: [{person_id: alice, weight: 1, modifier: 1.0},
                                {person_id: bob, weight: 1, modifier: 1.0},
                                {person_id: carol, weight: 1, modifier: 1.0},
                                {person_id: dave, weight: 1, modifier: 1.0},
                                {person_id: eve, weight: 1, modifier: 0.5}]}
        ]}
```

Viewing the expense detail:

```
S10 Expense Detail (P2 gap — per-line-item breakdown)
    Restaurant Dinner — Total: $451.00
    Paid by: Alice

    ┌──────────────────────────────────────────────┐
    │ Food         $200.00   5 people × $40.00     │
    │ Wine & Beer  $150.00   3 people × $50.00     │
    │ Dessert      $60.00    Alice $40 / Carol $20  │
    │ Service      $41.00    4×$9.11 + Eve $4.56   │
    └──────────────────────────────────────────────┘

    Per-person totals:
    Alice:  consumed $139.11  (paid $451.00 → owed $311.89)
    Bob:    consumed $99.11   (owes $99.11)
    Carol:  consumed $69.11   (owes $69.11)
    Dave:   consumed $99.11   (owes $99.11)
    Eve:    consumed $44.56   (owes $44.56)
```

Positions view:

```
S11 Balances
    ⚡ GET /events/{eid}/positions/persons
    → Checksum: $0.00 ✓

    Alice:  paid $451.00, consumed $139.11 → is owed $311.89
    Bob:    paid $0,      consumed $99.11  → owes $99.11
    Carol:  paid $0,      consumed $69.11  → owes $69.11
    Dave:   paid $0,      consumed $99.11  → owes $99.11
    Eve:    paid $0,      consumed $44.56  → owes $44.56
```

## Using Groups to Simplify Entry

Instead of manually selecting people per line item, Alice could use groups:

```
S08 Manage Groups → creates "Drinkers" group
    ⚡ POST /events/{eid}/groups {name: "Drinkers", type: "composite", member_group_ids: [...]}
    → creates composite group in a single call (JF-2B)

S09 → Line Item: "Wine & Beer" / $150.00
    → Split between: [Drinkers] group
    ⚡ POST /events/{eid}/groups/{gid}/expand {amount: 150.00}
    → returns per-person amounts based on group member weights
```

## Validates

- Multi-line-item transactions with different split groups per line
- Non-equal weights (dessert 2:1 split)
- Modifiers (service charge with Eve at 0.5)
- Group expansion (`POST .../expand`) for split convenience
- Per-line-item split customisation in transaction creation
- Expense detail view (S10, P2 gap) showing line-by-line breakdown
- Position accuracy with complex splits — checksum $0.00
- Formula: `person_share = amount × (weight × modifier) / Σ(weight × modifier)`

## Key Principle Demonstrated

> **"Fair ≠ equal"**
>
> Equal splitting would charge Carol $90.20 for alcohol she didn't drink
> and Eve full service charge when she arrived late. Line-item splits with
> weights and modifiers mean everyone pays exactly their fair share.
> This is the core Fair Go thesis.
