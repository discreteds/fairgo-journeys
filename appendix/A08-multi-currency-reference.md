# A08 — Multi-Currency Reference

**Source:** CR-002 (Multi-Currency & Time)
**Purpose:** Reference for how Fair Go handles multiple currencies, FX conversion, cross-currency settlement, and write-offs.

---

## Principle

> One trip, three wallets, one truth. Every participant sees what they owe in their own currency, computed from a single source of truth.

---

## FX_RATE Entity

The `FX_RATE` entity stores exchange rates used for converting between currencies.

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `event_id` | UUID | Owning event |
| `from_currency` | string | Source currency code (ISO 4217, e.g. "AUD") |
| `to_currency` | string | Target currency code (always the event base currency) |
| `rate` | decimal | Conversion rate (1 unit of `from` = `rate` units of `to`) |
| `source` | enum | `"manual"` or `"api"` |
| `effective_at` | datetime | When this rate became active |
| `created_by` | UUID | User who set the rate |
| `created_at` | datetime | Record creation timestamp |

**Lifecycle:**
- Created by admin via S05 Settings or per-expense on S09
- Immutable once used by a line item (historical accuracy)
- New rates can be added; old rates remain for audit
- Multiple rates for the same currency pair can exist (different `effective_at`)

**Source types:**
| Source | Meaning | Use Case |
|--------|---------|----------|
| `manual` | Admin enters rate manually | Known rate from Wise, bank, or receipt |
| `api` | Rate fetched from external API | Auto-populated default (admin can override) |

---

## Event Base Currency

Every event has a single **base currency** set at creation:

```
POST /events {name: "Bali Holiday", currency: "IDR"}
```

- All positions (balances) are computed in the base currency
- Set at event creation on S04 (defaults to locale currency, e.g. AUD for Australian users)
- Changeable by admin in event settings (recalculates all positions)
- Choose the local currency of the destination for travel events, or the most common currency for mixed groups

---

## Per-Line-Item FX Fields

Each line item records the FX rate used at entry time:

| Field | Type | Description |
|-------|------|-------------|
| `currency` | string | Currency of the line item amount (e.g. "AUD") |
| `amount` | decimal | Amount in the line item's currency |
| `fx_rate_used` | decimal | Snapshot of the FX rate at entry time |
| `fx_source` | enum | `"manual"` or `"api"` — how the rate was obtained |
| `amount_base` | decimal | Computed: `amount × fx_rate_used` (in base currency) |

**Example:**
```
Line item: "Surf lesson" — 150 AUD
  currency: "AUD"
  amount: 150.00
  fx_rate_used: 10300        (1 AUD = 10,300 IDR)
  fx_source: "manual"
  amount_base: 1,545,000 IDR (150 × 10,300)
```

The `fx_rate_used` is a **snapshot** — if the event's FX rate is later updated, existing line items retain their original rate. This ensures positions don't shift retroactively.

---

## Currency Picker UX

### Event-level default (S04 / S05 Settings)

- Set base currency at event creation
- Add FX rates for other currencies via Settings

### Per-expense override (S09)

```
S09 Add Expense
    ┌──────────────────────────────────────────────────┐
    │  Currency:  [IDR ▼]  ← event base currency       │
    │             AUD                                   │
    │             CAD                                   │
    │             USD                                   │
    │             IDR ✓ (base)                          │
    │                                                  │
    │  Amount:    [______]                              │
    │                                                  │
    │  FX Rate:   1 AUD = 10,300 IDR  (manual)         │
    │             [Change rate]                         │
    └──────────────────────────────────────────────────┘
```

- Dropdown shows base currency first, then currencies with configured FX rates
- Selecting a non-base currency shows the FX rate and allows override
- "Change rate" opens inline editor for one-off rate adjustment (creates new FX_RATE with `source: "manual"`)

---

## Cross-Currency Settlement

When participants have different home currencies, settlement amounts are converted:

### Position calculation (in base currency)

All positions are computed in the event's base currency using `amount_base` from each line item.

### Settlement suggestion display

```
S12 Settle Up
    ┌──────────────────────────────────────────────────┐
    │  Pierre & Chloé 🟢 → Sam & Jo 🔵                │
    │  Owes: 4,250,000 IDR                             │
    │        ≈ 412.62 AUD (Sam's currency)             │
    │        ≈ 566.67 CAD (Pierre's currency)          │
    └──────────────────────────────────────────────────┘
```

The settlement suggestion shows:
1. **Base currency amount** — the canonical debt
2. **Creditor's home currency** — what the creditor expects to receive
3. **Debtor's home currency** — what the debtor needs to pay

Conversions use the event's current FX rates (not line-item snapshots).

### Settlement creation

```
POST /events/{eid}/settlements
{
  from_pfg: "pierre_chloe_pfg_id",
  to_pfg: "sam_jo_pfg_id",
  amount: 4250000,              // base currency (IDR)
  currency: "IDR",
  fx_display: {
    creditor: {currency: "AUD", amount: 412.62, rate: 10300},
    debtor:   {currency: "CAD", amount: 566.67, rate: 7500}
  }
}
```

---

## Write-Off Entity

Sub-cent rounding remainders after all settlements are complete are handled by the write-off entity:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `event_id` | UUID | Owning event |
| `amount` | decimal | Remaining balance being written off (in base currency) |
| `reason` | string | Auto-generated: "Rounding remainder" |
| `created_by` | UUID | Admin who triggered write-off |
| `created_at` | datetime | Timestamp |

**When to use:**
- After all settlements are confirmed/paid
- Remaining balance < configurable threshold (e.g. < 1 unit of base currency)
- Admin triggers via S12 "Write off remainder" button

**Endpoint:**
```
POST /events/{eid}/write-offs
{amount: 0.50, reason: "Rounding remainder after all settlements"}
→ 201 Created
→ event positions recalculate to $0.00
```

---

## Affected Endpoints

| Endpoint | Currency-Related Fields | Notes |
|----------|------------------------|-------|
| `POST /events` | `currency` (base) | Sets event base currency |
| `GET /events/{eid}` | `currency`, `fx_rates[]` | Returns base currency and all configured rates |
| `POST /events/{eid}/fx-rates` | `from_currency`, `to_currency`, `rate`, `source` | Add/update FX rate |
| `GET /events/{eid}/fx-rates` | Full FX_RATE list | All rates for the event |
| `POST /events/{eid}/transactions` | Line items: `currency`, `fx_rate_used`, `fx_source` | Per-line-item currency |
| `GET /events/{eid}/positions/persons` | `currency` (base), per-person amounts in base | Positions always in base currency |
| `GET /events/{eid}/positions/pfgs` | Same as above, rolled up by PFG | PFG-level positions |
| `GET /events/{eid}/settlement-suggestions` | `amount` (base), `fx_display` (creditor/debtor) | Cross-currency display amounts |
| `POST /events/{eid}/settlements` | `amount`, `currency`, `fx_display` | Settlement in base currency with display conversions |
| `POST /events/{eid}/write-offs` | `amount` (base) | Write off rounding remainder |

---

## Cross-References

- **S04 Create Event** — base currency selection
- **S09 Add Expense** — per-expense currency picker and FX rate display
- **S11 Balances** — multi-currency balance display
- **S12 Settle Up** — cross-currency settlement suggestions and write-offs
- **SC14 Multi-Currency Bali Holiday** — end-to-end multi-currency scenario
- **A01 Orchestrations** — FX rate management orchestrations
