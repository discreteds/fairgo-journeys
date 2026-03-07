# S11 — Balances

**Purpose:** Who owes whom. The "positions" view, simplified.
**Visible to:** All event members.
**Rails:** R02, R03, R09
**Scenarios:** SC03, SC04, SC05, SC11, SC13, SC17, SC18, SC25

## Wireframe — By Settlement Group (Default)

```
┌──────────────────────────────┐
│  ← Balances · Bali Trip      │
├──────────────────────────────┤
│                              │
│  ┌──────────────────────────┐│
│  │  You owe $142.50         ││
│  │  ┌────────────────────┐  ││
│  │  │     Settle Up      │  ││
│  │  └────────────────────┘  ││
│  └──────────────────────────┘│
│                              │
│  By Settlement Group         │
│  ┌──────────────────────────┐│
│  │ 🔵 Alice & Partner       ││
│  │    owes $142.50          ││
│  │    ├ Alice    -$37.50    ││
│  │    └ Carol    -$105.00   ││
│  ├──────────────────────────┤│
│  │ 🔴 Bob                   ││
│  │    is owed $85.00        ││
│  ├──────────────────────────┤│
│  │ 🔴 Dave                  ││
│  │    owes $95.00           ││
│  ├──────────────────────────┤│
│  │ 🔴 Eve                   ││
│  │    owes $80.00           ││
│  ├──────────────────────────┤│
│  │ 🔴 Frank                 ││
│  │    is owed $132.50       ││
│  └──────────────────────────┘│
│                              │
│  Checksum: $0.00 ✓           │
│                              │
│  View: [By Group] [By Person]│
└──────────────────────────────┘
```

## Wireframe — By Person View

```
│  By Person                   │
│  ┌──────────────────────────┐│
│  │ Alice       -$37.50      ││
│  │ Bob         +$85.00      ││
│  │ Carol      -$105.00      ││
│  │ Dave        -$95.00      ││
│  │ Eve         -$80.00      ││
│  │ Frank      +$132.50      ││
│  └──────────────────────────┘│
│                              │
│  Checksum: $0.00 ✓           │
```

## Wireframe — Multi-Currency Variant

When an event contains expenses in multiple currencies, balances are displayed per currency:

```
┌──────────────────────────────┐
│  ← Balances · Bali Trip      │
├──────────────────────────────┤
│                              │
│  ┌──────────────────────────┐│
│  │  AUD  You owe $142.50    ││
│  │  IDR  You owe Rp 500,000 ││
│  │  ┌────────────────────┐  ││
│  │  │     Settle Up      │  ││
│  │  │  (select currency) │  ││
│  │  └────────────────────┘  ││
│  └──────────────────────────┘│
│                              │
│  AUD Balances                │
│  ┌──────────────────────────┐│
│  │ 🔵 Alice & Partner       ││
│  │    owes $142.50          ││
│  │    ├ Alice    -$37.50    ││
│  │    └ Carol    -$105.00   ││
│  ├──────────────────────────┤│
│  │ 🔴 Bob                   ││
│  │    is owed $85.00        ││
│  ├──────────────────────────┤│
│  │ 🔴 Frank                 ││
│  │    is owed $57.50        ││
│  └──────────────────────────┘│
│  Checksum: $0.00 ✓           │
│                              │
│  IDR Balances                │
│  ┌──────────────────────────┐│
│  │ 🔵 Alice & Partner       ││
│  │    owes Rp 500,000       ││
│  ├──────────────────────────┤│
│  │ 🔴 Dave                  ││
│  │    is owed Rp 500,000    ││
│  └──────────────────────────┘│
│  Checksum: Rp 0 ✓            │
│                              │
│  View: [By Group] [By Person]│
└──────────────────────────────┘
```

Balances are grouped by currency. Each currency section has its own independent checksum. The summary card at top shows the user's net position per currency.

## Wireframe — Shared PFG Decompose Action (CR-003)

When a settlement group is a shared PFG (non-singleton, e.g. a couple), the position card shows a "Decompose" action. This allows the group members to internally split their shared obligation into another event.

```
│  By Settlement Group         │
│  ┌──────────────────────────┐│
│  │ 🔵 Alex & Sam            ││
│  │    owes $200.00           ││
│  │    ├ Alex    -$100.00    ││
│  │    └ Sam     -$100.00    ││
│  │                           ││
│  │    [Decompose →]          ││
│  ├──────────────────────────┤│
│  │ 🔴 Dave                  ││
│  │    is owed $200.00        ││
│  └──────────────────────────┘│
```

The "Decompose →" action is only visible when:
- The settlement group is a shared PFG (2+ members)
- The current user is a member of that PFG
- At least one paid settlement exists for this PFG

Tapping "Decompose →" navigates to S12 with the decomposition flow pre-loaded (R09 rail). If no paid settlements exist yet, the action label changes to "Settle first to decompose" (disabled).

## Orchestration — Page Load

```
GET /events/{eid}/positions
→ returns:
  - PFG-level positions (grouped by settlement group), per currency
  - Per-person breakdown within each PFG, per currency
  - Event-level checksum per currency
```

Single API call. The positions endpoint returns everything needed for both views. In multi-currency events, positions are returned per currency — each currency has its own independent set of balances and checksum.

### Effectively Settled State (CR-015)

The positions response includes an `effectively_settled` boolean. When `true`, all outstanding balances are within rounding tolerance (1 cent per participant) — the event is "close enough" to fully settled. This is **server-authoritative** — the frontend must not apply its own tolerance calculation.

```
│  ┌──────────────────────────────┐│
│  │  🎉 All Settled!              ││
│  │  All balances are within      ││
│  │  rounding tolerance.          ││
│  └──────────────────────────────┘│
```

Display the "All Settled!" variant when `effectively_settled: true`, even if tiny cent-level remainders exist in the position data.

## Actions

| Action | Who | Target | Orchestration |
|--------|-----|--------|--------------|
| Settle Up | All (when owing) | → S12 | Pre-filled with user's PFG as from_group |
| Settlement group tap | All | → S12 | Pre-filled with tapped group |
| Toggle By Group / By Person | All | Same screen | Client-side re-render, no API call |
| Decompose → | Shared PFG member | → S12 (decompose) | R09 — navigate with decomposition context |

## Cross-Currency Settlements

When an event spans multiple currencies, settlement suggestions on S12 may involve cross-currency conversions:

- The "Settle Up" button on S11 navigates to S12 where settlement suggestions are computed per currency
- If a user owes in multiple currencies, they can settle each currency separately or settle in a single currency using the recorded FX rates
- The UI shows the **original currency** of each balance alongside the **equivalent settlement amount** when cross-currency conversion is applied (e.g. "IDR 500,000 ≈ AUD 50.00")
- FX rates are the rates recorded at the time each expense was created (not live rates)

## Smart Defaults

- Default view is "By Settlement Group" — this is what matters for settlement
- "By Person" is a toggle for detail, same data reorganised
- **"You owe $X"** card shows current user's PFG net position:
  - Negative = owes money (red, with "Settle Up" CTA)
  - Positive = owed money (green, "You're owed $X")
  - Zero = all settled (grey, "All settled ✓")
- Shared settlement groups show individual member contributions as sub-lines
- Checksum displayed as a verification badge — always $0.00 if data is correct
- Amounts formatted with + (owed) and - (owes) signs for clarity
- **Multi-currency events:** Balances are displayed per currency. Each currency section has its own checksum. The summary card shows the user's net position in each currency.
- **Shared PFG decompose action:** "Decompose →" shown on non-singleton settlement groups where the user is a member and paid settlements exist. Disabled with hint text if no paid settlements yet.

## Error States

| Error | Display |
|-------|---------|
| No transactions yet | "No expenses recorded yet. Add one to see balances." |
| Checksum non-zero | "⚠️ Balance error detected. Contact support." (should never happen) |
