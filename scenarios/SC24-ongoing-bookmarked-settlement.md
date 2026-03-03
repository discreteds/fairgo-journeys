# SC24 — Ongoing Bookmarked Settlement

> **Differentiator:** Household expenses don't end. Unlike one-off events that close after settlement, ongoing events keep running — settle the current period, bookmark it, and keep going. No new events, no lost history.

**Persona:** Alex & Sam — couple sharing household expenses month to month. Equal split.
**Screens:** S04 → S05 → S09 (×3) → S11 → S12 → S05 → S09 (×2) → S11 → S12
**Rails:** R01 (Onboarding), R02 (Expense), R03 (Settlement)

## Design Principle

**"Your household doesn't close at the end of the month."** Ongoing events model continuous financial relationships. Bookmarked settlements mark the boundary between periods — everything before the bookmark is settled, everything after is the current period. The event stays active indefinitely.

## Participants

| Person | Role | Settlement Group |
|--------|------|-----------------|
| Alex | Admin, payer | Alex 🔵 |
| Sam | Member, payer | Sam 🟢 |

Two people, solo PFGs. Equal split on all expenses.

## Narrative

### 1. Create ongoing event

```
S04 Create Event
    Event Name: "McKinnon Household"
    Currency: AUD
    Type: (•) Ongoing   ← key distinction

    ⚡ POST /events
       {name: "McKinnon Household", base_currency: "AUD", event_type: "ongoing"}
       → event created, Alex is admin + person

S05 Event Dashboard
    📅 McKinnon Household · Ongoing
    1 person · 0 expenses

    → adds Sam as person
    ⚡ POST /events/{eid}/persons {display_name: "Sam"}
```

### 2. Month 1 — Three expenses

```
S09 Add Expense × 3

    Expense 1: "Rent — January"
    Amount: $2,400.00, paid by Alex, split equally
    ⚡ POST /events/{eid}/transactions
       Headers: Idempotency-Key: <uuid>
       {line_items: [{description: "Rent — January", amount: "2400.00",
         payer_person_id: alex_id,
         splits: [{alex, expense, 1}, {alex, consumption, 1}, {sam, consumption, 1}]}]}

    Expense 2: "Electricity — January"
    Amount: $180.00, paid by Alex, split equally
    ⚡ POST /events/{eid}/transactions
       Headers: Idempotency-Key: <uuid>
       {line_items: [{description: "Electricity — January", amount: "180.00",
         payer_person_id: alex_id,
         splits: [{alex, expense, 1}, {alex, consumption, 1}, {sam, consumption, 1}]}]}

    Expense 3: "Groceries — January"
    Amount: $620.00, paid by Sam, split equally
    ⚡ POST /events/{eid}/transactions
       Headers: Idempotency-Key: <uuid>
       {line_items: [{description: "Groceries — January", amount: "620.00",
         payer_person_id: sam_id,
         splits: [{sam, expense, 1}, {alex, consumption, 1}, {sam, consumption, 1}]}]}
```

### 3. Month 1 — Balances

```
S11 Balances

    Person    Paid         Consumed     Net
    ──────    ────         ────────     ────
    Alex      $2,580.00    $1,600.00    +$980.00
    Sam       $620.00      $1,600.00    −$980.00
    ──────    ────         ────────     ────
    Total     $3,200.00    $3,200.00    $0.00 ✓

    Per-person consumed: ($2,400 + $180 + $620) / 2 = $1,600.00 each
```

### 4. Month 1 — Settle Current Period

```
S12 Settle Up

    📅 Current Period
    Since event created: 15 Jan 2026
    3 new transactions

    Suggested Settlement:
      Sam 🟢 → Alex 🔵  $980.00

    → taps [Settle Current Period]

    ⚡ POST /events/{eid}/settlements
       Headers: Idempotency-Key: <uuid>
       {from_pfg: sam_pfg, to_pfg: alex_pfg, amount: "980.00", currency: "AUD"}
       → settlement created, status: "proposed"
       → settled_through_date: "2026-01-31T23:59:59Z"

    ⚡ POST /events/{eid}/settlements/{s1}/confirm
       Headers: Idempotency-Key: <uuid>
       → status: "confirmed"

    ⚡ POST /events/{eid}/settlements/{s1}/pay
       Headers: Idempotency-Key: <uuid>
       → status: "paid"
       → bookmark created at settled_through_date
```

### 5. Month 2 — Two more expenses

The event stays active. Alex and Sam continue adding expenses.

```
S05 Event Dashboard
    📅 McKinnon Household · Ongoing
    Current period: since 31 Jan 2026

S09 Add Expense × 2

    Expense 4: "Rent — February"
    Amount: $2,400.00, paid by Alex, split equally
    ⚡ POST /events/{eid}/transactions
       Headers: Idempotency-Key: <uuid>
       {line_items: [{description: "Rent — February", amount: "2400.00",
         payer_person_id: alex_id,
         splits: [{alex, expense, 1}, {alex, consumption, 1}, {sam, consumption, 1}]}]}

    Expense 5: "Internet — February"
    Amount: $80.00, paid by Sam, split equally
    ⚡ POST /events/{eid}/transactions
       Headers: Idempotency-Key: <uuid>
       {line_items: [{description: "Internet — February", amount: "80.00",
         payer_person_id: sam_id,
         splits: [{sam, expense, 1}, {alex, consumption, 1}, {sam, consumption, 1}]}]}
```

### 6. Month 2 — Balances (current period only)

S11 shows only month 2 balances — transactions before the bookmark are already settled.

```
S11 Balances

    📅 Current period: since 31 Jan 2026

    Person    Paid         Consumed     Net
    ──────    ────         ────────     ────
    Alex      $2,400.00    $1,240.00    +$1,160.00
    Sam       $80.00       $1,240.00    −$1,160.00
    ──────    ────         ────────     ────
    Total     $2,480.00    $2,480.00    $0.00 ✓

    Per-person consumed (period 2): ($2,400 + $80) / 2 = $1,240.00 each
```

### 7. Month 2 — Settle Current Period

```
S12 Settle Up

    📅 Current Period
    Since last settlement: 31 Jan 2026
    2 new transactions

    Suggested Settlement:
      Sam 🟢 → Alex 🔵  $1,160.00

    → taps [Settle Current Period]

    ⚡ POST /events/{eid}/settlements
       Headers: Idempotency-Key: <uuid>
       {from_pfg: sam_pfg, to_pfg: alex_pfg, amount: "1160.00", currency: "AUD"}
       → settlement created, status: "proposed"
       → settled_through_date: "2026-02-28T23:59:59Z"

    ⚡ POST /events/{eid}/settlements/{s2}/confirm → status: "confirmed"
    ⚡ POST /events/{eid}/settlements/{s2}/pay → status: "paid"
       → second bookmark created
```

### 8. Event stays active

```
S05 Event Dashboard
    📅 McKinnon Household · Ongoing
    Current period: since 28 Feb 2026
    0 new transactions

    The event is NOT closed. Alex and Sam will continue
    adding March expenses when they arrive.
```

## Arithmetic Verification — Period 1

| Person | Rent $2,400 | Electricity $180 | Groceries $620 | Total Consumed | Paid | Net |
|--------|-------------|-------------------|----------------|----------------|------|-----|
| Alex | $1,200.00 | $90.00 | $310.00 | $1,600.00 | $2,580.00 | **+$980.00** |
| Sam | $1,200.00 | $90.00 | $310.00 | $1,600.00 | $620.00 | **−$980.00** |
| **Σ** | **$2,400.00** | **$180.00** | **$620.00** | **$3,200.00** | **$3,200.00** | **$0.00 ✓** |

Settlement: Sam pays Alex $980.00 → checksum $0.00 ✓

## Arithmetic Verification — Period 2

| Person | Rent $2,400 | Internet $80 | Total Consumed | Paid | Net |
|--------|-------------|--------------|----------------|------|-----|
| Alex | $1,200.00 | $40.00 | $1,240.00 | $2,400.00 | **+$1,160.00** |
| Sam | $1,200.00 | $40.00 | $1,240.00 | $80.00 | **−$1,160.00** |
| **Σ** | **$2,400.00** | **$80.00** | **$2,480.00** | **$2,480.00** | **$0.00 ✓** |

Settlement: Sam pays Alex $1,160.00 → checksum $0.00 ✓

## Cumulative Verification

| Period | Expenses | Sam → Alex | Running Total Settled |
|--------|----------|------------|----------------------|
| 1 (Jan) | $3,200.00 | $980.00 | $980.00 |
| 2 (Feb) | $2,480.00 | $1,160.00 | $2,140.00 |

Total expenses across both periods: $5,680.00. Total settled: $2,140.00 (Sam's total share = $2,840.00 consumed − $700.00 paid = $2,140.00 net owed ✓).

## Validates

- **Ongoing event creation:** `event_type: "ongoing"` on S04
- **Bookmarked partial settlement** with `settled_through_date`
- **Period indicator on S12:** "Since last settlement: ..." with transaction count
- **Balance calculation scoped to current period** — month 2 balances exclude month 1 (already settled)
- **Event stays active after settlement** — no close, no "All settled" badge on ongoing events
- **Period indicator on S05:** "Current period: since {date}"
- **Two independent settlement periods** with verified arithmetic for each
- **Cumulative consistency** — total settled across periods matches expected net obligations

## Key Moment

The payoff is step 5: after settling January, Alex opens the app in February and the dashboard says "Current period: since 31 Jan 2026, 0 new transactions." The slate is clean for the new month — but all history is preserved. No new event needed, no lost data.

> **"Same household, new month. The books are clean."**
