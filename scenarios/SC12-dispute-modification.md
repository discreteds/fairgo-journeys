# SC12 — Dispute and Modification Flow

**Persona:** Bob disagrees with how Alice split the dinner.
**Screens:** S10 → S16 → S05
**Rails:** R06 (Admin), R02 (Expense)

## Narrative

Bob opens the Restaurant Dinner expense (from SC11) and reviews the split:

```
Bob's S10 Expense Detail
    Restaurant Dinner — Total: $451.00
    Paid by: Alice

    Bob's share: $99.11
    Breakdown:
      Food:     $40.00 (equal split, 5 ways)
      Wine:     $50.00 (equal split, 3 ways)
      Service:  $9.11  (equal split, 5 ways)

    Bob thinks: "I only had one beer ($8), not $50 of drinks"
```

Bob creates a modification request:

```
Bob's S10 → taps [Dispute This Split]
S16 Modification Request (submission view)
    → Target: "Restaurant Dinner" (transaction)
    → Specific line item: "Wine & Beer" ($150.00)
    → Current split: Alice $50, Bob $50, Dave $50
    → Bob's message: "I only had one beer (~$8). Alice and Dave split
      the wine bottles. Suggest weights: Alice 7, Dave 7, Bob 1"
    ⚡ POST /events/{eid}/modification-requests
       {target_type: "line_item_split",
        target_id: line_item_id,
        message: "I only had one beer (~$8). Alice and Dave split the wine bottles. Suggest weights: Alice 7, Dave 7, Bob 1",
        suggested_changes: {
          consumption_splits: [
            {person_id: alice, weight: 7},
            {person_id: dave, weight: 7},
            {person_id: bob, weight: 1}
          ]
        }}
    → "Request sent to admin ✓"
    → status: pending
```

Alice sees the request in her moderation queue:

```
Alice's S05 → "⚠️ 1 modification request"
S16 Admin Moderation Queue
    → Bob's request:
      "Wine & Beer: Bob requests weight change"
      Current: 1:1:1 ($50/$50/$50)
      Proposed: 7:7:1 ($70/$70/$10)
      Bob's note: "I only had one beer..."

    → [Preview Changes] shows impact:
      Bob:   $99.11 → $59.11 (saves $40.00)
      Alice: $139.11 → $159.11 (pays $20 more)
      Dave:  $99.11 → $119.11 (pays $20 more)
      Checksum: still $0.00 ✓

    → Alice taps [Approve]
    ⚡ POST /events/{eid}/modification-requests/{rid}/resolve {status: "approved"}
    ⚡ PATCH /events/{eid}/transactions/{tid}/line-items/{lid}
       {consumption_splits: [{person_id: alice, weight: 7}, {person_id: dave, weight: 7}, {person_id: bob, weight: 1}]}
    → splits recalculated
    → Bob notified: "Your modification request was approved ✓"
```

Positions recalculate after the approved change:

```
S11 Balances (after modification)
    ⚡ GET /events/{eid}/positions/persons
    → Checksum: $0.00 ✓

    Alice:  paid $451.00, consumed $159.11 → is owed $291.89
    Bob:    paid $0,      consumed $59.11  → owes $59.11
    Carol:  paid $0,      consumed $69.11  → owes $69.11
    Dave:   paid $0,      consumed $119.11 → owes $119.11
    Eve:    paid $0,      consumed $44.56  → owes $44.56
```

## Variant: Settlement Voiding After Dispute

If settlements had already been created before the dispute:

```
S12 → existing settlement: Bob pays Alice $99.11 (status: confirmed)

Bob's dispute is approved → Bob's share changes to $59.11

S12 → settlement amount is now wrong ($99.11 ≠ $59.11)
    → ⚠️ "Settlement amount doesn't match current positions"
    → Alice voids the old settlement:
    ! POST /events/{eid}/settlements/{sid}/void
       → status: voided (preserved in history)
       → positions recalculate (voided settlement excluded)

    → Alice creates a corrected settlement:
    ⚡ POST /events/{eid}/settlements
       {from: bob_pfg, to: alice_pfg, amount: 59.11}
       → new settlement: proposed
```

## Variant: Alice Rejects the Request

```
Alice's S16 → reviews Bob's request
    → taps [Reject]
    → adds note: "You had three beers, not one. The split is fair."
    ⚡ POST /events/{eid}/modification-requests/{rid}/resolve
       {status: "rejected", note: "You had three beers, not one."}
    → Bob notified: "Your modification request was rejected"
    → Bob sees Alice's note
    → no splits changed
```

## Validates

- Transaction detail view with per-person split breakdown (S10)
- Modification request creation with suggested changes
- Admin notification and moderation queue (S16)
- Approval flow with split recalculation
- Rejection flow with admin notes
- Impact preview before approval (shows position changes)
- Checksum maintained after split modifications
- Settlement void-and-recreate when positions change (F1 gap)
- Notification to requesting member on resolution

## Key Principle Demonstrated

> **"Disputes are conversations, not conflicts"**
>
> Bob can't unilaterally change the split, but he can make a case.
> Alice sees the impact before deciding. The whole flow is transparent —
> proposed changes, preview, approve or reject with notes. Financial
> fairness requires communication, and the app facilitates it.
