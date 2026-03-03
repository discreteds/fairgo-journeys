# SC03 — Family Holiday: Shared Settlement Groups

> **Rework:** Replaces original SC03. Adds verified arithmetic, unequal splits, and simplified PFG discovery flow.

**Persona:** Mark (admin) organising a 4-adult holiday; Mark & Lisa are a couple who settle together.
**Screens:** S04 → S05 → S07 → S09 → S11 → S12
**Rails:** R01 (Onboarding), R02 (Expense), R03 (Settlement)

## Design Principle

**"How many mouths, how many credit cards?"**

Mark and Lisa are both "mouths" — they both consume. But they have one credit card — they settle from the same bank account. Fair Go models this exactly: two participants, one settlement group. At the end of the holiday, the couple owes one number and makes one transfer.

## Participants

| Person | Role | Settlement Group |
|--------|------|-----------------|
| Mark | Admin, payer | Mark & Lisa (shared PFG) |
| Lisa | Member, payer | Mark & Lisa (shared PFG) |
| Tom | Member, payer | Tom (solo) |
| Jane | Member, payer | Jane (solo) |

## Narrative

### 1. Event creation

```
S04 Create Event → "Family Holiday"
    → taps "Create"
    ⚡ POST /events  → event_id returned, Mark auto-added as admin
S05 Event Dashboard
    → taps "People ▸"
S07 Manage People → adds Lisa, Tom, Jane as placeholders
    ⚡ POST /events/{eid}/persons × 3
    All four participants now have singleton PFGs (the default)
```

### 2. PFG discovery — the app asks naturally

After adding people, the app surfaces a prompt — not a settings menu:

```
S07 Manage People
    ┌─────────────────────────────────────────────────────┐
    │  💡 Do any of these people settle together?         │
    │     (e.g. couples sharing a bank account)           │
    │                                        [Add group]  │
    └─────────────────────────────────────────────────────┘
```

Mark taps **Add group**:

```
S07 → "Add group" prompt appears
    → Mark selects: Mark ✓
    → Mark selects: Lisa ✓
    → optional name field — prefilled "Mark & Lisa"
    → taps "Done"
    ⚡ POST /events/{eid}/pfgs
       {name: "Mark & Lisa", member_ids: [mark_id, lisa_id]}
       → creates non-singleton PFG, assigns both persons in one call
```

This is **2 steps** (select members, confirm) — not 7. The app creates whatever backend entities it needs. Mark never sees the words "PFG" or "singleton."

### 3. People list after grouping

```
S07 Manage People
    🟢 Mark        🔵 Mark & Lisa
    🟢 Lisa        🔵 Mark & Lisa
    🟡 Tom         🔴 Tom (solo)
    🟡 Jane        🔴 Jane (solo)
```

The colour coding makes grouping immediately legible. 🔵 = shared; 🔴 = solo.

### 4. Adding expenses over the holiday

**Expense 1: Accommodation — Tom paid, 4-way equal**

```
S09 Add Expense → "Accommodation" / $800.00
    → Paid by: Tom
    → Split between: Everyone (4 people)
    → Weights: equal (1:1:1:1)
    → Each person's share: $200.00
    ⚡ POST /events/{eid}/transactions
```

**Expense 2: Groceries — Lisa paid, 4-way equal**

```
S09 Add Expense → "Groceries" / $240.00
    → Paid by: Lisa
    → Split between: Everyone (4 people)
    → Weights: equal (1:1:1:1)
    → Each person's share: $60.00
    ⚡ POST /events/{eid}/transactions
```

**Expense 3: Wine tasting — Mark paid, Mark & Tom only**

Lisa and Jane didn't attend. Custom split, 2 people only:

```
S09 Add Expense → "Wine Tasting" / $160.00
    → Paid by: Mark
    → Split between: Mark, Tom  (custom — tap to deselect Lisa and Jane)
    → Weights: equal (1:1)
    → Mark's share: $80.00
    → Tom's share: $80.00
    → Lisa: excluded (weight = 0)
    → Jane: excluded (weight = 0)
    ⚡ POST /events/{eid}/transactions
       consumption_splits: [{person_id: mark, weight: 1},
                            {person_id: tom,  weight: 1}]
```

**Expense 4: Activities — Jane paid, 4-way equal**

```
S09 Add Expense → "Activities" / $320.00
    → Paid by: Jane
    → Split between: Everyone (4 people)
    → Weights: equal (1:1:1:1)
    → Each person's share: $80.00
    ⚡ POST /events/{eid}/transactions
```

### 5. Individual positions before PFG grouping

```
S11 Balances (individual view — before PFG rollup)
    ⚡ GET /events/{eid}/positions/persons

    Mark:   paid $160,  consumed $420  →  owes $260
    Lisa:   paid $240,  consumed $340  →  owes $100
    Tom:    paid $800,  consumed $420  →  is owed $380
    Jane:   paid $320,  consumed $340  →  owes $20
```

Without PFG grouping, Mark owes $260 and Lisa owes $100 separately. As a couple sharing one account, they would need to make two transfers. That's the pain Fair Go solves.

### 6. The PFG magic moment — combined balance view

```
S11 Balances (by Settlement Group — default view)
    ⚡ GET /events/{eid}/positions/pfgs

    🔵 Mark & Lisa    owes $360.00
       ├ Mark   -$260.00
       └ Lisa   -$100.00

    🔴 Tom            is owed $380.00
    🔴 Jane           owes $20.00
```

Mark's $260 and Lisa's $100 collapse into a single line: **Mark & Lisa owe $360.** One number. One transfer. The sub-lines remain visible for transparency — Mark can see he's the bigger debtor within the couple — but settlement happens at the PFG level.

### 7. Settlement suggestions

```
S12 Settle Up
    ⚡ GET /events/{eid}/settlement-suggestions
       → server computes optimal payment paths at PFG level
       → minimises number of transfers (debt minimisation algorithm)

    Suggested settlements:
    🔵 Mark & Lisa  pays $360.00  →  🔴 Tom
    🔴 Jane         pays $20.00   →  🔴 Tom
```

Two transfers settle the entire holiday. Tom receives $380 total ($360 + $20), exactly what he is owed. Mark and Lisa make **one joint transfer** — they don't need to split it between themselves because that's an internal matter within their shared account.

## Arithmetic Verification

### Expenses and splits

| Expense | Amount | Paid by | Split | Per-person share |
|---------|--------|---------|-------|-----------------|
| Accommodation | $800 | Tom | Mark, Lisa, Tom, Jane (equal) | $200 each |
| Groceries | $240 | Lisa | Mark, Lisa, Tom, Jane (equal) | $60 each |
| Wine Tasting | $160 | Mark | Mark, Tom only (equal) | $80 each; Lisa & Jane $0 |
| Activities | $320 | Jane | Mark, Lisa, Tom, Jane (equal) | $80 each |
| **Total** | **$1,520** | | | |

### Per-person consumed

| Person | Accommodation | Groceries | Wine | Activities | Total consumed |
|--------|--------------|-----------|------|------------|---------------|
| Mark | $200 | $60 | $80 | $80 | **$420** |
| Lisa | $200 | $60 | $0 | $80 | **$340** |
| Tom | $200 | $60 | $80 | $80 | **$420** |
| Jane | $200 | $60 | $0 | $80 | **$340** |
| **Total** | $800 | $240 | $160 | $320 | **$1,520** ✓ |

### Net positions

| Person | Paid | Consumed | Net | Direction |
|--------|------|----------|-----|-----------|
| Mark | $160 | $420 | -$260 | owes $260 |
| Lisa | $240 | $340 | -$100 | owes $100 |
| Tom | $800 | $420 | +$380 | owed $380 |
| Jane | $320 | $340 | -$20 | owes $20 |
| **Checksum** | **$1,520** | **$1,520** | **$0.00** | **✓** |

### PFG rollup

| Settlement Group | Members | Combined net |
|----------------|---------|-------------|
| Mark & Lisa | Mark (-$260) + Lisa (-$100) | -$360 (owe $360) |
| Tom (solo) | Tom (+$380) | +$380 (owed $380) |
| Jane (solo) | Jane (-$20) | -$20 (owes $20) |
| **Checksum** | | **$0.00** ✓ |

### Settlement path verification

- Mark & Lisa pay $360 → Tom: Tom's running total = $360
- Jane pays $20 → Tom: Tom's running total = $380
- Tom is owed $380 — fully settled ✓
- Transfers needed: **2** (optimal — one per debtor PFG)

## Validates

- PFG creation via natural-language discovery prompt (not a settings menu)
- Simplified 2-step PFG setup: select members → confirm
- Visual colour coding: 🔵 shared PFG vs 🔴 solo PFG on S07
- Unequal splits: Wine Tasting excludes Lisa and Jane (weight = 0)
- Mixed payers across 4 expenses
- Individual balance view showing Mark and Lisa separately
- PFG rollup view combining Mark & Lisa into a single $360 balance
- Sub-line breakdown within PFG (individual contributions remain visible)
- Settlement suggestions at PFG level — one transfer for the couple
- Arithmetic verified: all position checksums = $0.00

## Key Principle Demonstrated

> **"How many mouths, how many credit cards?"**
>
> Mark and Lisa both consume (two mouths) but settle from one account (one credit card).
> Without PFG grouping: two separate balances, two transfers.
> With PFG grouping: one combined balance, one transfer.
> The complexity is absorbed by the app, not by the couple.
