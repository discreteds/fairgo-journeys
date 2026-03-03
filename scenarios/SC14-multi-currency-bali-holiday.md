# SC14 — Multi-Currency Bali Holiday

> **Variation of SC03.** Extends the shared-PFG holiday pattern with multi-currency expenses, FX conversion, and partial splits across 6 people and 3 home currencies.

**Persona:** Sam (admin) organising a 6-person Bali trip; 3 couples, each a settlement group, each with a different home currency.
**Screens:** S04 → S05 → S07 → S09 → S11 → S12
**Rails:** R01 (Onboarding), R02 (Expense), R03 (Settlement)

## Design Principle

**"One trip, three wallets, one truth."**

Sam, Pierre, and Ravi each pay in different currencies — AUD, CAD, USD — plus local IDR spend. Fair Go converts everything to a common base (IDR) using explicit FX rates, so each couple sees what they owe in their own currency. No spreadsheets, no "I think the exchange rate was..."

## Participants

| Person | Role | Settlement Group | Home Currency |
|--------|------|-----------------|---------------|
| Sam | Admin, payer | Sam & Jo 🔵 | AUD |
| Jo | Member, payer | Sam & Jo 🔵 | AUD |
| Pierre | Member, payer | Pierre & Chloé 🟢 | CAD |
| Chloé | Member | Pierre & Chloé 🟢 | CAD |
| Ravi | Member, payer | Ravi & Priya 🟡 | USD |
| Priya | Member, payer | Ravi & Priya 🟡 | USD |

## FX Rates

Set at event level via S05 → Settings or per-expense on S09:

| From | To IDR | Source |
|------|--------|--------|
| 1 AUD | 10,300 IDR | Manual (Wise mid-market) |
| 1 CAD | 7,500 IDR | Manual (Wise mid-market) |
| 1 USD | 15,800 IDR | Manual (Wise mid-market) |

All positions are computed in IDR (event base currency), then converted to each PFG's home currency for settlement display.

## Narrative

### 1. Event creation and people setup

```
S04 Create Event → "Bali Holiday 2026"
    → Currency: IDR (local currency as base)
    → taps "Create"
    ⚡ POST /events
       {name: "Bali Holiday 2026", currency: "IDR"}
       → event_id returned, Sam auto-added as admin

S05 Event Dashboard → taps "People ▸"
S07 Manage People → adds Jo, Pierre, Chloé, Ravi, Priya
    ⚡ POST /events/{eid}/persons × 5
```

### 2. PFG setup — three couples

```
S07 Manage People
    ┌─────────────────────────────────────────────────────┐
    │  💡 Do any of these people settle together?         │
    │     (e.g. couples sharing a bank account)           │
    │                                        [Add group]  │
    └─────────────────────────────────────────────────────┘

    → "Add group" × 3:
    ⚡ POST /events/{eid}/pfgs {name: "Sam & Jo", member_ids: [sam, jo]}
    ⚡ POST /events/{eid}/pfgs {name: "Pierre & Chloé", member_ids: [pierre, chloe]}
    ⚡ POST /events/{eid}/pfgs {name: "Ravi & Priya", member_ids: [ravi, priya]}

S07 Manage People (after grouping)
    🟢 Sam         🔵 Sam & Jo
    🟢 Jo          🔵 Sam & Jo
    🟡 Pierre      🟢 Pierre & Chloé
    🟡 Chloé       🟢 Pierre & Chloé
    🟡 Ravi        🟡 Ravi & Priya
    🟡 Priya       🟡 Ravi & Priya
```

### 3. Adding expenses — home-currency big-ticket items

**Expense 1: Hotel — AUD 2,400 (Sam), 6-way equal**

```
S09 Add Expense → "Hotel (4 nights)"
    → Amount: AUD 2,400.00
    → Currency: AUD (picker — not event default)
    → FX rate: 1 AUD = 10,300 IDR
    → Paid by: Sam
    → Split between: Everyone (6 people)
    → IDR equivalent: 24,720,000
    → Per person: IDR 4,120,000
    ⚡ POST /events/{eid}/transactions
       {description: "Hotel (4 nights)", currency: "AUD",
        line_items: [{amount: 2400.00,
          fx_rate_used: 10300, fx_source: "manual",
          expense_splits: [{person_id: sam, weight: 1, side: "expense"}],
          consumption_splits: [{person_id: sam, weight: 1},
                               {person_id: jo, weight: 1},
                               {person_id: pierre, weight: 1},
                               {person_id: chloe, weight: 1},
                               {person_id: ravi, weight: 1},
                               {person_id: priya, weight: 1}]}]}
```

**Expense 2: Car rental — CAD 1,200 (Pierre), 6-way equal**

```
S09 Add Expense → "Car Rental (1 week)"
    → Amount: CAD 1,200.00
    → Currency: CAD
    → FX rate: 1 CAD = 7,500 IDR
    → Paid by: Pierre
    → Split between: Everyone (6 people)
    → IDR equivalent: 9,000,000
    → Per person: IDR 1,500,000
    ⚡ POST /events/{eid}/transactions
       {description: "Car Rental (1 week)", currency: "CAD",
        line_items: [{amount: 1200.00,
          fx_rate_used: 7500, fx_source: "manual",
          expense_splits: [{person_id: pierre, weight: 1, side: "expense"}],
          consumption_splits: [all 6, weight: 1 each]}]}
```

**Expense 3: Cooking course — USD 480 (Ravi), 6-way equal**

```
S09 Add Expense → "Balinese Cooking Course"
    → Amount: USD 480.00
    → Currency: USD
    → FX rate: 1 USD = 15,800 IDR
    → Paid by: Ravi
    → Split between: Everyone (6 people)
    → IDR equivalent: 7,584,000
    → Per person: IDR 1,264,000
    ⚡ POST /events/{eid}/transactions
       {description: "Balinese Cooking Course", currency: "USD",
        line_items: [{amount: 480.00,
          fx_rate_used: 15800, fx_source: "manual",
          expense_splits: [{person_id: ravi, weight: 1, side: "expense"}],
          consumption_splits: [all 6, weight: 1 each]}]}
```

### 4. Adding expenses — IDR local group dinners

**Expense 4: Dinner at Locavore — IDR 1,800,000 (Sam), 6-way equal**

```
S09 Add Expense → "Dinner — Locavore"
    → Amount: IDR 1,800,000
    → Currency: IDR (event default — no FX needed)
    → Paid by: Sam
    → Split between: Everyone (6 people)
    → Per person: IDR 300,000
    ⚡ POST /events/{eid}/transactions
```

**Expense 5: Dinner at Mozaic — IDR 2,100,000 (Pierre), 6-way equal**

```
S09 Add Expense → "Dinner — Mozaic"
    → Amount: IDR 2,100,000
    → Paid by: Pierre
    → Per person: IDR 350,000
```

**Expense 6: Dinner at Sardine — IDR 1,500,000 (Ravi), 6-way equal**

```
S09 Add Expense → "Dinner — Sardine"
    → Amount: IDR 1,500,000
    → Paid by: Ravi
    → Per person: IDR 250,000
```

### 5. Adding expenses — IDR partial splits

**Expense 7: Beach bar — IDR 960,000 (Jo), 4 people only**

```
S09 Add Expense → "Beach Bar — Sundowners"
    → Amount: IDR 960,000
    → Paid by: Jo
    → Split between: Sam, Jo, Pierre, Chloé (custom — Ravi & Priya at hotel)
    → Per person: IDR 240,000
    ⚡ POST /events/{eid}/transactions
       {description: "Beach Bar — Sundowners",
        line_items: [{amount: 960000,
          expense_splits: [{person_id: jo, weight: 1, side: "expense"}],
          consumption_splits: [{person_id: sam, weight: 1},
                               {person_id: jo, weight: 1},
                               {person_id: pierre, weight: 1},
                               {person_id: chloe, weight: 1}]}]}
```

**Expense 8: Night bar — IDR 780,000 (Priya), 3 people only**

```
S09 Add Expense → "Night Bar — Jazz Club"
    → Amount: IDR 780,000
    → Paid by: Priya
    → Split between: Ravi, Priya, Sam (custom — others went home)
    → Per person: IDR 260,000
    ⚡ POST /events/{eid}/transactions
       {description: "Night Bar — Jazz Club",
        line_items: [{amount: 780000,
          expense_splits: [{person_id: priya, weight: 1, side: "expense"}],
          consumption_splits: [{person_id: ravi, weight: 1},
                               {person_id: priya, weight: 1},
                               {person_id: sam, weight: 1}]}]}
```

### 6. Multi-currency balance view

```
S11 Balances (by Settlement Group)
    ⚡ GET /events/{eid}/positions/pfgs

    🔵 Sam & Jo       is owed IDR 11,172,000  (≈ AUD 1,084.66)
       ├ Sam    +IDR 18,236,000
       └ Jo     −IDR 7,064,000

    🟢 Pierre & Chloé  owes IDR 4,948,000  (≈ CAD 659.73)
       ├ Pierre +IDR 3,076,000
       └ Chloé  −IDR 8,024,000

    🟡 Ravi & Priya    owes IDR 6,224,000  (≈ USD 393.92)
       ├ Ravi   +IDR 1,040,000
       └ Priya  −IDR 7,264,000
```

The multi-currency balance view (S11) shows each PFG's net in the event base currency (IDR) with a home-currency equivalent in parentheses. Within each PFG, individual members can see their contribution to the group position, but settlement happens at the PFG level.

### 7. Settlement suggestions

```
S12 Settle Up
    ⚡ GET /events/{eid}/settlement-suggestions
       → server computes at PFG level, shows home-currency equivalents

    Suggested settlements:
    🟢 Pierre & Chloé  pays IDR 4,948,000  →  🔵 Sam & Jo
       (CAD 659.73 at 1 CAD = 7,500 IDR)

    🟡 Ravi & Priya    pays IDR 6,224,000  →  🔵 Sam & Jo
       (USD 393.92 at 1 USD = 15,800 IDR)
```

Two transfers settle the entire holiday. Sam & Jo receive IDR 11,172,000 total (equivalent to AUD 1,084.66), exactly what they are owed. Each debtor couple sees the amount in their home currency — no manual FX conversion needed.

## Arithmetic Verification

### IDR conversion table

| # | Expense | Original | FX Rate | IDR Equivalent |
|---|---------|----------|---------|----------------|
| 1 | Hotel | AUD 2,400 | ×10,300 | 24,720,000 |
| 2 | Car Rental | CAD 1,200 | ×7,500 | 9,000,000 |
| 3 | Cooking Course | USD 480 | ×15,800 | 7,584,000 |
| 4 | Dinner — Locavore | IDR 1,800,000 | ×1 | 1,800,000 |
| 5 | Dinner — Mozaic | IDR 2,100,000 | ×1 | 2,100,000 |
| 6 | Dinner — Sardine | IDR 1,500,000 | ×1 | 1,500,000 |
| 7 | Beach Bar | IDR 960,000 | ×1 | 960,000 |
| 8 | Night Bar | IDR 780,000 | ×1 | 780,000 |
| | **Total** | | | **48,444,000** |

### Per-person consumed (all values IDR)

| Person | Hotel | Car | Cook | Din 1 | Din 2 | Din 3 | Beach | Night | **Total** |
|--------|-------|-----|------|-------|-------|-------|-------|-------|-----------|
| Sam | 4,120,000 | 1,500,000 | 1,264,000 | 300,000 | 350,000 | 250,000 | 240,000 | 260,000 | **8,284,000** |
| Jo | 4,120,000 | 1,500,000 | 1,264,000 | 300,000 | 350,000 | 250,000 | 240,000 | — | **8,024,000** |
| Pierre | 4,120,000 | 1,500,000 | 1,264,000 | 300,000 | 350,000 | 250,000 | 240,000 | — | **8,024,000** |
| Chloé | 4,120,000 | 1,500,000 | 1,264,000 | 300,000 | 350,000 | 250,000 | 240,000 | — | **8,024,000** |
| Ravi | 4,120,000 | 1,500,000 | 1,264,000 | 300,000 | 350,000 | 250,000 | — | 260,000 | **8,044,000** |
| Priya | 4,120,000 | 1,500,000 | 1,264,000 | 300,000 | 350,000 | 250,000 | — | 260,000 | **8,044,000** |
| **Column ✓** | 24,720,000 | 9,000,000 | 7,584,000 | 1,800,000 | 2,100,000 | 1,500,000 | 960,000 | 780,000 | **48,444,000** ✓ |

### Per-person paid (IDR)

| Person | Hotel | Car | Cook | Din 1 | Din 2 | Din 3 | Beach | Night | **Total** |
|--------|-------|-----|------|-------|-------|-------|-------|-------|-----------|
| Sam | 24,720,000 | — | — | 1,800,000 | — | — | — | — | **26,520,000** |
| Jo | — | — | — | — | — | — | 960,000 | — | **960,000** |
| Pierre | — | 9,000,000 | — | — | 2,100,000 | — | — | — | **11,100,000** |
| Chloé | — | — | — | — | — | — | — | — | **0** |
| Ravi | — | — | 7,584,000 | — | — | 1,500,000 | — | — | **9,084,000** |
| Priya | — | — | — | — | — | — | — | 780,000 | **780,000** |
| **Total** | | | | | | | | | **48,444,000** ✓ |

### Net positions (IDR)

| Person | Paid | Consumed | Net | Direction |
|--------|------|----------|-----|-----------|
| Sam | 26,520,000 | 8,284,000 | +18,236,000 | owed |
| Jo | 960,000 | 8,024,000 | −7,064,000 | owes |
| Pierre | 11,100,000 | 8,024,000 | +3,076,000 | owed |
| Chloé | 0 | 8,024,000 | −8,024,000 | owes |
| Ravi | 9,084,000 | 8,044,000 | +1,040,000 | owed |
| Priya | 780,000 | 8,044,000 | −7,264,000 | owes |
| **Checksum** | **48,444,000** | **48,444,000** | **0** | **✓** |

### PFG rollup

| Settlement Group | Members | Combined Net (IDR) | Home Currency |
|-----------------|---------|-------------------|---------------|
| 🔵 Sam & Jo | Sam (+18,236,000) + Jo (−7,064,000) | +11,172,000 (owed) | AUD 1,084.66 |
| 🟢 Pierre & Chloé | Pierre (+3,076,000) + Chloé (−8,024,000) | −4,948,000 (owes) | CAD 659.73 |
| 🟡 Ravi & Priya | Ravi (+1,040,000) + Priya (−7,264,000) | −6,224,000 (owes) | USD 393.92 |
| **Checksum** | | **0** | **✓** |

### Settlement path verification

- Pierre & Chloé pay IDR 4,948,000 → Sam & Jo: Sam & Jo running total = 4,948,000
- Ravi & Priya pay IDR 6,224,000 → Sam & Jo: Sam & Jo running total = 11,172,000
- Sam & Jo are owed 11,172,000 — fully settled ✓
- Transfers needed: **2** (optimal — one per debtor PFG)

## Validates

- Multi-currency event with 4 currencies (AUD, CAD, USD, IDR)
- FX rate entry via currency picker on S09 (`fx_rate_used`, `fx_source: "manual"`)
- IDR as event base currency with home-currency equivalents on S11 and S12
- Partial splits (Beach Bar: 4/6 people, Night Bar: 3/6 people)
- PFG rollup across couples with different home currencies
- Cross-currency settlement suggestions showing both IDR and home-currency amounts
- Arithmetic verified: all position checksums = 0, column totals = expense totals
- Settlement minimisation: 3 debtor/creditor PFGs → 2 transfers

## Key Principle Demonstrated

> **"One trip, three wallets, one truth."**
>
> Sam paid in AUD, Pierre in CAD, Ravi in USD, and everyone spent IDR locally.
> Without Fair Go: a spreadsheet nightmare of currency conversions and manual netting.
> With Fair Go: each couple sees one number in their own currency. Two transfers. Done.
