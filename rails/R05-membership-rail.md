# R05 — Membership & Funding Rail

**Purpose:** Manage subscription tier and fund events.
**Primary persona:** Admin (funding), any user (membership).

## Rail Path

```
S15 Profile
  │ "Membership ▸"
  ▼
S13 Membership → view tier / quotas
  │ [Upgrade] or [Change plan]
  ▼
S13 Membership → tier changed, quotas updated
  │
  │ ······ (user returns to event) ······
  │
  ▼
S05 Event Dashboard
  │ admin sees "⚠️ Unfunded" or "💳 Funding Status ▸"
  ▼
S14 Event Funding
  │ [Fund Event] (slot or $4.99)
  ▼
S14 Event Funding → funded
  │ [Spread Costs] (optional)
  ▼
S05 Event Dashboard → limits lifted, full access
```

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S14 → S13 | No subscription slots, user wants to upgrade | Detour to membership |
| S13 → S14 | After upgrading, return to fund event | Back to funding |
| S09 → S14 | Expense blocked by funding gate | Error link to funding |
| S14 → S05 | Event funded | Return to dashboard |

## Key Orchestration Sequence

Upgrading + funding an event:

```
POST /memberships {tier: "standard", billing_cycle: "monthly"}
POST /events/{eid}/funding {funding_type: "subscription_slot", slot_type: "one_off"}
POST /events/{eid}/funding/cost-spread   # optional
```

Or the simpler per-event fee path:

```
POST /events/{eid}/funding {funding_type: "per_event_fee"}
POST /events/{eid}/funding/cost-spread   # optional
```

## Scenarios Using This Rail

- SC06 (Free User Hits Limits) — funding gate → per-event fee → cost spread
- SC06 variant — funding gate → upgrade membership → subscription slot
