# SC18 — Weekend Away: Multiple Payers & Net Settlement

> **Renumbered:** Originally proposed as "SC06" in feedback files. Renumbered to SC18 to avoid collision with existing SC06 (Free User Hits Limits).

**Persona:** Alice (admin) — a weekend trip where each person pays for different things.
**Screens:** S05 → S09 (×4) → S11 → S12
**Rails:** R01, R02, R03

## Design Principle

**"You don't have to figure out who owes whom for each thing separately."** This scenario demonstrates Fair Go's net settlement: four expenses paid by three different people, one with a partial exclusion (Bob didn't attend the surf lesson), all collapsed into two payments at the end. Complexity of inputs; simplicity of output.

## The Situation

Alice, Bob, and Carol spend a weekend at the coast. Over two days, each person pays for something. Not every expense involves everyone — Bob skips the surf lesson entirely, so he shouldn't pay for it. At checkout, instead of doing mental maths across four transactions with three payers, Fair Go nets all positions and presents the minimum number of payments needed.

| Expense | Amount | Paid By | Split Between |
|---------|--------|---------|---------------|
| Airbnb | $360.00 | Alice | Everyone (3-way equal) |
| Groceries | $84.00 | Bob | Everyone (3-way equal) |
| Surf lesson | $120.00 | Carol | Alice & Carol only (Bob excluded) |
| Dinner out | $93.00 | Alice | Everyone (3-way equal) |

Total spend: $657.00 across 4 expenses and 3 payers.

## Narrative

### Setup

```
S05 Event Dashboard — Alice opens "Weekend at the Coast" (already created)
    → 3 participants: Alice (admin), Bob, Carol
    → 0 expenses so far
    → taps "+ Add Expense" to begin entering costs
```

### Expense 1 — Airbnb (Alice paid, everyone shares)

```
S09 Add Expense
    → Description: "Airbnb"
    → Amount: $360.00
    → Paid by: Alice  ← default (current user)
    → Split: Everyone equal

    Split preview:
    ┌─────────────────────────────────────────────────┐
    │  Alice   $120.00  (weight 1 of 3)               │
    │  Bob     $120.00  (weight 1 of 3)               │
    │  Carol   $120.00  (weight 1 of 3)               │
    │  Total   $360.00  ✓                             │
    └─────────────────────────────────────────────────┘

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Airbnb",
        line_items: [{
          description: "Airbnb",
          amount: "360.00",
          payer_person_id: alice_id,
          splits: [
            {person_id: alice_id, side: "consumption", weight: 1},
            {person_id: bob_id,   side: "consumption", weight: 1},
            {person_id: carol_id, side: "consumption", weight: 1}
          ]
        }]}
```

### Expense 2 — Groceries (Bob paid, everyone shares)

```
S09 Add Expense
    → Description: "Groceries"
    → Amount: $84.00
    → Paid by: Bob  ← Alice changes payer from default (herself) to Bob
        Alice taps the "Paid by" field → selects "Bob" from participant list
    → Split: Everyone equal

    Split preview:
    ┌─────────────────────────────────────────────────┐
    │  Alice   $28.00  (weight 1 of 3)                │
    │  Bob     $28.00  (weight 1 of 3)                │
    │  Carol   $28.00  (weight 1 of 3)                │
    │  Total   $84.00  ✓                              │
    └─────────────────────────────────────────────────┘

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Groceries",
        line_items: [{
          description: "Groceries",
          amount: "84.00",
          payer_person_id: bob_id,
          splits: [
            {person_id: alice_id, side: "consumption", weight: 1},
            {person_id: bob_id,   side: "consumption", weight: 1},
            {person_id: carol_id, side: "consumption", weight: 1}
          ]
        }]}
```

### Expense 3 — Surf Lesson (Carol paid, Bob excluded)

```
S09 Add Expense
    → Description: "Surf Lesson"
    → Amount: $120.00
    → Paid by: Carol  ← Alice changes payer to Carol
    → Split: starts as "Everyone"
        Alice taps Bob's row → sets weight to 0
        Reason: "Bob didn't come to the surf lesson"

    Split preview:
    ┌─────────────────────────────────────────────────┐
    │  Alice   $60.00  (weight 1 of 2 active)         │
    │  Bob     $0.00   (weight 0 — didn't attend)     │
    │  Carol   $60.00  (weight 1 of 2 active)         │
    │  Total   $120.00 ✓                              │
    └─────────────────────────────────────────────────┘

    Bob is in the consumption list but with weight: 0.
    He is not excluded from the event — he simply has no
    financial obligation on this particular line item.

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Surf Lesson",
        line_items: [{
          description: "Surf Lesson",
          amount: "120.00",
          payer_person_id: carol_id,
          splits: [
            {person_id: alice_id, side: "consumption", weight: 1},
            {person_id: bob_id,   side: "consumption", weight: 0},
            {person_id: carol_id, side: "consumption", weight: 1}
          ]
        }]}
```

### Expense 4 — Dinner (Alice paid, everyone shares)

```
S09 Add Expense
    → Description: "Dinner"
    → Amount: $93.00
    → Paid by: Alice  ← default (current user)
    → Split: Everyone equal

    Split preview:
    ┌─────────────────────────────────────────────────┐
    │  Alice   $31.00  (weight 1 of 3)                │
    │  Bob     $31.00  (weight 1 of 3)                │
    │  Carol   $31.00  (weight 1 of 3)                │
    │  Total   $93.00  ✓                              │
    └─────────────────────────────────────────────────┘

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Dinner",
        line_items: [{
          description: "Dinner",
          amount: "93.00",
          payer_person_id: alice_id,
          splits: [
            {person_id: alice_id, side: "consumption", weight: 1},
            {person_id: bob_id,   side: "consumption", weight: 1},
            {person_id: carol_id, side: "consumption", weight: 1}
          ]
        }]}
```

## Arithmetic Verification

**Per-person consumption:**

| Person | Airbnb | Groceries | Surf Lesson | Dinner | Total Consumed |
|--------|--------|-----------|-------------|--------|----------------|
| Alice | $120.00 | $28.00 | $60.00 | $31.00 | **$239.00** |
| Bob | $120.00 | $28.00 | $0.00 | $31.00 | **$179.00** |
| Carol | $120.00 | $28.00 | $60.00 | $31.00 | **$239.00** |
| **Σ** | $360.00 | $84.00 | $120.00 | $93.00 | **$657.00** |

**Per-person payments:**

| Person | Paid |
|--------|------|
| Alice | $360.00 + $93.00 = **$453.00** |
| Bob | **$84.00** |
| Carol | **$120.00** |
| **Σ** | **$657.00** |

**Net positions:**

| Person | Consumed | Paid | Net | Status |
|--------|----------|------|-----|--------|
| Alice | $239.00 | $453.00 | **−$214.00** | is owed $214 |
| Bob | $179.00 | $84.00 | **+$95.00** | owes $95 |
| Carol | $239.00 | $120.00 | **+$119.00** | owes $119 |
| **Σ** | **$657.00** | **$657.00** | **$0.00 ✓** | |

## Balances Screen

```
S11 Balances
    ⚡ GET /events/{eid}/positions/persons

    ┌─────────────────────────────────────────────────┐
    │  🟢 Alice    is owed $214.00                    │
    │  🔴 Bob      owes $95.00                        │
    │  🔴 Carol    owes $119.00                       │
    └─────────────────────────────────────────────────┘

    Checksum: −$214.00 + $95.00 + $119.00 = $0.00 ✓

    → Alice taps "Settle Up"
```

## Settlement Screen

```
S12 Settle Up
    ⚡ GET /events/{eid}/settlements/suggested

    Suggested payments (minimised):
    ┌─────────────────────────────────────────────────┐
    │  Bob   → Alice   $95.00                         │
    │  Carol → Alice   $119.00                        │
    │                                                 │
    │  2 payments  (not 4 reimbursements)             │
    └─────────────────────────────────────────────────┘

    Without netting, the tally would be:
      Bob owes Alice for Airbnb ($120) and Dinner ($31)
      Bob is owed by Alice for Groceries ($28 of $84)
      Carol owes Alice for Airbnb ($120) and Dinner ($31)
      Alice and Carol cross-owe on the surf lesson (Alice
        consumed $60, Carol paid $120 and consumed $60)

    The system collapses all cross-owing into 2 direct transfers:
    Bob pays Alice $95 and Carol pays Alice $119. Done.
```

## The Differentiator Moment

The settlement screen. Alice doesn't need to reason through "Bob owes me for the Airbnb minus what I owe him for the groceries, but Carol owes me for the Airbnb and the dinner, and then there's the surf lesson she paid for..." — the system nets everything. Bob pays Alice $95. Carol pays Alice $119. Two payments instead of four, with no mental arithmetic required.

The surf lesson exclusion makes this concrete: Bob didn't go, so Bob doesn't pay — and that's handled with a single weight change, not a separate event or a manual side payment. Fair Go tracks who consumed what, who paid what, and presents only the minimum transfers needed to settle the whole weekend.

## Validates

- **Multiple payers across an event** — Alice, Bob, and Carol each pay for something
- **Payer switching on expense entry** — admin changes "Paid by" away from the default (self) to another participant
- **Consumption-based exclusion (weight = 0)** — Bob is in the event but has weight 0 on the surf lesson; he is not excluded from the event, only from that line item's cost
- **Mixed split groups** — some expenses are 3-way, one is 2-way
- **Net settlement calculation** — four expenses across three payers collapse to two payments
- **Settlement minimisation** — the system finds the optimal (minimum transfers) payment set
- **Multi-expense event** — expenses entered at different times as they occur
- **Arithmetic conservation** — total consumed = total paid = $657.00; net positions sum to $0.00
