# S12 — Settle Up

**Purpose:** Create, confirm, and mark settlements as paid.
**Visible to:** All can create/view. Admin confirms. Payer marks paid.
**Rails:** R03 (Settlement)
**Scenarios:** SC03, SC04, SC05, SC13, SC17, SC18

> **Idempotency requirement (CR-001):** All settlement mutation endpoints (`POST` create, confirm, pay, void) require an `Idempotency-Key: <client-generated-uuid>` header. The client generates a UUID v4 before each mutation and includes it in the request. If the same key is sent twice, the server returns the original response without re-executing the operation. This prevents duplicate settlements from network retries or double-taps.

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

Admin can void proposed or confirmed settlements. Voided settlements display who voided them and when (JF-1B, JF-5):

```
|  | Alice & Partner             ||
|  | -> Frank  $142.50           ||
|  | Status: ⏳ Proposed          ||
|  | [Confirm]  [Void]           || <- admin only
```

```
│  Voided Settlements            │
│  ┌──────────────────────────┐│
│  │ 🔴 Dave                  ││
│  │ → 🔴 Bob    $85.00      ││
│  │ Status: ❌ Voided         ││
│  │ Voided by: Alice          ││
│  │ Voided at: 15 Jan 2:45pm ││
│  └──────────────────────────┘│
```

Voided settlement response includes `voided_at` (ISO 8601 timestamp) and `voided_by` (person display name + user ID).

## Settlement Status Flow

```
proposed -----> confirmed -----> paid
    |               |
    +--- voided <---+

[Confirm]       [Mark Paid]     (done)
admin only      payer or admin

[Void]          [Void]
admin only      admin only
```

Voided settlements are preserved for audit but excluded from positions and settlement suggestions.

## Orchestration — Page Load

```
1. GET /events/{eid}/positions                  → PFG positions (balances)
2. GET /events/{eid}/settlements                → existing settlements
3. GET /events/{eid}/settlement-suggestions     → server-computed optimal payment plan
```

Suggested settlements are computed **server-side** via `GET /events/{eid}/settlement-suggestions` using a debt simplification algorithm (minimise number of payments). The UI fetches suggestions from the server rather than computing them locally — this ensures consistency and allows the algorithm to account for voided settlements, cross-currency conversions, and write-off thresholds.

## Orchestration — "Create Settlement"

```
POST /events/{eid}/settlements
  Headers: Idempotency-Key: <client-generated-uuid>
  {from_group_id, to_group_id, amount, currency,
   source_currency, target_currency, fx_rate_used}
→ settlement created (status: proposed)
```

When the event involves multiple currencies (CR-002), the settlement includes `source_currency` (payer's currency), `target_currency` (payee's currency), and `fx_rate_used` (rate at time of creation). The UI displays both original and converted amounts:

```
│  │ 🔵 Alice & Partner       ││
│  │     pays A$142.50         ││
│  │     (€89.12 @ 0.6254)    ││
│  │     → 🔴 Frank           ││
│  │  [Create Settlement]     ││
```

## Orchestration — "Confirm" (Admin)

```
POST /events/{eid}/settlements/{sid}/confirm
  Headers: Idempotency-Key: <client-generated-uuid>
→ status: confirmed
```

## Orchestration — "Mark as Paid"

```
POST /events/{eid}/settlements/{sid}/pay
  Headers: Idempotency-Key: <client-generated-uuid>
→ status: paid
```

## Orchestration — "Void" (Admin)

```
POST /events/{eid}/settlements/{sid}/void
  Headers: Idempotency-Key: <client-generated-uuid>
→ status: voided
→ response includes voided_at, voided_by
→ positions recalculate (voided settlement excluded)
```

Only proposed and confirmed settlements can be voided. Paid settlements cannot be voided.

## Smart Defaults

- Suggested settlements fetched from server (`GET /events/{eid}/settlement-suggestions`) — one tap to create
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

## Settlement Guard — Closed Events (JF-1B)

When an event is closed, new settlements cannot be created. The UI disables creation controls and displays a message:

```
┌──────────────────────────────┐
│  ← Settle Up                 │
├──────────────────────────────┤
│                              │
│  ⓘ This event is closed.    │
│  No new settlements can be   │
│  created.                    │
│                              │
│  Existing Settlements        │
│  ┌──────────────────────────┐│
│  │ (settlements still       ││
│  │  visible in read-only)   ││
│  └──────────────────────────┘│
│                              │
│  [Create Settlement]         │ ← disabled / greyed out
│  [+ Custom Settlement]       │ ← disabled / greyed out
└──────────────────────────────┘
```

The backend enforces this — `POST /events/{eid}/settlements` returns `409 Conflict` with `"Event is closed"` if the event status is `closed`. Existing settlements in proposed/confirmed states remain actionable (confirm, pay, void) even after closure.

## Write-Off for Rounding Remainders (CR-002)

When settlement suggestions include amounts below a configurable threshold (e.g. $0.05), the server may suggest a **write-off** instead of a settlement. Write-offs zero out trivial balances without requiring a payment.

```
│  │ 🔴 Dave                  ││
│  │     owes $0.03            ││
│  │     → 🔴 Frank           ││
│  │  [Write Off]              ││
```

Write-offs are created via `POST /events/{eid}/write-offs` and recorded in the audit trail. The write-off threshold is configurable per event.

## Error States

| Error | Display |
|-------|---------|
| Settlement blocked (unfunded) | "Event must be funded before settlements can be created" → S14 |
| Settlement blocked (event closed) | "This event is closed. No new settlements can be created." |
| Duplicate settlement | "A settlement between these groups already exists" |
| Duplicate request (idempotency) | Server returns original response (transparent to user) |
| Invalid amount | "Amount must be greater than zero" |
| Confirm non-proposed | Backend enforces status flow — button hidden if not applicable |
