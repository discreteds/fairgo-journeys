# R09 — Decomposition Rail

**Purpose:** After a shared-PFG settlement is paid, decompose it into a target event (typically a household ongoing event).
**Primary persona:** Couple or household member who just settled a shared-PFG expense at a public event.

## Rail Path

```
S12 Settle Up (settlement paid, shared PFG)
  │
  ├──── "Decompose →" on paid settlement
  │     │
  │     ▼
  │   Decompose Modal
  │     │
  │     ├── [Add to existing event] → event picker → target event
  │     ├── [Create from template] → template picker → new event
  │     └── [Create new event] → S04 (amount pre-filled) → new event
  │           │
  │           ▼
  │     ⚡ Create seed transaction in target event
  │     ⚡ Create EVENT_LINK (decomposition type)
  │           │
  │           ▼
  │     Confirmation: "Added $X to {event name}"
  │     [View →] → S05 Target Event Dashboard
  │                 (seed tx visible, linked events shown)
  │
  ├── Alternative entry: S11 Balances
  │   "Decompose →" on shared-PFG position card
  │   → S12 (decomposition context) → same flow above
  │
  └── Alternative entry: S05 Event Dashboard
      Post-settlement decomposition prompt
      → same decomposition modal → same flow above
```

## Entry Points

The decomposition rail can be entered from three screens, all for the same purpose:

| Entry | Screen | Context |
|-------|--------|---------|
| Primary | S12 Settle Up | Paid settlement card shows "Decompose →" — most contextual |
| Secondary | S11 Balances | Shared-PFG position card shows "Decompose →" |
| Tertiary | S05 Dashboard | Post-settlement prompt when all settlements paid |

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S12 → Decompose modal | Paid settlement on shared PFG | Target selection |
| S11 → S12 → Decompose | Shared PFG with paid settlements | R03 then R09 |
| S05 → Decompose modal | All settlements paid, shared PFG involved | Target selection |
| Decompose modal → S05 | Target event selected/created, seed tx + link created | Target event dashboard |

## Key Orchestration Sequence

```
POST /events/{target_eid}/transactions          # 1. Seed transaction in target event
  {description, line_items: [{amount, payer, splits}]}
POST /event-links                                # 2. Link source settlement to target tx
  {source_event_id, source_settlement_id,
   target_event_id, target_transaction_id,
   link_type: "decomposition", metadata: {...}}
```

Two API calls. The seed transaction amount equals the settlement amount. The link creates a traceable connection between the public event's settlement and the household event's transaction.

## Scenarios Using This Rail

- SC25 (Dinner Decomposition Pipeline) — friends' dinner → shared-PFG settlement → decompose into household ongoing event
