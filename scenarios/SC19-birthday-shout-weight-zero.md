# SC19 — Birthday Shout: Weight = 0 Exclusion

> **Renumbered:** Originally proposed as "SC07" in feedback files. Renumbered to SC19 to avoid collision with existing SC07 (Charlie Claims Person).

**Persona:** Frank (admin/payer) — organising Dave's birthday dinner. The group shouts Dave's food; Dave insists on paying for his own drinks.
**Screens:** S05 → S09 → S10 → S05
**Rails:** R02 (Expense)

## Design Principle

**Weight = 0 is not the same as exclusion.** Dave is present at the dinner and eats the food — he is a participant. His weight of zero on the Food line item means he does not *pay* for it. His weight of 1 on the Drinks line item means he pays his equal share. The system tracks participation and financial responsibility independently.

## The Situation

Frank, Alice, Bob, Carol, and Dave go out for Dave's birthday. The group agrees: Dave doesn't pay for food — it's his birthday and the rest of the table is shouting him. But Dave insists on paying for his own drinks.

Frank enters a single expense ("Birthday Dinner") with two line items. On the Food line item, he taps Dave's weight and changes it from 1 to 0. The app instantly recalculates. On the Drinks line item, Dave remains at weight 1 — an equal share with everyone else.

| Line Item | Amount | Paid By | Participants | Dave's Weight |
|-----------|--------|---------|--------------|---------------|
| Food (mains + shared plates) | $280.00 | Frank | Frank, Alice, Bob, Carol, Dave | **0** (shouted) |
| Drinks | $95.00 | Frank | Frank, Alice, Bob, Carol, Dave | **1** (equal share) |

## Flow Diagram

```
S05 Event Dashboard (Frank, "Dave's Birthday" event already created with all 5 people)
    │
    ↓
S09 Add Expense — "Birthday Dinner"
    │  Line item 1: "Food" $280 → set Dave weight to 0
    │  Line item 2: "Drinks" $95 → all weight 1 (default)
    │
    ↓
S10 Expense Detail — review split breakdown
    │
    ↓
S05 Event Dashboard — balances updated
```

## Narrative

### Before the Weight Change (BEFORE state)

Frank opens S09 to add the expense. He enters "Birthday Dinner" and adds the two line items. Initially, all five people have weight 1 on both line items. The app shows a preview before Frank has adjusted Dave's weight:

```
S09 Add Expense — initial preview (all weights = 1, before adjustment)

    Line item 1: "Food" $280.00
    ┌─────────────────────────────────────────────────┐
    │  Frank   weight 1  →  $56.00  (280 ÷ 5)        │
    │  Alice   weight 1  →  $56.00                   │
    │  Bob     weight 1  →  $56.00                   │
    │  Carol   weight 1  →  $56.00                   │
    │  Dave    weight 1  →  $56.00  ← to be changed  │
    └─────────────────────────────────────────────────┘

    Line item 2: "Drinks" $95.00
    ┌─────────────────────────────────────────────────┐
    │  Frank   weight 1  →  $19.00  (95 ÷ 5)         │
    │  Alice   weight 1  →  $19.00                   │
    │  Bob     weight 1  →  $19.00                   │
    │  Carol   weight 1  →  $19.00                   │
    │  Dave    weight 1  →  $19.00                   │
    └─────────────────────────────────────────────────┘

    At this point, Dave's total would be $56 + $19 = $75.
```

### The Key Moment: Setting Dave's Food Weight to Zero

Frank taps Dave's weight field on the Food line item and changes it from **1** to **0**. The system recalculates instantly.

```
S09 Add Expense — AFTER setting Dave's food weight to 0

    Frank taps Dave's weight on "Food" → changes 1 → 0
    → "It's Dave's birthday — the group is shouting his food"

    Line item 1: "Food" $280.00 — NOW a 4-way split
    ┌─────────────────────────────────────────────────┐
    │  Frank   weight 1  →  $70.00  (280 ÷ 4)        │
    │  Alice   weight 1  →  $70.00                   │
    │  Bob     weight 1  →  $70.00                   │
    │  Carol   weight 1  →  $70.00                   │
    │  Dave    weight 0  →   $0.00  ← shouted        │
    └─────────────────────────────────────────────────┘

    Line item 2: "Drinks" $95.00 — unchanged, 5-way equal split
    ┌─────────────────────────────────────────────────┐
    │  Frank   weight 1  →  $19.00  (95 ÷ 5)         │
    │  Alice   weight 1  →  $19.00                   │
    │  Bob     weight 1  →  $19.00                   │
    │  Carol   weight 1  →  $19.00                   │
    │  Dave    weight 1  →  $19.00                   │
    └─────────────────────────────────────────────────┘

    Dave's total: $0 + $19 = $19  (was $75 before the weight change)
    Frank and others' food share: $56 → $70 (the $280 now divides 4 ways)

    Total expense: $375.00
```

One tap on Dave's weight changes his food share from $70 to $0 — and the remaining $280 is automatically redistributed to the 4 remaining participants. No competitor handles this without manual recalculation.

### Saving the Expense

Frank taps "Save Expense".

```
    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Birthday Dinner",
        line_items: [
          {description: "Food", amount: "280.00",
           payer_person_id: frank_id,
           splits: [
             {person_id: frank_id, side: "expense",      weight: 1},
             {person_id: frank_id, side: "consumption",  weight: 1},
             {person_id: alice_id, side: "consumption",  weight: 1},
             {person_id: bob_id,   side: "consumption",  weight: 1},
             {person_id: carol_id, side: "consumption",  weight: 1},
             {person_id: dave_id,  side: "consumption",  weight: 0}
           ]},
          {description: "Drinks", amount: "95.00",
           payer_person_id: frank_id,
           splits: [
             {person_id: frank_id, side: "expense",      weight: 1},
             {person_id: frank_id, side: "consumption",  weight: 1},
             {person_id: alice_id, side: "consumption",  weight: 1},
             {person_id: bob_id,   side: "consumption",  weight: 1},
             {person_id: carol_id, side: "consumption",  weight: 1},
             {person_id: dave_id,  side: "consumption",  weight: 1}
           ]}
        ]}
```

Note that Dave appears in both consumption split lists. He participated in both line items. His weight of 0 on Food records that he was present but not financially responsible. This is structurally different from omitting him from the Food split entirely — the system has an explicit record of the shout decision.

### Expense Detail Review

Frank is taken to S10 (Expense Detail) to review the saved expense before returning to the dashboard.

```
S10 Expense Detail — "Birthday Dinner"

    ┌────────────────────────────────────────────────────────┐
    │  Birthday Dinner                         $375.00       │
    │  Paid by: Frank                                        │
    │                                                        │
    │  Food                                   $280.00        │
    │    Frank    $70.00                                     │
    │    Alice    $70.00                                     │
    │    Bob      $70.00                                     │
    │    Carol    $70.00                                     │
    │    Dave      $0.00  (shouted — weight 0)               │
    │                                                        │
    │  Drinks                                  $95.00        │
    │    Frank    $19.00                                     │
    │    Alice    $19.00                                     │
    │    Bob      $19.00                                     │
    │    Carol    $19.00                                     │
    │    Dave     $19.00                                     │
    └────────────────────────────────────────────────────────┘

    → taps back to S05
```

### Event Dashboard — Final Balances

```
S05 Event Dashboard — "Dave's Birthday"

    1 expense ("Birthday Dinner"), $375.00 total

    🔴 Frank    is owed $286.00
    🔴 Alice    owes    $89.00
    🔴 Bob      owes    $89.00
    🔴 Carol    owes    $89.00
    🔴 Dave     owes    $19.00  ← only his drinks, not his food
    Checksum: $0.00 ✓
```

## Arithmetic Verification

| Person | Food ($280, weight 0 excluded → ÷ 4) | Drinks ($95 ÷ 5) | Total Consumed | Paid     | Net Position           |
|--------|--------------------------------------|-------------------|----------------|----------|------------------------|
| Frank  | $70.00                               | $19.00            | $89.00         | $375.00  | **+$286.00** (owed)    |
| Alice  | $70.00                               | $19.00            | $89.00         | $0.00    | **−$89.00** (owes)     |
| Bob    | $70.00                               | $19.00            | $89.00         | $0.00    | **−$89.00** (owes)     |
| Carol  | $70.00                               | $19.00            | $89.00         | $0.00    | **−$89.00** (owes)     |
| Dave   | $0.00                                | $19.00            | $19.00         | $0.00    | **−$19.00** (owes)     |
| **Σ**  | **$280.00**                          | **$95.00**        | **$375.00**    | **$375.00** | **$0.00 ✓**         |

**BEFORE / AFTER comparison for Dave:**

| State | Dave's Food Share | Dave's Drinks Share | Dave's Total |
|-------|-------------------|---------------------|--------------|
| Before (all weight 1) | $56.00 (280 ÷ 5) | $19.00 (95 ÷ 5) | **$75.00** |
| After (food weight 0) | $0.00 (shouted) | $19.00 (95 ÷ 5) | **$19.00** |

**BEFORE / AFTER comparison for Frank, Alice, Bob, Carol:**

| State | Each person's Food Share |
|-------|--------------------------|
| Before (all weight 1) | $56.00 (280 ÷ 5) |
| After (Dave weight 0) | $70.00 (280 ÷ 4) |

Checksum: −$286.00 + $89.00 + $89.00 + $89.00 + $19.00 = **$0.00 ✓**

## Validates

- **Weight = 0 to exclude someone from financial responsibility** — the "shouting" pattern; Dave eats but does not pay for food
- **Per-line-item weight variance for the same person** — Dave has weight 0 on Food and weight 1 on Drinks within the same expense
- **Instant recalculation on weight change** — changing Dave's food weight from 1 to 0 redistributes $280 across 4 participants without manual arithmetic
- **Participation vs. financial responsibility are distinct** — Dave is listed in the Food consumption split with weight 0, not omitted; the record of the shout is preserved
- **Large group (5 people)** — validates the weight system scales beyond the typical 2–3 person scenarios
- **Single payer, multi-line-item, mixed weights** — one Frank payment, two line items, different weight configurations per item
- **Penny-exact arithmetic** — $280 ÷ 4 = $70.00 exactly; $95 ÷ 5 = $19.00 exactly; no rounding required in this scenario
- **Verified checksum** — net positions sum to $0.00

## Differentiator Moment

Frank taps Dave's weight on the Food line item and changes it from 1 to 0. The preview updates instantly: Dave's food share drops from $70 to $0, and Frank, Alice, Bob, and Carol each go from $56 to $70.

In Splitwise, the only options are: split equally (Dave pays for food he was supposed to be shouted), or create a separate expense that manually excludes Dave, which requires Frank to calculate the drinks-only share himself. In Fair Go, one tap handles the entire recalculation. The system tracks that Dave was present and ate — it just records that his financial weight for that line item is zero. That distinction between participation and financial responsibility is unique to Fair Go's two-sided split model.

> **"It's Dave's birthday. The group is shouting his food. One tap — done."**
