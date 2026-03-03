# SC17 — Work Lunch: Penny-Exact Arithmetic

> **Renumbered:** Originally proposed as "SC05" in feedback files. Renumbered to SC17 to avoid collision with existing SC05 (Settle and Close).

**Persona:** Frank (admin/payer) — splitting a quick work lunch with two colleagues.
**Screens:** S05 → S09 → S05 → S11 → S12
**Rails:** R02 (Expense), R03 (Settlement)

## Design Principle

**Fast and exact.** This scenario is deliberately small. Three people, one expense, one equal split. The entire flow takes under two minutes. The point is not the interface complexity — it is the arithmetic. When a bill does not divide evenly into whole cents, Fair Go allocates every cent deterministically. Nothing is rounded away. Nothing disappears into rounding drift. The total is exact.

## The Situation

Eve, Frank, and Grace grab lunch together. Frank puts the $47.30 bill on his card. They want to split it equally. The problem: $47.30 does not divide evenly by three.

```
$47.30 ÷ 3 = $15.7666...
```

This is the rounding problem every expense-splitting app must solve. There are three naive approaches, all wrong:

| Approach | Result | Problem |
|----------|--------|---------|
| Round each share up | $15.77 × 3 = **$47.31** | Phantom cent — one cent never existed |
| Round each share down | $15.76 × 3 = **$47.28** | Two cents vanish — the bill is underpaid |
| Round two up, one down | Depends on which person gets which | Non-deterministic unless the rule is specified |

Fair Go uses the **largest-remainder method** — a deterministic algorithm that allocates every cent, no more and no less.

## Largest-Remainder Method Explained

The algorithm proceeds in three steps:

**Step 1: Floor allocation**
Give everyone the floor (rounded-down) share:

```
floor($47.30 ÷ 3) = floor($15.7666...) = $15.76
Subtotal: $15.76 × 3 = $47.28
```

**Step 2: Count the remainder**
How many extra cents need to be distributed?

```
$47.30 − $47.28 = $0.02  →  2 extra cents
```

**Step 3: Distribute remainder cents**
Award one extra cent to the participants with the largest fractional remainders. All three participants have the same fractional remainder ($0.0066...) so the tiebreaker is alphabetical order. The first two participants alphabetically each receive one extra cent:

```
Eve   → $15.76 + $0.01 = $15.77  (first alphabetically)
Frank → $15.76 + $0.01 = $15.77  (second alphabetically)
Grace → $15.76 + $0.00 = $15.76  (no remainder cent)
```

**Verification:** $15.77 + $15.77 + $15.76 = **$47.30 ✓**

The bill is paid exactly. No phantom cent. No vanishing change.

## Flow Diagram

```
S05 Event Dashboard (Frank is already in an event, or creates one)
        │
        ↓
S09 Add Expense → "Tuesday Lunch" / $47.30 / paid: Frank / split: Everyone equal
        │
        ↓ system applies largest-remainder allocation
        │
S05 Event Dashboard → balances shown, conservation verified ✓
        │
        ↓
S11 Balances
        │
        ↓
S12 Settle Up → Eve pays Frank $15.77 / Grace pays Frank $15.76
```

## Narrative

### The Lunch Event

Frank has already created the "Tuesday Lunch" event and added Eve and Grace as participants. (Event creation and people-adding follow the same pattern as SC01 — not repeated here, as this scenario's focus is the arithmetic.)

```
S05 Event Dashboard → 3 people (Eve, Frank, Grace), 0 expenses
    → Frank taps "+ Add Expense"
```

### Entering the Expense

Frank enters a single expense: the full bill amount, paid by him, split equally among all three.

```
S09 Add Expense
    Description: "Tuesday Lunch"
    Amount:      $47.30
    Paid by:     Frank  ← default (current user), correct
    Split:       Everyone equal (Eve, Frank, Grace — weight 1 each)

    ┌──────────────────────────────────────────────────────────┐
    │  Split preview (largest-remainder applied):              │
    │                                                          │
    │  Eve     $15.77  ●──────────────────────────────────     │
    │  Frank   $15.77  ●──────────────────────────────────     │
    │  Grace   $15.76  ●─────────────────────────────────      │
    │                                                          │
    │  Total:  $47.30  ✓  Conservation verified ✓             │
    └──────────────────────────────────────────────────────────┘

    → taps "Save Expense"

    ⚡ POST /events/{eid}/transactions
       {description: "Tuesday Lunch",
        line_items: [{
          description: "Tuesday Lunch",
          amount: "47.30",
          payer_person_id: frank_id,
          splits: [
            {person_id: eve_id,   side: "expense",      weight: 0},
            {person_id: frank_id, side: "expense",      weight: 1},
            {person_id: eve_id,   side: "consumption",  weight: 1},
            {person_id: frank_id, side: "consumption",  weight: 1},
            {person_id: grace_id, side: "consumption",  weight: 1}
          ]
        }]}

    API response includes calculated shares (server applies largest-remainder):
       eve_share:   "15.77"
       frank_share: "15.77"
       grace_share: "15.76"
       total_check: "47.30"  ← conservation invariant confirmed server-side
```

### Dashboard: Balances Appear

Frank is returned to the event dashboard. Balances are immediately visible.

```
S05 Event Dashboard
    1 expense ("Tuesday Lunch"), $47.30 total

    ┌──────────────────────────────────────────────────────────┐
    │  Tuesday Lunch                                           │
    │                                                          │
    │  🟢 Frank   is owed $31.53                               │
    │     (paid $47.30, consumed $15.77)                       │
    │  🔴 Eve     owes $15.77                                  │
    │  🔴 Grace   owes $15.76                                  │
    │                                                          │
    │  Conservation verified ✓                                 │
    └──────────────────────────────────────────────────────────┘

    → taps "Balances" to see the full breakdown
```

### Balances Screen

```
S11 Balances

    🟢 Frank    is owed $31.53
    🔴 Eve      owes $15.77
    🔴 Grace    owes $15.76

    Checksum: −$31.53 + $15.77 + $15.76 = $0.00 ✓

    → taps "Settle Up"
```

### Settlement

```
S12 Settle Up
    ⚡ GET /events/{eid}/settlement-suggestions
    → server returns minimised payment plan:

    Suggested settlements:
    Eve   → Frank   $15.77
    Grace → Frank   $15.76

    → Frank taps [Create Settlement] for each
    ⚡ POST /events/{eid}/settlements × 2
       {from: eve_id,   to: frank_id, amount: "15.77"}
       {from: grace_id, to: frank_id, amount: "15.76"}

    → Frank confirms both
    ⚡ POST /events/{eid}/settlements/{sid}/confirm × 2

    → Eve and Grace each tap [Mark as Paid]
    ⚡ POST /events/{eid}/settlements/{sid}/pay × 2

    Final: all balances zero. $47.30 recovered exactly.
```

## Arithmetic Verification

| Person | Consumed | Paid | Net Position |
|--------|----------|------|-------------|
| Eve | $15.77 | $0.00 | **−$15.77** (owes) |
| Frank | $15.77 | $47.30 | **+$31.53** (owed) |
| Grace | $15.76 | $0.00 | **−$15.76** (owes) |
| **Σ** | **$47.30** | **$47.30** | **$0.00 ✓** |

Checksum: +$31.53 − $15.77 − $15.76 = **$0.00 ✓**

The split summary displayed by the app:

```
┌─────────────────────────────────────────────────────────────┐
│  Eve: $15.77,  Frank: $15.77,  Grace: $15.76                │
│  Total: $47.30  ✓                                           │
│  Conservation verified ✓                                    │
└─────────────────────────────────────────────────────────────┘
```

## Validates

- **Penny-exact arithmetic** — the core "accounting-grade" claim
- **Largest-remainder allocation** — deterministic, alphabetical tiebreaker, not random
- **Conservation invariant** — $15.77 + $15.77 + $15.76 = $47.30 exactly, verified server-side
- **Fast flow** — event + expense + balances + settlement in under 2 minutes
- **Small group, simple split** — Fair Go is not overkill for a quick lunch
- **Payer defaults to current user** — Frank opens the expense screen and his name is pre-selected
- **Equal split as the default** — no configuration needed; "Everyone, equal weight" is the out-of-the-box behaviour
- **Settlement minimisation** — two simple payments, no cross-owing

## Differentiator Moment

The split summary shows **$47.30 exactly**. Not $47.31. Not $47.28.

This is subtle, but it matters. Splitwise would show $15.77 × 3 = **$47.31** — a phantom cent that never gets paid, that exists nowhere in the real world, that someone absorbs silently or that accumulates across dozens of splits into a non-trivial error. In spreadsheets, someone always manually adjusts one person's share to make the total work, and that adjustment is invisible to everyone else.

Fair Go allocates every cent deterministically. The algorithm is the same every time: floor first, remainder distributed alphabetically. Eve and Frank each absorb one extra cent not because someone decided to be generous, but because the algorithm says so — and the algorithm is the same for every split, for every event, for every user.

The "Conservation verified ✓" label in the split summary is not decorative. It is the server confirming that the sum of all allocated shares equals the original amount to the cent. This check runs on every expense save. If it ever failed — due to a floating-point bug, a rounding error, a database truncation — the server would reject the transaction. The label means the invariant held.

> **"Splitwise would show $15.77 × 3 = $47.31 — a phantom cent that never gets paid."**
>
> **Fair Go shows $47.30. Because that is what the bill was.**
