# S09 — Add Expense

**Purpose:** The most-used screen. Single-screen expense entry with aggressive defaults.
**Visible to:** All event members.
**Rails:** R02 (Expense)
**Scenarios:** SC01, SC02, SC04, SC06

This is where "aggressive defaults" matters most. The common case — "I paid, everyone splits equally" — is two fields (what + amount) and one tap (Save).

## Wireframe — Simple Mode (Default)

```
┌──────────────────────────────┐
│  ← Add Expense               │
├──────────────────────────────┤
│                              │
│  What for?     [Restaurant_] │
│  Amount        [$620.00____] │
│                              │
│  Who paid?                   │
│  ┌──────────────────────────┐│
│  │ ● You (Alice)            ││
│  │ ○ Bob                    ││
│  │ ○ Carol                  ││
│  │ ○ Multiple...            ││
│  └──────────────────────────┘│
│                              │
│  Split between               │
│  ┌──────────────────────────┐│
│  │ ● Everyone (6)           ││
│  │ ○ Drinkers (4)           ││
│  │ ○ Custom...              ││
│  └──────────────────────────┘│
│                              │
│  Split method                │
│  ┌──────────────────────────┐│
│  │ ● Equal                  ││
│  │ ○ Custom weights         ││
│  └──────────────────────────┘│
│                              │
│  ┌──────────────────────────┐│
│  │       Save Expense       ││
│  └──────────────────────────┘│
│                              │
│  ⊕ Add another line item    │
└──────────────────────────────┘
```

## Wireframe — Multi-Line-Item Mode

Tapping "Add another line item" expands the form:

```
┌──────────────────────────────┐
│  ← Add Expense               │
├──────────────────────────────┤
│                              │
│  Description   [Restaurant_] │
│                              │
│  Who paid?                   │
│  ● You (Alice)               │
│                              │
│  ── Line Item 1 ──          │
│  What?    [Food___________]  │
│  Amount   [$480.00________]  │
│  Split:   Everyone · Equal   │
│  [Change split]              │
│                              │
│  ── Line Item 2 ──          │
│  What?    [Alcohol________]  │
│  Amount   [$140.00________]  │
│  Split:   Drinkers · Equal   │
│  [Change split]              │
│                              │
│  ⊕ Add another line item    │
│                              │
│  Total: $620.00              │
│                              │
│  ┌──────────────────────────┐│
│  │       Save Expense       ││
│  └──────────────────────────┘│
└──────────────────────────────┘
```

## Wireframe — Custom Weights

When "Custom weights" is selected for a line item:

```
│  Split: Custom weights       │
│  ┌──────────────────────────┐│
│  │ Alice    [1_]  $103.33   ││
│  │ Bob      [1_]  $103.33   ││
│  │ Carol    [1_]  $103.33   ││
│  │ Dave     [2_]  $206.67   ││
│  │ Eve      [0.5] $ 51.67   ││
│  │ Frank    [0.5] $ 51.67   ││
│  └──────────────────────────┘│
│  Total: $620.00              │
```

Live-calculated shares update as weights change.

## Orchestration — "Save Expense" (Single Line Item, Equal Split)

```
POST /events/{eid}/transactions
  {
    description: "Restaurant",
    currency: "AUD",
    line_items: [{
      description: "Restaurant",
      amount: 620.00,
      splits: [
        {person_id: alice_id, side: "expense",     weight: 1, modifier: 1},
        {person_id: alice_id, side: "consumption",  weight: 1, modifier: 1},
        {person_id: bob_id,   side: "consumption",  weight: 1, modifier: 1},
        {person_id: carol_id, side: "consumption",  weight: 1, modifier: 1},
        {person_id: dave_id,  side: "consumption",  weight: 1, modifier: 1},
        {person_id: eve_id,   side: "consumption",  weight: 1, modifier: 1},
        {person_id: frank_id, side: "consumption",  weight: 1, modifier: 1}
      ]
    }]
  }
→ transaction + line item + all splits created in one API call
→ S05 (event dashboard, updated balances)
```

## Orchestration — "Save Expense" (Multi-Line-Item)

Same single `POST /events/{eid}/transactions` but with `line_items` array containing 2+ items, each with their own splits:

```
POST /events/{eid}/transactions
  {
    description: "Restaurant",
    currency: "AUD",
    line_items: [
      {
        description: "Food",
        amount: 480.00,
        splits: [
          {person_id: alice_id, side: "expense", weight: 1, modifier: 1},
          // consumption splits for all 6 persons...
        ]
      },
      {
        description: "Alcohol",
        amount: 140.00,
        splits: [
          {person_id: alice_id, side: "expense", weight: 1, modifier: 1},
          // consumption splits for 4 drinkers only...
        ]
      }
    ]
  }
```

## Orchestration — "Multiple..." Payer

When multiple people paid:

```
│  Who paid?                   │
│  ┌──────────────────────────┐│
│  │ Alice    [$400.00______] ││
│  │ Bob      [$220.00______] ││
│  └──────────────────────────┘│
```

This generates multiple `side: "expense"` splits with weights proportional to amounts paid.

## Smart Defaults

| Field | Default | Override |
|-------|---------|----------|
| Who paid? | Current user ("You") | Pick any person or "Multiple" |
| Split between | Everyone in event | Pick a group or "Custom" |
| Split method | Equal (weight: 1 each) | Custom weights per person |
| Line item description | Same as transaction description (single item mode) | Separate descriptions |
| Currency | Event currency | Not overridable per-transaction |

- Most expenses: fill "What for" + "Amount" → tap Save. Two fields, one tap.
- Groups appear as named shortcuts in the "Split between" picker
- Multi-line-item is opt-in, not forced
- "Custom weights" shows live-calculated shares as weights are adjusted
- Modifier field is hidden (always 1.0) — available as future feature for child/dietary adjustments

## Empty State

If the event has no other people yet:

```
┌──────────────────────────────┐
│  ← Add Expense               │
├──────────────────────────────┤
│                              │
│  ⚠️ Add people first         │
│  You're the only person in   │
│  this event. Add others to   │
│  split expenses with.        │
│                              │
│  [Add People →]              │
└──────────────────────────────┘
```

## Error States

| Error | Display |
|-------|---------|
| Transaction limit reached (unfunded) | "This event needs funding to add more expenses" → S14 or "Contact your event admin" |
| Line item limit reached (unfunded) | Same as above |
| Amount = 0 or negative | Inline: "Enter an amount greater than zero" |
| No description | Inline: "What was this expense for?" |
| Expense side doesn't match total | "Amounts paid ($X) don't match the total ($Y)" |
