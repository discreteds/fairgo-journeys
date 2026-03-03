# R03 — Settlement Rail

**Purpose:** View balances, create settlements, confirm, and mark paid.
**Primary persona:** All members (create), admin (confirm), payer (mark paid).

## Rail Path

```
S11 Balances (or S05 "Settle Up" button)
  │
  ▼
S12 Settle Up → Create settlements from suggestions
  │
  ▼ (status: proposed)
S12 Settle Up → Admin confirms
  │
  ▼ (status: confirmed)
S12 Settle Up → Payer marks paid
  │
  ▼ (status: paid)
S11 Balances → $0.00 ✓ (all settled)
  │
  ▼
S05 Event Dashboard → "All settled ✓" badge
```

## Status Flow

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

## Settlement Guard: Closed Events (JF-1B)

If the event has been **closed**, the "Settle Up" action is blocked. Users cannot create new settlements on a closed event.

```
S05 / S11
  │
  ├── "Settle Up" tapped
  │     │
  │     ├── [event open?] → S12 (normal flow)
  │     │
  │     └── [event closed?] → BLOCKED
  │           "This event is closed. Reopen it to create new settlements."
  │           [Reopen Event] (admin only)
  │
  ▼
S12 → [event closed?] → blocked
```

- Existing settlements (proposed/confirmed) remain visible but cannot be modified
- Paid settlements are unaffected (already final)
- Admin must reopen the event (`POST /events/{eid}/reopen`) before creating new settlements
- The "Settle Up" button appears disabled with a tooltip on closed events

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S05 → S12 | User taps "Settle Up" (owes money, event open) | Direct to S12 |
| S11 → S12 | User taps "Settle Up" on balances (event open) | Direct to S12 |
| S12 → S11 | All settlements paid | Back to balances (all clear) |

## Key Orchestration Sequence

```
POST /events/{eid}/settlements                    # Create (proposed)
POST /events/{eid}/settlements/{sid}/confirm       # Admin confirms
POST /events/{eid}/settlements/{sid}/pay           # Payer marks paid
```

Three calls per settlement, but each is a single tap on S12.

## Scenarios Using This Rail

- SC03 (Family Holiday) — shared settlement group pays as one unit
- SC04 (Housemates Monthly Bills) — solo settlement groups, multiple settlements
- SC05 (Settle and Close) — full lifecycle through to event closure
