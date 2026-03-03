# SC25 — Dinner Decomposition Pipeline

> **Differentiator:** When a couple pays as one unit at a friends' dinner, they still need to split their share between themselves. Decomposition bridges the gap between public-facing shared PFGs and private household accounting — automatically.

**Persona:** Alex & Sam — couple who go to dinner with friends Dave and Lisa. Alex & Sam share a PFG in the dinner event. After settling, they decompose their share into the ongoing household event.
**Screens:** S05 (dinner) → S09 → S11 → S12 → [Decompose] → S05 (household)
**Rails:** R02 (Expense), R03 (Settlement), R09 (Decomposition)

## Design Principle

**"Settle as a unit, split at home."** At the restaurant, the couple presents as one financial entity — they don't want to complicate the group split by explaining their internal arrangements. But at home, they still need to account for who owes what. Decomposition connects these two worlds seamlessly.

## Participants

### Dinner Event — "Friday Dinner with Dave & Lisa"

| Person | Settlement Group | Notes |
|--------|-----------------|-------|
| Alex | 🔵 A&S (shared PFG) | Couple, admin |
| Sam | 🔵 A&S (shared PFG) | Couple |
| Dave | 🔴 Dave (singleton) | Friend, payer |
| Lisa | 🟡 Lisa (singleton) | Friend |

### Household Event — "McKinnon Household" (ongoing, pre-existing)

| Person | Settlement Group | Notes |
|--------|-----------------|-------|
| Alex | Alex (singleton) | Admin |
| Sam | Sam (singleton) | Member |

## Narrative

### 1. Dinner event setup

The dinner event already exists with 4 people. Alex and Sam share a PFG ("A&S") — they settle with friends as a unit.

```
S05 Event Dashboard ("Friday Dinner with Dave & Lisa")
    4 people · 0 expenses
    Groups: A&S (Alex, Sam), Dave (solo), Lisa (solo)
    PFGs: 🔵 A&S, 🔴 Dave, 🟡 Lisa
```

### 2. Dave pays the $400 dinner

Dave puts the entire dinner on his card. One transaction, one line item, split equally across all 4 people.

```
S09 Add Expense
    Description: "Dinner"
    Paid by: Dave
    Line item 1:
      Description: "Dinner"
      Amount: $400.00
      Split: Everyone equally (Alex, Sam, Dave, Lisa)
      Preview: $100.00 each

    ⚡ POST /events/{eid}/transactions
       Headers: Idempotency-Key: <uuid>
       {description: "Dinner",
        line_items: [{
          description: "Dinner", amount: "400.00",
          payer_person_id: dave_id,
          splits: [
            {person_id: dave_id, side: "expense", weight: 1},
            {person_id: alex_id, side: "consumption", weight: 1},
            {person_id: sam_id, side: "consumption", weight: 1},
            {person_id: dave_id, side: "consumption", weight: 1},
            {person_id: lisa_id, side: "consumption", weight: 1}
          ]
        }]}
```

### 3. Balances — PFG-level view

```
S11 Balances

    By Settlement Group:
      🔵 A&S          owes $200.00
         ├ Alex     -$100.00
         └ Sam      -$100.00
      🔴 Dave         is owed $300.00
      🟡 Lisa         owes $100.00

    Checksum: $0.00 ✓
```

Note: A&S owes $200 as a unit (Alex $100 + Sam $100). Dave is owed $300 ($400 paid − $100 consumed). Lisa owes $100.

### 4. Settlement — PFG-level

Settlements are between PFGs, not individuals. A&S settles with Dave as one payment.

```
S12 Settle Up
    ⚡ GET /events/{eid}/settlement-suggestions

    Suggestion 1: 🔵 A&S → 🔴 Dave     $200.00
    Suggestion 2: 🟡 Lisa → 🔴 Dave     $100.00
                                         ─────────
    Total: $300.00

    → Alex creates both settlements

    ⚡ POST /events/{eid}/settlements
       Headers: Idempotency-Key: <uuid>
       {from_pfg: as_pfg, to_pfg: dave_pfg, amount: "200.00"}
       → s1, status: "proposed"

    ⚡ POST /events/{eid}/settlements/{s1}/confirm → status: "confirmed"
    ⚡ POST /events/{eid}/settlements/{s1}/pay → status: "paid"

    ⚡ POST /events/{eid}/settlements
       Headers: Idempotency-Key: <uuid>
       {from_pfg: lisa_pfg, to_pfg: dave_pfg, amount: "100.00"}
       → s2, status: "proposed"

    ⚡ POST /events/{eid}/settlements/{s2}/confirm → status: "confirmed"
    ⚡ POST /events/{eid}/settlements/{s2}/pay → status: "paid"
```

### 5. Decomposition offer appears

After s1 (A&S → Dave, $200) is marked paid, the decomposition prompt appears because A&S is a shared PFG.

```
S12 Settle Up

    Active Settlements:
    ┌──────────────────────────┐
    │ 🔵 A&S                   │
    │ → 🔴 Dave    $200.00     │
    │ Status: 💰 Paid           │
    │                           │
    │ Split this between your   │
    │ household?                │
    │ [Decompose →]             │
    └──────────────────────────┘
    ┌──────────────────────────┐
    │ 🟡 Lisa                  │
    │ → 🔴 Dave    $100.00     │
    │ Status: 💰 Paid           │
    └──────────────────────────┘
```

Lisa's settlement has no decomposition prompt — she's a singleton PFG.

### 6. Alex decomposes into household event

```
Alex taps [Decompose →]

    Bottom sheet: "Where should this go?"
    ┌──────────────────────────┐
    │ [Add to existing event]  │  ← Alex taps this
    │ [Create from template]   │
    │ [Create new event]       │
    └──────────────────────────┘

    Event picker (filtered to ongoing + Alex's events):
    ┌──────────────────────────┐
    │ 📅 McKinnon Household    │  ← Alex taps this
    └──────────────────────────┘

    System creates:

    ⚡ POST /events/{household_eid}/transactions
       Headers: Idempotency-Key: <uuid>
       {description: "Friday Dinner with Dave & Lisa — couple's share",
        line_items: [{
          description: "Couple's dinner share",
          amount: "200.00",
          payer_person_id: alex_id_in_household,
          splits: [
            {person_id: alex_id_in_household, side: "expense", weight: 1},
            {person_id: alex_id_in_household, side: "consumption", weight: 1},
            {person_id: sam_id_in_household, side: "consumption", weight: 1}
          ]
        }]}
       → seed_tx_id

    ⚡ POST /event-links
       {source_event_id: dinner_eid,
        source_settlement_id: s1_id,
        target_event_id: household_eid,
        target_transaction_id: seed_tx_id,
        link_type: "decomposition",
        metadata: {
          source_event_name: "Friday Dinner with Dave & Lisa",
          settlement_amount: "200.00",
          settlement_currency: "AUD"
        }}
       → link created

    Toast: "Added $200.00 to McKinnon Household" [View →]
```

### 7. Household event — seed transaction visible

```
Alex taps [View →]

S05 Event Dashboard ("McKinnon Household")
    📅 Ongoing
    Current period: since 28 Feb 2026

    Recent Expenses:
    ┌──────────────────────────┐
    │ Friday Dinner with Dave  │
    │ & Lisa — couple's share  │
    │ Alex paid · $200.00      │
    │ Just now                 │
    └──────────────────────────┘

    Linked Events:
    ┌──────────────────────────┐
    │ ← Friday Dinner with     │
    │   Dave & Lisa             │
    │   Decomposition · $200   │
    │   [View source event →]  │
    └──────────────────────────┘
```

### 8. Household balances — Sam owes Alex $100

```
S11 Balances ("McKinnon Household")

    Person    Paid         Consumed     Net
    ──────    ────         ────────     ────
    Alex      $200.00      $100.00      +$100.00
    Sam       $0.00        $100.00      −$100.00
    ──────    ────         ────────     ────
    Total     $200.00      $200.00      $0.00 ✓

    Sam owes Alex $100 for their share of the dinner.
```

## Arithmetic Verification — Dinner Event

| Person | Dinner ($400 ÷ 4) | Total Consumed | Paid | Net Position |
|--------|-------------------|----------------|------|-------------|
| Alex | $100.00 | $100.00 | $0.00 | **−$100.00** |
| Sam | $100.00 | $100.00 | $0.00 | **−$100.00** |
| Dave | $100.00 | $100.00 | $400.00 | **+$300.00** |
| Lisa | $100.00 | $100.00 | $0.00 | **−$100.00** |
| **Σ** | **$400.00** | **$400.00** | **$400.00** | **$0.00 ✓** |

PFG-level settlements:
- A&S → Dave: $200.00 (Alex $100 + Sam $100 as a unit)
- Lisa → Dave: $100.00
- Total: $300.00 = Dave's net +$300.00 ✓

## Arithmetic Verification — Household Event (after decomposition)

| Person | Dinner Share ($200 ÷ 2) | Total Consumed | Paid | Net Position |
|--------|------------------------|----------------|------|-------------|
| Alex | $100.00 | $100.00 | $200.00 | **+$100.00** |
| Sam | $100.00 | $100.00 | $0.00 | **−$100.00** |
| **Σ** | **$200.00** | **$200.00** | **$200.00** | **$0.00 ✓** |

Settlement: Sam pays Alex $100.00 → checksum $0.00 ✓

## Cross-Event Traceability

```
Dinner Event                           Household Event
─────────────                          ────────────────
$400 dinner (Dave pays)                $200 seed tx (Alex pays)
  ↓                                      ↑
A&S owes $200 to Dave                 Alex consumed $100
  ↓                                   Sam consumed $100
Settlement: A&S → Dave $200              ↓
  ↓                                   Sam owes Alex $100
  └──── EVENT_LINK (decomposition) ────┘
```

The $400 dinner → $200 couple's share → $100 each in household. Every dollar is traceable across events.

## Validates

- **Shared PFG in public event** — couple presents as one settlement unit (A&S)
- **Settlement decomposition flow from S12** — "Decompose →" prompt on paid shared-PFG settlements
- **"Add to existing event" path** — filtered event picker showing ongoing events
- **EVENT_LINK creation and display** — decomposition link visible on both events
- **Seed transaction in target event** — auto-created with settlement amount and descriptive name
- **Cross-event traceability** — linked events section on S05 shows source/target connections
- **Arithmetic chain:** $400 dinner → $200 couple's share → $100 each in household
- **Singleton PFGs excluded** — Lisa's paid settlement shows no decomposition prompt
- **Ongoing event compatibility** — seed transaction lands in the current period of the household event

## Key Moment

The payoff is step 6: Alex taps "Decompose →", picks "McKinnon Household", and the system creates a $200 seed transaction in the household event. Sam now owes Alex $100 for their share of the dinner — tracked automatically, linked back to the original event. No manual entry, no mental arithmetic, no spreadsheets.

> **"Settled as a couple. Split at home. Every dollar traced."**
