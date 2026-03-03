# S14 — Event Funding

**Purpose:** Fund an event to unlock full features. Manage cost spread.
**Visible to:** Admin only.
**Rails:** R05 (Membership & Funding), R06 (Admin)
**Scenarios:** SC06, SC10, SC28

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

## Wireframe — Membership Suggestion

When the user has multiple memberships (solo + shared), the funding screen shows a ranked suggestion list before the slot type selection.

```
┌──────────────────────────────┐
│  ← Funding · Weekend BBQ     │
├──────────────────────────────┤
│                              │
│  Use Subscription Slot       │
│                              │
│  Recommended Membership      │
│  ┌──────────────────────────┐│
│  │ ⭐ Household (shared)     ││
│  │   2/6 participant overlap ││
│  │   8 one-off slots left   ││
│  │   Ceiling: 2 remaining   ││
│  │   [Use This] ← recommended││
│  ├──────────────────────────┤│
│  │   Solo Standard          ││
│  │   1/6 participant overlap ││
│  │   2 one-off slots left   ││
│  │   [Use This]             ││
│  └──────────────────────────┘│
│                              │
│  ── or ──                    │
│                              │
│  ● Pay $2 one-time        ││
│    No subscription needed    │
│                              │
│  [Fund Event]                │
└──────────────────────────────┘
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
  {funding_type: "subscription_slot", slot_type: "one_off", membership_id: "..."}
→ slot consumed from membership quota, event funded
→ membership_id required when user has multiple memberships
→ omit membership_id to use default (solo) membership
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
→ returns: funding_status, funding_type, cost_spread status, limits object
```

> **Funding discovery (JF-2B):** The funding endpoint always returns a data response, even for unfunded events. It does NOT return a 404 for unfunded events. Instead, unfunded events return `funding_status: "none"` with the current limits. The frontend handles the "no funding" state as a normal data response, not an error.

The `limits` object in the response includes:

| Field | Description |
|-------|-------------|
| `person_limit` | Maximum persons allowed (e.g. 5 for unfunded) |
| `transaction_limit` | Maximum transactions allowed (e.g. 1 for unfunded) |
| `group_limit` | Maximum groups allowed (e.g. 1 for unfunded) |
| `person_count` | Current number of persons |
| `transaction_count` | Current number of transactions |
| `group_count` | Current number of groups |
| `can_add_person` | Boolean — whether the event can accept more persons |
| `can_add_transaction` | Boolean — whether the event can accept more transactions |
| `can_add_group` | Boolean — whether the event can accept more groups |

If unfunded, also needs membership quota info:

```
GET /memberships/me
→ returns: available slots for the subscription option display
```

## Orchestration — Smart Suggestion

```
GET /events/{eid}/funding/suggest
→ returns: ranked list of user's memberships with:
   [{membership_id, name, type (solo/shared), overlap_count, overlap_ratio,
     one_off_remaining, ongoing_remaining, ceiling_remaining,
     recommended: true/false}, ...]
→ ranking: overlap_ratio → quota_remaining → recency → alphabetical
```

## Orchestration — Fund from Shared Membership

```
POST /events/{eid}/funding
  {funding_type: "subscription_slot", slot_type: "one_off",
   membership_id: "shared-membership-id"}
→ slot consumed from shared membership pool
→ counts against funding user's per-member ceiling
→ 409 if ceiling exceeded, 409 if shared pool exhausted
```

## Smart Defaults

- Per-event fee pre-selected if user has no subscription slots available
- Subscription slot pre-selected if user has available slots
- Cost spread shown immediately after funding — natural next step
- Cost spread amount capped at per-event fee ($2) to prevent profiteering
- Slot type explanation is inline — users don't need to understand the quota model in depth
- Ongoing vs one-off guidance: "Use ongoing for events that will last a while (trips). Use one-off for single-day events (dinners)."
- Suggestion ranking: overlap ratio (how many event participants are membership co-members) → quota remaining → recency of membership use → alphabetical
- When only one membership exists, suggestion step is skipped — slot selection shown directly

## Error States

| Error | Display |
|-------|---------|
| No slots available | "No subscription slots available. Use per-event fee or upgrade your plan." → link to S13 |
| Already funded | "This event is already funded" (show funded state) |
| Cost spread already exists | "Cost spread already active" (prevent duplicate — known backend gap R9) |
| Event locked | "This event is locked and can no longer be modified" |
| Ceiling exceeded | "You've reached your per-member slot ceiling for this membership. Ask a co-member to fund, or use a different membership." |
| Shared membership quota exhausted | "This shared membership has no slots remaining this cycle. Upgrade or use per-event fee." |
