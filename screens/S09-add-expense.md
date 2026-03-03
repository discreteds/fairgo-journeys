# S09 — Add Expense

**Purpose:** The most-used screen. Single-screen expense entry with aggressive defaults.
**Visible to:** All event members.
**Rails:** R02 (Expense)
**Scenarios:** SC01, SC04, SC06, SC08, SC11, SC13, SC17, SC18, SC19, SC24, SC25

This is where "aggressive defaults" matters most. The common case — "I paid, everyone splits equally" — is two fields (what + amount) and one tap (Save).

## Wireframe — Simple Mode (Default)

```
┌──────────────────────────────┐
│  ← Add Expense               │
├──────────────────────────────┤
│                              │
│  What for?     [Restaurant_] │
│  Amount        [$620.00____] │
│  Currency      [AUD ▼______] │
│  Date          [Today ▼____] │
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
│  Currency      [AUD ▼______] │
│  Date          [Today ▼____] │
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
│  Currency      [AUD ▼______] │
│  Date          [Today ▼____] │
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

### Weight Patterns

The custom weights picker supports several common patterns demonstrated across scenarios:

**Equal weights (default) — SC17:**
```
│  Split: Equal                  │
│  ┌──────────────────────────┐│
│  │ Alice    [1_]  $15.77    ││
│  │ Bob      [1_]  $15.77    ││
│  │ Carol    [1_]  $15.76    ││
│  └──────────────────────────┘│
│  Total: $47.30 (penny-exact) │
```

**Proportional ratio — SC13 (income split 3:2):**
```
│  Split: Custom weights         │
│  ┌──────────────────────────┐│
│  │ Mel      [3_]  $54.00    ││
│  │ Jake     [2_]  $36.00    ││
│  └──────────────────────────┘│
│  Total: $90.00 (60/40 split) │
```

**Zero-weight exclusion — SC19 (birthday shout), SC01 (non-drinker):**
```
│  Split: Custom weights         │
│  ┌──────────────────────────┐│
│  │ Alice    [1_]  $17.50    ││
│  │ Bob      [1_]  $17.50    ││
│  │ Carol    [0_]  $0.00     ││  ← excluded
│  │ Dave     [1_]  $17.50    ││
│  │ Eve      [1_]  $17.50    ││
│  └──────────────────────────┘│
│  Total: $70.00               │
```

Weight = 0 means the person is excluded from that line item entirely. Their share is $0.00 and they don't appear in the split calculations. This enables patterns like:
- Non-drinker excluded from alcohol line item (SC01)
- Birthday person's food is shouted by others (SC19)
- Vegan excluded from meat-based grocery items (SC04)

**Payer switching — SC18:**

The "Who paid?" picker defaults to the current user but can be changed to any participant. This enables multi-payer events where different people pay for different expenses (e.g. one person pays for accommodation, another for fuel).

## Orchestration — "Save Expense" (Split-Pending, ≤1 Person)

When the event has ≤1 person, the expense is saved without splits:

```
POST /events/{eid}/transactions
  Headers: Idempotency-Key: <client-generated-uuid>
  {
    description: "Pizza",
    splits_status: "pending",
    currency: "AUD",
    occurred_at: "2026-03-03T00:00:00Z",
    line_items: [{
      description: "Pizza",
      amount: 45.00,
      splits: []
    }]
  }
→ transaction created with splits_status: "pending"
→ S05 (event dashboard, expense shows "⏳ splits pending")
```

> **Idempotency-Key** — Required on all `POST /events/{eid}/transactions` requests. The client generates a UUID v4 before submission. If the same key is re-sent (e.g. network retry, double-tap), the server returns the original response without creating a duplicate transaction. This is the financial mutation — idempotency prevents double-charges.

When people are later added via S07, the backend auto-assigns splits to all pending transactions.

## Orchestration — "Save Expense" (Single Line Item, Equal Split)

When the event has ≥2 people, the normal split flow applies:

```
POST /events/{eid}/transactions
  Headers: Idempotency-Key: <client-generated-uuid>
  {
    description: "Restaurant",
    currency: "AUD",
    occurred_at: "2026-03-03T00:00:00Z",
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

Same single `POST /events/{eid}/transactions` (with `Idempotency-Key` header) but with `line_items` array containing 2+ items, each with their own splits:

```
POST /events/{eid}/transactions
  Headers: Idempotency-Key: <client-generated-uuid>
  {
    description: "Restaurant",
    currency: "AUD",
    occurred_at: "2026-03-03T00:00:00Z",
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
| Currency | Event currency | Picker to select a different currency (multi-currency events); records FX rate when overridden |
| Date (occurred_at) | Current date/time ("Today") | Date picker to backdate expense to when it actually occurred |
| Splits status | "assigned" (≥2 people) / "pending" (≤1 person) | Automatic based on person count |

- Most expenses: fill "What for" + "Amount" → tap Save. Two fields, one tap.
- **Split-pending mode** activates automatically when ≤1 person — no redirect, no blocking
- Groups appear as named shortcuts in the "Split between" picker
- Multi-line-item is opt-in, not forced
- "Custom weights" shows live-calculated shares as weights are adjusted
- Modifier field is hidden by default (1.0); backend accepts 0.0-2.0. Values >1.0 represent surcharges (e.g. 1.5 = pays 150% share). Surface in UI for "advanced splitting" scenarios (dietary adjustments, service charges).
- **Multi-currency:** Currency defaults to event currency but can be overridden via the currency picker. When a non-default currency is selected, the system records the FX rate at time of entry for balance calculations.
- **Backdating:** The date field defaults to "Today" but allows selecting a past date, so expenses can be recorded with the date they actually occurred (e.g. adding yesterday's taxi after the fact).

## Error States

| Error | Display |
|-------|---------|
| Transaction limit reached (unfunded) | "This event needs funding to add more expenses" → S14 or "Contact your event admin" |
| Line item limit reached (unfunded) | Same as above |
| Amount = 0 or negative | Inline: "Enter an amount greater than zero" |
| No description | Inline: "What was this expense for?" |
| Expense side doesn't match total | "Amounts paid ($X) don't match the total ($Y)" |
