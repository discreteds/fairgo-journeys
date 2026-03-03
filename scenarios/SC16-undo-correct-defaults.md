# SC16 — Undo / Correct Aggressive Defaults

> **Differentiator:** Aggressive defaults are only valuable if they're easy to correct. This scenario proves that corrections flow through the same simple UI — no workarounds, no awkward recalculations.

**Persona:** Alice (admin), correcting a default split after discovering it was wrong.
**Screens:** S05 → S09 → S05 → S10 → S11
**Rails:** R02 (Expense)

## Design Principle

**"Default boldly, correct easily."**

Fair Go defaults the payer to "you" and the split to "everyone equally." Most of the time this is right. When it isn't, the correction is a tap — not a delete-and-recreate.

## Participants

| Person | Role | Settlement Group |
|--------|------|-----------------|
| Alice | Admin, payer | Alice 🔵 |
| Bob | Member | Bob 🟢 |
| Carol | Member | Carol 🟡 |
| Dave | Member | Dave 🔴 |
| Eve | Member | Eve 🟣 |

Five people, all solo PFGs.

## Narrative

### 1. Create expense with all defaults

Alice adds the team dinner. Everything defaults: she's the payer, everyone splits equally.

```
S09 Add Expense
    Description: "Team Dinner"
    Amount: $500.00
    Paid by: You (Alice) ← default
    Split: Everyone equally ← default (Alice, Bob, Carol, Dave, Eve)
    Preview: $100.00 each

    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       Headers: Idempotency-Key: <uuid> (see A07)
       {description: "Team Dinner",
        line_items: [{
          description: "Team Dinner", amount: "500.00",
          payer_person_id: alice_id,
          splits: [
            {person_id: alice_id, side: "expense", weight: 1},
            {person_id: alice_id, side: "consumption", weight: 1},
            {person_id: bob_id,   side: "consumption", weight: 1},
            {person_id: carol_id, side: "consumption", weight: 1},
            {person_id: dave_id,  side: "consumption", weight: 1},
            {person_id: eve_id,   side: "consumption", weight: 1}
          ]
        }]}
```

### 2. Positions after initial save

```
S05 Event Dashboard → S11 Balances

    Person    Paid      Consumed    Net
    ──────    ────      ────────    ────
    Alice     $500.00   $100.00     +$400.00
    Bob       $0.00     $100.00     −$100.00
    Carol     $0.00     $100.00     −$100.00
    Dave      $0.00     $100.00     −$100.00
    Eve       $0.00     $100.00     −$100.00
    ──────    ────      ────────    ────
    Total     $500.00   $500.00     $0.00 ✓
```

### 3. Correction: Bob didn't attend

Alice realises Bob wasn't actually at the dinner. She opens the expense and edits the split.

```
S05 → taps "Team Dinner" → S10 Expense Detail
    → taps [Edit]
    → returns to S09 (edit mode, pre-filled)

S09 Add Expense (edit mode)
    Description: "Team Dinner"
    Amount: $500.00
    Paid by: Alice ← unchanged
    Split: Alice, Bob, Carol, Dave, Eve ← current

    → Alice taps the split selector
    → Person picker opens:
        ☑ Alice
        ☑ Bob    ← Alice taps to uncheck
        ☐ Bob    ← Bob is now excluded
        ☑ Carol
        ☑ Dave
        ☑ Eve

    Preview: $125.00 each (4 people)

    → taps "Save"
    ⚡ PATCH /events/{eid}/transactions/{tid}/line-items/{lid}/splits
       Headers: Idempotency-Key: <uuid> (see A07)
       {splits: [
         {person_id: alice_id, side: "expense", weight: 1},
         {person_id: alice_id, side: "consumption", weight: 1},
         {person_id: carol_id, side: "consumption", weight: 1},
         {person_id: dave_id,  side: "consumption", weight: 1},
         {person_id: eve_id,   side: "consumption", weight: 1}
       ]}
    → positions recalculate
```

### 4. Positions after correction

```
S11 Balances (updated)

    Person    Paid      Consumed    Net
    ──────    ────      ────────    ────
    Alice     $500.00   $125.00     +$375.00
    Bob       $0.00     $0.00       $0.00
    Carol     $0.00     $125.00     −$125.00
    Dave      $0.00     $125.00     −$125.00
    Eve       $0.00     $125.00     −$125.00
    ──────    ────      ────────    ────
    Total     $500.00   $500.00     $0.00 ✓
```

Bob's share is $0. The remaining $500 is split 4 ways at $125 each. Positions sum to zero.

### 5. Member correction path — Carol raises a modification request

Carol checks the expense and believes the split should be unequal — she had a starter only ($80 value) while Dave and Eve had mains ($140 each). Alice herself had $140. Carol requests a weighted correction.

```
S10 Expense Detail → Carol taps [Suggest Change]

    ⚡ POST /events/{eid}/modification-requests
       Headers: Idempotency-Key: <uuid> (see A07)
       {type: "edit_expense",
        target_id: tid,
        proposed_changes: {
          line_items: [{
            id: lid,
            splits: [
              {person_id: alice_id, side: "expense", weight: 1},
              {person_id: alice_id, side: "consumption", weight: 140},
              {person_id: carol_id, side: "consumption", weight: 80},
              {person_id: dave_id,  side: "consumption", weight: 140},
              {person_id: eve_id,   side: "consumption", weight: 140}
            ]
          }]
        },
        message: "Carol had starter only ($80), others had mains ($140)"}
```

Alice sees the request in her admin queue:

```
S16 Admin Moderation
    ┌──────────────────────────────────────────────────┐
    │  📋 Modification Request from Carol              │
    │  "Carol had starter only ($80), others had       │
    │   mains ($140)"                                  │
    │                                                  │
    │  Proposed change:                                │
    │  Alice: $140, Carol: $80, Dave: $140, Eve: $140  │
    │                                                  │
    │  [Approve]  [Reject]                             │
    └──────────────────────────────────────────────────┘

    → Alice taps [Approve]
    ⚡ POST /events/{eid}/modification-requests/{mrid}/approve
       → auto-applies the proposed changes (no separate PATCH needed)
       → positions recalculate
```

### 6. Final positions after weighted correction

```
S11 Balances (final)

    Person    Paid      Consumed    Net
    ──────    ────      ────────    ────
    Alice     $500.00   $140.00     +$360.00
    Bob       $0.00     $0.00       $0.00
    Carol     $0.00     $80.00      −$80.00
    Dave      $0.00     $140.00     −$140.00
    Eve       $0.00     $140.00     −$140.00
    ──────    ────      ────────    ────
    Total     $500.00   $500.00     $0.00 ✓

    Arithmetic: 140 + 80 + 140 + 140 = 500 ✓
    Weights: Alice 140/500, Carol 80/500, Dave 140/500, Eve 140/500
```

## Validates

- Aggressive defaults can be corrected with a simple edit (admin path)
- Members can propose corrections via modification requests (member path)
- Modification request auto-applies on approve — no separate API call
- Positions recalculate correctly after split changes
- Both equal and weighted corrections work through the same UI
- Zero-sum checksum maintained through all correction steps

## Key Principle Demonstrated

> **"Default boldly, correct easily."** Fair Go's aggressive defaults save time in the common case. When corrections are needed, the same split editor handles equal exclusions (remove Bob) and weighted adjustments (Carol's starter vs mains) — no delete-and-recreate, no manual calculations. The admin edits directly; members propose changes through the modification request system.
