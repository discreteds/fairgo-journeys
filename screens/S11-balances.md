# S11 — Balances

**Purpose:** Who owes whom. The "positions" view, simplified.
**Visible to:** All event members.
**Rails:** R02, R03
**Scenarios:** SC03, SC04, SC05

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

## Orchestration — Page Load

```
GET /events/{eid}/positions
→ returns:
  - PFG-level positions (grouped by settlement group)
  - Per-person breakdown within each PFG
  - Event-level checksum
```

Single API call. The positions endpoint returns everything needed for both views.

## Actions

| Action | Who | Target | Orchestration |
|--------|-----|--------|--------------|
| Settle Up | All (when owing) | → S12 | Pre-filled with user's PFG as from_group |
| Settlement group tap | All | → S12 | Pre-filled with tapped group |
| Toggle By Group / By Person | All | Same screen | Client-side re-render, no API call |

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

## Error States

| Error | Display |
|-------|---------|
| No transactions yet | "No expenses recorded yet. Add one to see balances." |
| Checksum non-zero | "⚠️ Balance error detected. Contact support." (should never happen) |
