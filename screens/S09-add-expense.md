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

## Wireframe — Split-Pending Mode (≤1 Person in Event)

When Alice is the only person in the event, split controls are replaced with a pending indicator. The expense can still be saved — splits will be auto-assigned when people are added.

```
┌──────────────────────────────┐
│  ← Add Expense               │
├──────────────────────────────┤
│                              │
│  What for?     [Pizza______] │
│  Amount        [$45.00_____] │
│                              │
│  ┌──────────────────────────┐│
│  │ ℹ️ Splits will be assigned ││
│  │ when you add people.     ││
│  │                          ││
│  │ You can add people after ││
│  │ saving this expense.     ││
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

Weights are integers (1, 2, etc). The modifier field (not shown by default) allows fractional adjustments: 0.5 = half share (child discount), 1.5 = 150% share (surcharge). Modifier range: 0.0-2.0.

## Orchestration — "Save Expense" (Split-Pending, ≤1 Person)

When the event has ≤1 person, the expense is saved without splits:

```
POST /events/{eid}/transactions
  {
    description: "Pizza",
    splits_status: "pending",
    currency: "AUD",
    line_items: [{
      description: "Pizza",
      amount: 45.00,
      splits: []
    }]
  }
→ transaction created with splits_status: "pending"
→ S05 (event dashboard, expense shows "⏳ splits pending")
```

When people are later added via S07, the backend auto-assigns splits to all pending transactions.

## Orchestration — "Save Expense" (Single Line Item, Equal Split)

When the event has ≥2 people, the normal split flow applies:

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

## Orchestration — "Save Expense" (Auto-Assign When People Exist)

If the event already has people but the user creates a split-pending transaction (edge case), the backend auto-assigns splits immediately using the existing people, transitioning `splits_status` to `"assigned"` on save.

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
| Splits status | "assigned" (≥2 people) / "pending" (≤1 person) | Automatic based on person count |

- Most expenses: fill "What for" + "Amount" → tap Save. Two fields, one tap.
- **Split-pending mode** activates automatically when ≤1 person — no redirect, no blocking
- Groups appear as named shortcuts in the "Split between" picker
- Multi-line-item is opt-in, not forced
- "Custom weights" shows live-calculated shares as weights are adjusted
- Modifier field is hidden by default (1.0); backend accepts 0.0-2.0. Values >1.0 represent surcharges (e.g. 1.5 = pays 150% share). Surface in UI for "advanced splitting" scenarios (dietary adjustments, service charges).

## Error States

| Error | Display |
|-------|---------|
| Transaction limit reached (unfunded) | "This event needs funding to add more expenses" → S14 or "Contact your event admin" |
| Line item limit reached (unfunded) | Same as above |
| Amount = 0 or negative | Inline: "Enter an amount greater than zero" |
| No description | Inline: "What was this expense for?" |
| Expense side doesn't match total | "Amounts paid ($X) don't match the total ($Y)" |
