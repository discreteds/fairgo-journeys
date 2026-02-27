# SC05 — Settle and Close

**Persona:** Alice (admin) closing out an event after all debts are settled.
**Screens:** S11 → S12 → S05
**Rails:** R03 (Settlement)

## Narrative

The event has run its course. Alice opens balances:

```
S11 Balances → outstanding debts visible
    🔵 Alice & Partner owes $142.50
    🔴 Dave owes $95.00
    🔴 Eve owes $80.00
    → taps "Settle Up"
```

Creating settlements:

```
S12 Settle Up → suggested settlements shown
    → taps [Create Settlement] for each suggested pair
    ⚡ POST /events/{eid}/settlements × 3
       {from: Alice&Partner, to: Frank, amount: 142.50}
       {from: Dave, to: Bob, amount: 85.00}
       {from: Dave, to: Frank, amount: 10.00}
       {from: Eve, to: Frank, amount: 80.00}
    → all created with status: proposed
```

Alice confirms each settlement (admin action):

```
S12 → [Confirm] on each proposed settlement
    ⚡ POST /events/{eid}/settlements/{sid}/confirm × 4
    → all now status: confirmed
```

Each payer marks their payment as done:

```
Alice (for Alice & Partner):
S12 → [Mark as Paid]
    ⚡ POST /events/{eid}/settlements/{sid}/pay
    → status: paid

Dave:
S12 → [Mark as Paid] on both his settlements
    ⚡ POST /events/{eid}/settlements/{sid}/pay × 2

Eve:
S12 → [Mark as Paid]
    ⚡ POST /events/{eid}/settlements/{sid}/pay
```

All settled:

```
S11 Balances → all zeroed out
    Checksum: $0.00 ✓
    "All settled ✓"

S05 Event Dashboard → "All settled ✓" badge on balance card
```

Alice closes the event:

```
S05 → ⚙️ Event Settings → [Close Event]
    Confirm: "Close this event? No more expenses or settlements can be added."
    ⚡ POST /events/{eid}/close
S03 Home → event shows as closed/archived with muted styling
```

## Validates

- Full settlement lifecycle: proposed → confirmed → paid
- Admin confirms, payer marks paid (role-appropriate actions)
- Multiple settlements created from suggestions
- Balance zeroing after all settlements paid
- Event closure flow
- Home screen reflecting closed event status

## Settlement Status Visibility

```
S12 shows the status flow clearly:

  ⏳ Proposed    → [Confirm] (admin)
  ✅ Confirmed   → [Mark as Paid] (payer or admin)
  💰 Paid        → (done, no actions)
```
