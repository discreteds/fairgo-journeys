# S12 — Settle Up

**Purpose:** Create, confirm, and mark settlements as paid. Offers decomposition for shared-PFG settlements.
**Visible to:** All can create/view. Admin confirms. Payer marks paid.
**Rails:** R03 (Settlement), R09 (Decomposition)
**Scenarios:** SC03, SC04, SC05, SC13, SC17, SC18, SC24, SC25

> **Idempotency (CR-001, CR-015):** Resource-creation endpoints (`POST` create) require an `Idempotency-Key: <client-generated-uuid>` header. State-transition endpoints (confirm, pay, void) do NOT require Idempotency-Key — they are naturally idempotent.

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

## Wireframe — Partially Paid Settlement (CR-021)

Settlement card in `partially_paid` state with progress tracking and payment history.

```
┌─────────────────────────────────┐
│  ← Settle Up                   │
├─────────────────────────────────┤
│                                 │
│  🔵 Johnson Family → 🔴 You    │
│  $200.00 AUD                   │
│                                 │
│  ████████░░░░░░░░  $120 / $200 │
│  Partially Paid                 │
│                                 │
│  Payments:                      │
│  • $80.00  — 2 Mar 2026        │
│  • $40.00  — 5 Mar 2026        │
│                                 │
│  ┌─────────────────────────┐   │
│  │    Pay Remaining $80    │   │
│  └─────────────────────────┘   │
│  ┌─────────────────────────┐   │
│  │     Pay Other Amount    │   │
│  └─────────────────────────┘   │
│                                 │
└─────────────────────────────────┘
```

## Wireframe — Ongoing Event Period Indicator

For ongoing events (`event_type: ongoing`), the settle-up screen shows a period indicator above the settlements. This scopes the current settlement to transactions since the last bookmark.

```
┌──────────────────────────────┐
│  ← Settle Up                 │
├──────────────────────────────┤
│  📅 Current Period            │
│  Since last settlement:      │
│  31 Jan 2026                 │
│  8 new transactions          │
│                              │
│  [Settle Current Period]     │
├──────────────────────────────┤
│                              │
│  Suggested Settlements       │
│  (scoped to current period)  │
│  ...                         │
└──────────────────────────────┘
```

When no previous settlement exists (first period), the indicator shows "Since event created" with the event creation date. The `settled_through_date` on proposed settlements marks the bookmark cutoff for the current period.

## Wireframe — Decomposition Offer (Paid Shared-PFG Settlement)

When a settlement involving a shared PFG (non-singleton) is marked as paid, a decomposition prompt appears. This lets the couple/household split their shared obligation internally.

```
│  Active Settlements          │
│  ┌──────────────────────────┐│
│  │ 🔵 Alex & Sam            ││
│  │ → 🔴 Dave    $200.00     ││
│  │ Status: 💰 Paid           ││
│  │                           ││
│  │ Split this between your   ││
│  │ household?                ││
│  │ [Decompose →]             ││
│  └──────────────────────────┘│
```

The "Decompose →" prompt only appears when:
- The settlement is paid
- The from_group or to_group is a shared PFG (non-singleton)
- The current user is a member of that shared PFG
- No existing EVENT_LINK references this settlement

## Orchestration — Decomposition Flow

```
1. User taps "Decompose →" on a paid settlement
2. Bottom sheet / modal: "Where should this go?"
   a) [Add to existing event] → event picker (filtered to ongoing + user's events)
   b) [Create from template] → template picker → POST /templates/{id}/instantiate
   c) [Create new event] → S04 with amount pre-filled
3. System creates:
   a) Transaction in target event:
      POST /events/{target_eid}/transactions
        {description: "{source_event_name} — couple's share",
         line_items: [{amount: settlement_amount,
           payer_person_id: payer_in_target_event,
           splits: [target event members]}]}
   b) Event link:
      POST /event-links
        {source_event_id, source_settlement_id,
         target_event_id, target_transaction_id,
         link_type: "decomposition",
         metadata: {source_event_name, settlement_amount, settlement_currency}}
4. Confirmation toast: "Added $200.00 to McKinnon Household" [View →]
   → [View →] navigates to S05 of target event
```

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

### Partial Payment Path (CR-021)

Settlements now support partial payments. The updated status flow:

```
proposed ──confirm──▶ confirmed ──pay──▶ paid
                         │                 ▲
                         │     pay(partial) │
                         ▼                 │
                    partially_paid ─pay──▶─┘
    (any non-paid) ──void──▶ voided
```

A settlement enters `partially_paid` when a payment is recorded for less than the remaining balance. The `paid_total` and `remaining` fields track cumulative progress. Subsequent payments can bring it to `paid`.

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
   period_label,
   source_currency, target_currency, fx_rate_used}
→ settlement created (status: proposed)
```

`period_label` (optional string, CR-015): Free-text label for the settlement period (e.g., "March 2026"). Useful for ongoing events to identify which billing cycle a settlement covers.

> **Permission change (CR-021):** Members can now create settlements where `from_group_id` is their own PFG. Admin role is still required for creating settlements on behalf of others. The "Propose Settlement" button appears for any member who has an outstanding balance, not just admins.

### Over-Settlement Guard (CR-015)

The server rejects settlement amounts exceeding 101% of the outstanding balance between two PFGs. This prevents accidental over-payment while allowing a small tolerance for rounding.

```
POST /events/{eid}/settlements {amount: "<over 101% of outstanding>"}
→ 422 Unprocessable Entity
  {code: "OVER_SETTLEMENT", outstanding: "142.50", attempted: "200.00"}
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
→ status: confirmed
```

## Orchestration — "Mark as Paid"

```
POST /events/{eid}/settlements/{sid}/pay
→ status: paid
```

## Orchestration — "Pay Partial Amount" (CR-021)

When the user taps "Pay Other Amount" on a confirmed or partially_paid settlement:

```
1. UI shows amount input (pre-filled with remaining balance)
2. POST /events/{eid}/settlements/{sid}/pay
   Headers: Idempotency-Key: <uuid>
   Body: { "amount": "40.00" }
   → 200: settlement with status "partially_paid", updated paid_total and remaining
3. If amount >= remaining → status becomes "paid" (full settlement)
4. If amount > remaining → 422 VALIDATION_ERROR
```

"Pay Remaining" is a convenience shortcut that calls the same endpoint without the `amount` field (or with `amount` equal to `remaining`), which completes the settlement to `paid` status.

## Orchestration — "Void" (Admin)

```
POST /events/{eid}/settlements/{sid}/void
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
- **Ongoing events:** Period indicator shown above settlements, scoped to transactions since last bookmark
- **Ongoing events:** "Settle Current Period" replaces generic "Create Settlement" label — clarifies scope
- **Ongoing events:** `settled_through_date` set on proposed settlements, displayed as "Covers through {date}"
- **Decomposition offer:** Only shown on paid settlements where from/to group is shared PFG and user is a member
- **Decomposition:** "Add to existing event" filters to ongoing events the user belongs to — most common path
- **Decomposition:** Seed transaction description auto-generated from source event name

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
| Over-settlement (CR-015) | "Settlement amount exceeds outstanding balance" — 422 `OVER_SETTLEMENT` with `outstanding` and `attempted` amounts |
