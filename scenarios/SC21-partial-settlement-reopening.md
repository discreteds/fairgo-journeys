# SC21 — Partial Settlement & Event Reopening

> **Differentiator:** Real groups don't settle perfectly in one go. Someone can only pay half now. A forgotten expense surfaces after closing. Fair Go handles both without losing track.

**Persona:** Alice (admin) managing a ski trip where settlement doesn't go smoothly.
**Screens:** S09 → S11 → S12 → S05
**Rails:** R02 (Expense), R03 (Settlement)

## Design Principle

**"Life doesn't settle in one shot."**

Partial payments, forgotten expenses, and reopened events are normal. Fair Go tracks what's been paid, what's remaining, and recalculates correctly when new expenses appear — even after the event was "closed."

## Participants

| Person | Role | Settlement Group |
|--------|------|-----------------|
| Alice | Admin, payer | Alice 🔵 |
| Bob | Member, payer | Bob 🟢 |
| Carol | Member | Carol 🟡 |
| Dave | Member | Dave 🔴 |

Four people, all solo PFGs.

## Narrative

### 1. Expenses

Three expenses from the ski trip:

```
Expense 1: "Ski Passes" — Alice pays $800, split 4 ways equally
Expense 2: "Cabin Rental" — Bob pays $1,200, split 4 ways equally
Expense 3: "Groceries" — Alice pays $200, split 4 ways equally
```

```
⚡ POST /events/{eid}/transactions × 3
   Headers: Idempotency-Key: <uuid> (see A07)
```

### 2. Positions

```
S11 Balances

    Person    Paid        Consumed    Net
    ──────    ────        ────────    ────
    Alice     $1,000.00   $550.00     +$450.00
    Bob       $1,200.00   $550.00     +$650.00
    Carol     $0.00       $550.00     −$550.00
    Dave      $0.00       $550.00     −$550.00
    ──────    ────        ────────    ────
    Total     $2,200.00   $2,200.00   $0.00 ✓

    Per-person consumed: ($800 + $1200 + $200) / 4 = $550.00 each
```

### 3. Settlement suggestions

```
S12 Settle Up
    ⚡ GET /events/{eid}/settlement-suggestions

    Suggestion 1: Carol 🟡 → Bob 🟢    $550.00
    Suggestion 2: Dave 🔴 → Alice 🔵   $450.00
    Suggestion 3: Dave 🔴 → Bob 🟢     $100.00
                                        ─────────
    Total: $1,100.00

    Verification:
      Alice receives $450 ✓ (net +$450)
      Bob receives $550 + $100 = $650 ✓ (net +$650)
      Carol pays $550 ✓ (net −$550)
      Dave pays $450 + $100 = $550 ✓ (net −$550)
```

### 4. Settlements created and confirmed

Alice creates the settlements from the suggestions:

```
⚡ POST /events/{eid}/settlements
   Headers: Idempotency-Key: <uuid>
   {from_pfg: carol_pfg, to_pfg: bob_pfg, amount: 550.00}
   → settlement_id: s1, status: "proposed"

⚡ POST /events/{eid}/settlements/{s1}/confirm
   Headers: Idempotency-Key: <uuid>
   → status: "confirmed"

⚡ POST /events/{eid}/settlements
   Headers: Idempotency-Key: <uuid>
   {from_pfg: dave_pfg, to_pfg: alice_pfg, amount: 450.00}
   → settlement_id: s2, status: "proposed"

⚡ POST /events/{eid}/settlements/{s2}/confirm → status: "confirmed"

⚡ POST /events/{eid}/settlements
   Headers: Idempotency-Key: <uuid>
   {from_pfg: dave_pfg, to_pfg: bob_pfg, amount: 100.00}
   → settlement_id: s3, status: "proposed"

⚡ POST /events/{eid}/settlements/{s3}/confirm → status: "confirmed"
```

### 5. Partial payment — Dave can only pay half now

Carol pays Bob in full ($550). But Dave can only pay $275 of his $450 debt to Alice right now.

```
S12 Settle Up

    ⚡ POST /events/{eid}/settlements/{s1}/pay
       Headers: Idempotency-Key: <uuid>
       {amount: 550.00, method: "bank_transfer"}
       → s1 status: "completed"

    ⚡ POST /events/{eid}/settlements/{s2}/payments
       Headers: Idempotency-Key: <uuid>
       {amount: 275.00, method: "bank_transfer", reference: "Dave partial"}
       → s2 status: "partially_paid"
       → paid: 275.00, remaining: 175.00

S12 display:
    ┌──────────────────────────────────────────────────┐
    │  Carol → Bob:   $550.00  ✅ Paid                 │
    │  Dave → Alice:  $450.00  ⏳ $275 paid · $175 left │
    │  Dave → Bob:    $100.00  ⏳ Confirmed              │
    └──────────────────────────────────────────────────┘
```

### 6. Event closed

Alice decides to close the event — the remaining payments can continue outside the app.

```
S05 Event Dashboard → taps [Close Event]
    ⚡ POST /events/{eid}/close
       → event status: "closed"
       → settlement guard active: no new settlements can be created
       → existing settlements (including partial) preserved
```

### 7. Forgotten expense — event reopened

Two days later, Bob realises he forgot to add the $160 equipment rental he paid for.

```
Bob: "Hey Alice, I forgot to add the equipment rental — $160"

Alice → S05 Event Dashboard (closed state)
    → taps [Reopen Event]
    ⚡ POST /events/{eid}/reopen
       → event status: "active"
       → settlement guard lifted

S09 Add Expense (Bob)
    Description: "Equipment Rental"
    Amount: $160.00
    Paid by: Bob
    Split: Everyone equally ($40.00 each)

    ⚡ POST /events/{eid}/transactions
       Headers: Idempotency-Key: <uuid>
       {line_items: [{amount: "160.00", payer_person_id: bob_id,
         splits: [{bob, expense, 1}, {all 4, consumption, 1}]}]}
```

### 8. Recalculated positions (accounting for partial payments)

```
S11 Balances (after reopening + new expense)

    Person    Paid        Consumed    Net (raw)    Already Settled    Remaining
    ──────    ────        ────────    ─────────    ───────────────    ─────────
    Alice     $1,000.00   $590.00     +$410.00     +$275.00 received  +$135.00
    Bob       $1,360.00   $590.00     +$770.00     +$550.00 received  +$220.00
    Carol     $0.00       $590.00     −$590.00     −$550.00 paid      −$40.00
    Dave      $0.00       $590.00     −$590.00     −$275.00 paid      −$315.00
    ──────    ────        ────────    ─────────    ───────────────    ─────────
    Total     $2,360.00   $2,360.00   $0.00 ✓      $0.00 ✓           $0.00 ✓

    Per-person consumed: ($800 + $1200 + $200 + $160) / 4 = $590.00 each
```

### 9. New settlement suggestions (accounting for what's already paid)

```
S12 Settle Up → new suggestions account for completed/partial payments

    Remaining settlement 1: Carol 🟡 → Bob 🟢    $40.00    (was $550, paid $550, new debt $40)
    Remaining settlement 2: Dave 🔴 → Alice 🔵   $135.00   (was $450, paid $275, new debt −$40 net)
    Remaining settlement 3: Dave 🔴 → Bob 🟢     $180.00   (was $100, unpaid, new debt +$80)
                                                  ─────────
    Total remaining: $355.00

    Verification:
      Alice receives $135 → total received $275 + $135 = $410 ✓
      Bob receives $40 + $180 → total received $550 + $40 + $180 = $770 ✓
      Carol pays $40 → total paid $550 + $40 = $590 ✓
      Dave pays $135 + $180 → total paid $275 + $135 + $180 = $590 ✓
```

## Validates

- Partial payment tracking on confirmed settlements
- Settlement status lifecycle: proposed → confirmed → partially_paid → completed
- Event close with settlement guard (no new settlements on closed events)
- Event reopen restores full functionality
- Position recalculation preserves partial payment history
- Settlement suggestions account for already-paid amounts
- Zero-sum checksum maintained across raw positions, settled amounts, and remaining balances

## Key Principle Demonstrated

> **"Life doesn't settle in one shot."** Dave pays half now, the rest later. A forgotten expense surfaces after closing. Fair Go tracks every partial payment, reopens cleanly, recalculates with the new expense, and shows only what's still owed — no spreadsheets, no "who paid what again?"
