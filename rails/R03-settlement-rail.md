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
proposed ──────→ confirmed ──────→ paid
    │                │               │
 [Confirm]      [Mark Paid]      (done)
 admin only     payer or admin
```

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S05 → S12 | User taps "Settle Up" (owes money) | Direct to S12 |
| S11 → S12 | User taps "Settle Up" on balances | Direct to S12 |
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
