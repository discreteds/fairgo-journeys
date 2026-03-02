# S14 — Event Funding

**Purpose:** Fund an event to unlock full features. Manage cost spread.
**Visible to:** Admin only.
**Rails:** R05 (Membership & Funding), R06 (Admin)
**Scenarios:** SC06

## Wireframe — Unfunded Event

```
┌──────────────────────────────┐
│  ← Funding · Bali Trip       │
├──────────────────────────────┤
│                              │
│  Status: ⚠️ Unfunded         │
│                              │
│  Current Limits              │
│  ┌──────────────────────────┐│
│  │ 1 transaction             ││
│  │ 5 line items              ││
│  │ 1 group                   ││
│  │ 5 participants            ││
│  │ No settlements            ││
│  └──────────────────────────┘│
│                              │
│  Fund This Event             │
│  ┌──────────────────────────┐│
│  │ ○ Use subscription slot  ││
│  │   3 one-off slots left   ││
│  │   1 ongoing slot left    ││
│  │                          ││
│  │ ● Pay $2 one-time     ││
│  │   No subscription needed ││
│  │                          ││
│  │  [Fund Event]            ││
│  └──────────────────────────┘│
└──────────────────────────────┘
```

## Wireframe — Funded Event

```
┌──────────────────────────────┐
│  ← Funding · Bali Trip       │
├──────────────────────────────┤
│                              │
│  Status: ✅ Funded            │
│  Method: Per-event fee       │
│                              │
│  Limits: Full access         │
│  50 transactions, 500 line   │
│  items, 20 groups, 50        │
│  participants, settlements   │
│  enabled                     │
│                              │
│  Cost Spread                 │
│  ┌──────────────────────────┐│
│  │ Spread the $2 fee     ││
│  │ across all 6 participants ││
│  │ ($0.83 each)?            ││
│  │                          ││
│  │ This adds a line item    ││
│  │ to the event so everyone ││
│  │ shares the cost.         ││
│  │                          ││
│  │ [Spread Costs]  [Skip]   ││
│  └──────────────────────────┘│
│                              │
│  ── after spreading ──       │
│                              │
│  Cost Spread: ✅ Active       │
│  $2 split across 6 people │
│  ($0.83 per person)          │
└──────────────────────────────┘
```

## Wireframe — Subscription Slot Options

When "Use subscription slot" is selected:

```
│  Slot Type                   │
│  ┌──────────────────────────┐│
│  │ ● One-off slot           ││
│  │   Permanently consumed.  ││
│  │   3 remaining.           ││
│  │                          ││
│  │ ○ Ongoing slot           ││
│  │   Released when event    ││
│  │   closes. 1 remaining.  ││
│  └──────────────────────────┘│
```

## Orchestration — "Fund Event" (Per-Event Fee)

```
POST /events/{eid}/funding
  {funding_type: "per_event_fee"}
→ event funded, limits lifted to full access
```

## Orchestration — "Fund Event" (Subscription Slot)

```
POST /events/{eid}/funding
  {funding_type: "subscription_slot", slot_type: "one_off"}
→ slot consumed from user's membership quota, event funded
```

## Orchestration — "Spread Costs"

```
POST /events/{eid}/funding/cost-spread
  {spread_to: "all_participants"}
→ line item created in a system transaction, spreading $2
  across all event participants as consumption splits
```

## Orchestration — Page Load

```
GET /events/{eid}/funding
→ returns: funding_status, funding_type, cost_spread status
```

If unfunded, also needs membership quota info:

```
GET /memberships/me
→ returns: available slots for the subscription option display
```

## Smart Defaults

- Per-event fee pre-selected if user has no subscription slots available
- Subscription slot pre-selected if user has available slots
- Cost spread shown immediately after funding — natural next step
- Cost spread amount capped at per-event fee ($2) to prevent profiteering
- Slot type explanation is inline — users don't need to understand the quota model in depth
- Ongoing vs one-off guidance: "Use ongoing for events that will last a while (trips). Use one-off for single-day events (dinners)."

## Error States

| Error | Display |
|-------|---------|
| No slots available | "No subscription slots available. Use per-event fee or upgrade your plan." → link to S13 |
| Already funded | "This event is already funded" (show funded state) |
| Cost spread already exists | "Cost spread already active" (prevent duplicate — known backend gap R9) |
| Event locked | "This event is locked and can no longer be modified" |
