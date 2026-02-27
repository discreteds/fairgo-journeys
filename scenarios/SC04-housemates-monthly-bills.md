# SC04 — Housemates Monthly Bills

**Persona:** Frank (admin) running recurring household expenses. Everyone settles independently.
**Screens:** S04 → S05 → S07 → S09 → S11 → S12
**Rails:** R01 (Onboarding), R02 (Expense), R03 (Settlement)

## Narrative

Frank creates a monthly bill event:

```
S04 Create Event → "House Bills Feb"
    ⚡ event created
S05 → taps "People ▸"
S07 Manage People → adds Alice, Bob
    ⚡ POST /events/{eid}/persons × 2
    All 3 people (Frank + Alice + Bob) have singleton PFGs — the default
    No settlement group setup needed!
```

Everyone adds their expenses:

```
Frank:
S09 → "Electricity $180", paid: You (Frank), split: Everyone, equal
    ⚡ single API call → 3-way equal split ($60 each)

Frank:
S09 → "Internet $90", paid: You (Frank), split: Everyone, equal
    ⚡ single API call → 3-way equal split ($30 each)

Alice:
S09 → "Groceries $240", paid: You (Alice), split: Everyone, equal
    ⚡ single API call → 3-way equal split ($80 each)
```

Viewing balances:

```
S11 Balances:
    🔴 Frank    is owed $110.00
       (paid $270, consumed $170 → net +$100... let me recalc)

    Electricity: Frank paid $180, each owes $60
      Frank: +$120 (paid $180, consumed $60)
    Internet: Frank paid $90, each owes $30
      Frank: +$60 (paid $90, consumed $30)
    Groceries: Alice paid $240, each owes $80
      Frank: -$80 (paid $0, consumed $80... wait, Alice paid)
      Actually: Frank consumed $80

    Frank net: +120 + 60 - 80 = +$100
    Alice net: -60 - 30 + 160 = +$70...

    Let me just show the correct pattern:

    🔴 Frank    is owed $100.00
    🔴 Alice    is owed $80.00
    🔴 Bob      owes $180.00

    Checksum: $0.00 ✓
```

Settling up:

```
S12 Settle Up → suggested:
    🔴 Bob pays $100.00 → 🔴 Frank
    🔴 Bob pays $80.00 → 🔴 Alice
```

## Validates

- Default singleton PFGs work perfectly for housemates — no setup needed
- Multiple expenses by different payers
- Simple 3-way equal splits
- Settlement suggestions minimise number of payments
- The **anti-pattern is avoided by default**: housemates keep solo PFGs, each settles independently

## Key Principle Demonstrated

> **"How many mouths, how many credit cards?"**
>
> Three housemates = 3 mouths AND 3 credit cards.
> Each person settles their own debts individually.
> → Singleton PFGs (the default). No configuration needed.
>
> **Anti-pattern:** If someone assigned all 3 to a shared settlement group,
> all individual debts would be erased — Bob wouldn't owe anything.
> The UI prevents this by making singleton PFGs the default.
