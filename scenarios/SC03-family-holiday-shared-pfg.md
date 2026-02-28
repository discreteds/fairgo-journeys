# SC03 — Family Holiday with Shared Settlement Group

**Persona:** Alice (admin) organising a trip where she and partner Carol settle together.
**Screens:** S04 → S05 → S07 → S11 → S12
**Rails:** R01 (Onboarding), R02 (Expense), R03 (Settlement)

## Narrative

Alice creates a trip event and adds everyone:

```
S04 Create Event → "Bali Trip"
    ⚡ event created, invite code generated
S05 Event Dashboard
    → taps "People ▸"
S07 Manage People → adds Bob, Carol, Dave, Eve, Frank as placeholders
    ⚡ POST /events/{eid}/persons × 5
    All created with singleton PFGs (the default)
```

Alice sets up the shared settlement group for her and Carol:

```
S07 → taps Carol's row → "Change Settlement Group"
    → picker shows: "Create new shared group"
    → names it "Alice & Partner"
    ⚡ POST /events/{eid}/groups {name: "Alice & Partner", is_singleton: false}
    ⚡ PUT /events/{eid}/persons/{carol_id}/pfg {group_id: new_group_id}
    → Carol now shows 🔵 Alice & Partner

S07 → taps Alice's row → "Change Settlement Group"
    → picker shows: "Alice & Partner" (existing)
    ⚡ PUT /events/{eid}/persons/{alice_id}/pfg {group_id: alice_partner_id}
    → Alice now shows 🔵 Alice & Partner
```

**Simplified with singleton auto-group (I3):** If Alice adds Carol directly to her singleton group instead of creating a new group first, the backend auto-creates a new non-singleton group with both members. Alice's original singleton remains untouched. This reduces the PFG setup from 4 steps to 2 (add member + reassign PFG).

People list now shows:

```
🟢 Alice       🔵 Alice & Partner
🟢 Carol       🔵 Alice & Partner
🟡 Bob         🔴 Bob (solo)
🟡 Dave        🔴 Dave (solo)
🟡 Eve         🔴 Eve (solo)
🟡 Frank       🔴 Frank (solo)
```

Alice adds expenses over the trip:

```
S09 → "Restaurant $620", paid: Alice, split: Everyone
S09 → "Airport Taxi $85", paid: Bob, split: Everyone
S09 → "Hotel $542", paid: Alice, split: Everyone
```

Viewing balances:

```
S11 Balances (By Settlement Group):
    🔵 Alice & Partner    owes $142.50
       ├ Alice    -$37.50
       └ Carol    -$105.00
    🔴 Bob                is owed $85.00
    🔴 Dave               owes $95.00
    🔴 Eve                owes $80.00
    🔴 Frank              is owed $132.50
```

Alice and Carol's debts are **combined** into a single settlement group. One payment covers both:

```
S12 Settle Up → suggested:
    🔵 Alice & Partner pays $142.50 → 🔴 Frank
    🔴 Dave pays $95.00 → 🔴 Bob ($85) + 🔴 Frank ($10)
    🔴 Eve pays $80.00 → 🔴 Frank
```

## Validates

- PFG reassignment via S07 ("Change Settlement Group")
- Visual colour coding: 🔵 shared vs 🔴 solo
- Combined balance display for shared settlement groups on S11
- Individual member contributions visible as sub-lines
- Settlement suggestions operate at PFG level — one payment for the couple
- The PFG principle: "How many credit cards?" — Alice & Carol = 1 credit card

## Key Principle Demonstrated

> **"How many mouths, how many credit cards?"**
>
> Alice and Carol are a couple — they settle from one bank account.
> They're both "mouths" (both consume) but one "credit card" (settle together).
> → Shared settlement group. One payment.
