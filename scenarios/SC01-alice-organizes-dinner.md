# SC01 — Alice Organizes a Dinner

> **Replaces:** Original SC01 (equal splits). Rework adds line-item splits and differentiator demonstration.

**Persona:** First-time user, becomes event admin. The loyalty maven — she always puts the bill on her card.
**Screens:** S01 → S02 → S03 → S04 → S05 → S07 → S09 → S05 → S06
**Rails:** R01 (Onboarding), R02 (Expense), R04 (Invitation)

## Design Principle

**Capture first, share last.** Alice records what happened (expenses with line items), adds who was there (people), then shares the event when everything is ready. This scenario demonstrates Fair Go's core differentiator: **line-item-level splits with different consumption groups**, showing why equal division is unfair and how Fair Go makes it right in two taps.

## The Situation

Alice went to dinner with Bob and Carol. The bill was $75.00. But it was not an equal night:

- **Everyone** shared the food ($45.00 — pizza and sides, split three ways)
- **Alice and Bob** split the wine ($30.00)
- **Carol drove** — she does not drink

Under an equal split, Carol would pay **$25.00**. Under Fair Go's line-item split, Carol pays **$15.00** — exactly what she consumed. That is a **$10.00 difference**. That is the problem Fair Go solves.

## Flow Diagram

```
S01 → S02 → S03 → S04 → S05 (Event Dashboard)
                                │
                         S07 Manage People
                         (Alice, Bob, Carol)
                                │
                         S05 (Dashboard)
                                │
                    ┌───────────┴───────────┐
                    ↓                       ↓
              S09 Add Expense         S09 Add Expense
              "Food" — Everyone       "Wine" — Alice, Bob only
              ($45 ÷ 3 = $15 each)    ($30 ÷ 2 = $15 each)
                    │                       │
                    └───────────┬───────────┘
                                ↓
                          S05 (Dashboard)
                                ↓
                          S06 Prepare & Share
                          ("Carol owes $15, not $25")
```

## Narrative

### Onboarding

Alice downloads the app and sees the welcome screen.

```
S01 Welcome → taps "Get Started"
S02 Register → creates account (display name: "Alice", email, password)
    ⚡ POST /auth/register → tokens stored
S03 Home → empty state ("No events yet!")
    → taps "+ New Event"
S04 Create Event → types "Italian Dinner"
    ⚡ POST /events → event created, Alice auto-added as admin + person
    ⚡ POST /events/{eid}/invite-codes → default invite code generated (NOT shared yet)
S05 Event Dashboard → Alice sees her event, 1 person, 0 expenses
```

### Adding People

Alice adds everyone who was at dinner before entering expenses. She knows who was there and wants to assign splits as she goes — this way the person picker is available when she enters each line item.

```
S05 → taps "People ▸"
S07 Manage People
    → adds Bob
    ⚡ POST /events/{eid}/persons {display_name: "Bob"}
    → adds Carol
    ⚡ POST /events/{eid}/persons {display_name: "Carol"}

    People list:
    🔴 Alice (you)    — solo settlement group
    🔴 Bob            — solo settlement group
    🔴 Carol          — solo settlement group

    → back to S05
```

### Expense 1: Food (split equally across everyone)

Alice enters the food portion of the bill. All three people ate, so the default "everyone" split is correct.

```
S09 Add Expense
    Description: "Italian Dinner"
    Paid by: You (Alice) ← default, correct — Alice put it on her card

    → taps "+ Add Line Item"

    Line item 1:
    ┌──────────────────────────────────────────────────┐
    │  Description:  Food                              │
    │  Amount:       $45.00                            │
    │  Split:        Everyone ✓ (Alice, Bob, Carol)    │
    │  Preview:      $15.00 each                       │
    └──────────────────────────────────────────────────┘

    ⚡ (line item staged, not yet saved)
```

### Expense 2: Wine (Carol excluded — this is the differentiator moment)

Alice taps "+ Add Line Item" to add the wine to the same transaction. The default is "everyone" — but Carol did not drink.

```
    → taps "+ Add Line Item"

    Line item 2:
    ┌──────────────────────────────────────────────────┐
    │  Description:  Wine                              │
    │  Amount:       $30.00                            │
    │  Split:        Everyone ← default, needs change  │
    │                                                  │
    │  → Alice taps the split selector                 │
    │  → Person picker opens:                          │
    │      ☑ Alice                                     │
    │      ☑ Bob                                       │
    │      ☑ Carol  ← Alice taps to uncheck            │
    │      ☐ Carol  ← Carol is now excluded            │
    │                                                  │
    │  Split:        Alice, Bob only                   │
    │  Preview:      $15.00 each (Alice $15, Bob $15)  │
    │                Carol: $0.00                      │
    └──────────────────────────────────────────────────┘

    Total expense: $75.00

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Italian Dinner",
        line_items: [
          {description: "Food", amount: "45.00",
           payer_person_id: alice_id,
           splits: [
             {person_id: alice_id, side: "expense",      weight: 1},
             {person_id: alice_id, side: "consumption",  weight: 1},
             {person_id: bob_id,   side: "consumption",  weight: 1},
             {person_id: carol_id, side: "consumption",  weight: 1}
           ]},
          {description: "Wine", amount: "30.00",
           payer_person_id: alice_id,
           splits: [
             {person_id: alice_id, side: "expense",      weight: 1},
             {person_id: alice_id, side: "consumption",  weight: 1},
             {person_id: bob_id,   side: "consumption",  weight: 1}
           ]}
        ]}
```

Carol has no consumption split on the wine line item. She is not charged. This outcome is structurally impossible in apps that only record a single total per expense — they can only split $75 three ways ($25 each) or require Alice to manually calculate and enter separate expenses.

### Dashboard and Sharing

Alice returns to the event dashboard. The balances are already calculated.

```
S05 Event Dashboard
    1 expense ("Italian Dinner"), $75.00 total
    Alice is owed $45.00
    Bob owes $30.00
    Carol owes $15.00

    → taps "Share Event →"

S06 Prepare & Share
    → Person checklist:
      Bob   — placeholder [Add phone / Create personal link]
      Carol — placeholder [Add phone / Create personal link]
    → Share options:
      [Copy Group Link]
      [Share via...]
      [Create Personal Links]

    Each recipient sees their specific balance:
      Bob's link   → "You owe Alice $30.00"
      Carol's link → "You owe Alice $15.00"
```

## Arithmetic Verification

| Person | Food ($45 ÷ 3) | Wine ($30 ÷ 2) | Total Consumed | Paid   | Net Position          |
|--------|----------------|----------------|----------------|--------|-----------------------|
| Alice  | $15.00         | $15.00         | $30.00         | $75.00 | **+$45.00** (owed)    |
| Bob    | $15.00         | $15.00         | $30.00         | $0.00  | **−$30.00** (owes)    |
| Carol  | $15.00         | $0.00          | $15.00         | $0.00  | **−$15.00** (owes)    |
| **Σ**  | **$45.00**     | **$30.00**     | **$75.00**     | **$75.00** | **$0.00 ✓**       |

Checksum: +$45.00 − $30.00 − $15.00 = **$0.00 ✓**

Under an equal split Carol would have paid $25.00 — $10.00 more than she consumed. One tap on the wine person picker corrects this permanently.

## Validates

- **Full onboarding flow:** S01 → S02 → S03 → S04 → S05
- **Line-item-level splits with different consumption groups** — the core differentiator
- **Person picker on split assignment** — unchecking Carol from the wine line item
- **Payer defaults to current user** — explicitly shown and correct for the loyalty maven persona
- **Multi-line-item single expense** — "Italian Dinner" has two line items with independent splits
- **Penny-exact arithmetic** — all amounts verified, checksum = $0.00
- **Two-sided split model** — expense side (who paid) and consumption side (who consumed) shown in API calls
- **The excluded person is protected** — Carol pays only for what she consumed
- **Prepare & Share as final step** — personalised balance per recipient
- **Auto-creation of invite codes on event creation** — not shared until S06

## Differentiator Moment

The scenario's key moment is when Alice unchecks Carol from the wine line item. In a simple expense-splitting app the only options are: split $75 equally ($25 each), or enter two separate manual expenses and do the arithmetic yourself. In Fair Go, Alice adds one expense ("Italian Dinner"), marks the two line items and who consumed each, and the system handles the rest.

**With one tap on Carol's wine split, Carol's share drops from $25.00 to $15.00 — and the calculation is instant, exact, and permanent.**

This is the interaction that justifies the product's existence. Every other step in this scenario (registration, event creation, adding people, sharing) is table stakes that any competitor provides. The line-item person picker is why Fair Go is different.

> **"Carol drove. She didn't drink. She's not splitting the wine. Fair go."**
