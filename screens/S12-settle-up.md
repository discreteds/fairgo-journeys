# S12 — Settle Up

**Purpose:** Create, confirm, and mark settlements as paid.
**Visible to:** All can create/view. Admin confirms. Payer marks paid.
**Rails:** R03 (Settlement)
**Scenarios:** SC03, SC04, SC05

## Wireframe — Suggested Settlements

```
┌──────────────────────────────┐
│  ← Settle Up                 │
├──────────────────────────────┤
│                              │
│  Suggested Settlements       │
│  ┌──────────────────────────┐│
│  │ 🔵 Alice & Partner       ││
│  │     pays $142.50         ││
│  │     → 🔴 Frank           ││
│  │  [Create Settlement]     ││
│  ├──────────────────────────┤│
│  │ 🔴 Dave                  ││
│  │     pays $95.00          ││
│  │     → 🔴 Bob  $85.00    ││
│  │     → 🔴 Frank $10.00   ││
│  │  [Create Settlement]     ││
│  ├──────────────────────────┤│
│  │ 🔴 Eve                   ││
│  │     pays $80.00          ││
│  │     → 🔴 Frank           ││
│  │  [Create Settlement]     ││
│  └──────────────────────────┘│
│                              │
│  Active Settlements          │
│  ┌──────────────────────────┐│
│  │ (none yet)               ││
│  └──────────────────────────┘│
└──────────────────────────────┘
```

## Wireframe — Active Settlements (Various States)

```
│  Active Settlements          │
│  ┌──────────────────────────┐│
│  │ 🔵 Alice & Partner       ││
│  │ → 🔴 Frank  $142.50     ││
│  │ Status: ⏳ Proposed       ││
│  │ [Confirm]                ││ ← admin only
│  ├──────────────────────────┤│
│  │ 🔴 Dave                  ││
│  │ → 🔴 Bob    $85.00      ││
│  │ Status: ✅ Confirmed      ││
│  │ [Mark as Paid]           ││ ← payer or admin
│  ├──────────────────────────┤│
│  │ 🔴 Eve                   ││
│  │ → 🔴 Frank  $80.00      ││
│  │ Status: 💰 Paid          ││
│  └──────────────────────────┘│
```

## Settlement Status Flow

```
proposed ──────→ confirmed ──────→ paid
    │                │               │
 [Confirm]      [Mark Paid]      (done)
 admin only     payer or admin
```

## Orchestration — Page Load

```
1. GET /events/{eid}/positions     → PFG positions (for suggested settlements)
2. GET /events/{eid}/settlements   → existing settlements
```

Suggested settlements are computed **client-side** from positions data using a debt simplification algorithm (minimise number of payments).

## Orchestration — "Create Settlement"

```
POST /events/{eid}/settlements
  {from_group_id, to_group_id, amount, currency}
→ settlement created (status: proposed)
```

## Orchestration — "Confirm" (Admin)

```
POST /events/{eid}/settlements/{sid}/confirm
→ status: confirmed
```

## Orchestration — "Mark as Paid"

```
POST /events/{eid}/settlements/{sid}/pay
→ status: paid
```

## Smart Defaults

- Suggested settlements pre-computed from positions — one tap to create
- Debt simplification minimises number of payments (e.g. instead of A→B and A→C, compute net A→C if B→C exists)
- From/to groups and amounts are pre-filled — user just confirms
- Settlement flow is linear and clear: create → confirm → paid
- "Confirm" only appears for admins (prevents premature confirmation)
- "Mark as Paid" available to payer's settlement group members or admin
- Once all settlements are paid, S11 shows "All settled ✓"

## Custom Settlement

If the user wants to create a settlement not in the suggestions:

```
│  [+ Custom Settlement]       │
│  ┌──────────────────────────┐│
│  │ From: [Select group ▾]   ││
│  │ To:   [Select group ▾]   ││
│  │ Amount: [$_________]     ││
│  │ [Create]                 ││
│  └──────────────────────────┘│
```

## Error States

| Error | Display |
|-------|---------|
| Settlement blocked (unfunded) | "Event must be funded before settlements can be created" → S14 |
| Duplicate settlement | "A settlement between these groups already exists" |
| Invalid amount | "Amount must be greater than zero" |
| Confirm non-proposed | Backend enforces status flow — button hidden if not applicable |
