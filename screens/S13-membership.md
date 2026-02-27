# S13 — Membership

**Purpose:** View/manage subscription tier and quotas.
**Visible to:** Authenticated users (from S15 Profile).
**Rails:** R05 (Membership & Funding)
**Scenarios:** SC06

## Wireframe — Free User

```
┌──────────────────────────────┐
│  ← Membership                │
├──────────────────────────────┤
│                              │
│  Current Plan                │
│  ┌──────────────────────────┐│
│  │  FREE                    ││
│  │  No event slots          ││
│  │  Pay $4.99 per event     ││
│  └──────────────────────────┘│
│                              │
│  Upgrade                     │
│  ┌──────────────────────────┐│
│  │  ⭐ Standard  $9.99/mo   ││
│  │  5 one-off + 2 ongoing   ││
│  │  event slots per month   ││
│  │  [Upgrade]               ││
│  ├──────────────────────────┤│
│  │  💎 Premium  $19.99/mo   ││
│  │  20 one-off + 10 ongoing ││
│  │  event slots per month   ││
│  │  [Upgrade]               ││
│  └──────────────────────────┘│
└──────────────────────────────┘
```

## Wireframe — Subscribed User

```
┌──────────────────────────────┐
│  ← Membership                │
├──────────────────────────────┤
│                              │
│  Current Plan                │
│  ┌──────────────────────────┐│
│  │  ⭐ STANDARD             ││
│  │  $9.99/mo · Monthly      ││
│  │  Next billing: Mar 15    ││
│  └──────────────────────────┘│
│                              │
│  Slot Usage This Cycle       │
│  ┌──────────────────────────┐│
│  │ One-off:   3 / 5 used   ││
│  │ ████████░░░░░░           ││
│  │                          ││
│  │ Ongoing:   1 / 2 active  ││
│  │ █████░░░░░░░░░           ││
│  └──────────────────────────┘│
│                              │
│  ┌──────────────────────────┐│
│  │ [Upgrade to Premium]     ││
│  │ [Change to Annual]       ││
│  │ [Cancel Subscription]    ││
│  └──────────────────────────┘│
└──────────────────────────────┘
```

## Orchestration — Page Load

```
GET /memberships/me
→ returns: tier, billing_cycle, quotas (one_off used/total, ongoing used/total),
   next_billing_date
```

## Orchestration — "Upgrade" (Free → Standard)

```
POST /memberships
  {tier: "standard", billing_cycle: "monthly"}
→ membership created, quotas activated
```

## Orchestration — "Upgrade to Premium" (Standard → Premium)

```
PUT /memberships/me
  {tier: "premium"}
→ tier upgraded, quotas reset to premium levels
```

## Orchestration — "Change to Annual"

```
PUT /memberships/me
  {billing_cycle: "annual"}
→ billing cycle changed
```

## Orchestration — "Cancel Subscription"

```
1. Confirm dialog: "Cancel your subscription? You'll keep access until the end
   of your billing period. Your ongoing event slots will remain active."
2. PUT /memberships/me
     {tier: "free"}
   → downgraded to free at end of cycle
```

## Tier Comparison

| Feature | Free | Standard | Premium |
|---------|------|----------|---------|
| Per-event fee | $4.99 | $4.99 | $4.99 |
| One-off slots/month | 0 | 5 | 20 |
| Ongoing slots/month | 0 | 2 | 10 |
| Monthly price | $0 | $9.99 | $19.99 |

## Smart Defaults

- Slot usage shown as visual progress bars
- Upgrade path highlighted based on current usage (if near limits, suggest upgrade)
- Billing cycle defaults to monthly (lower commitment)
- Cancel confirmation explains what happens to existing events

## Error States

| Error | Display |
|-------|---------|
| Already cancelled | "Your subscription is already cancelled" (409) |
| Payment failure | "Payment couldn't be processed. Update your payment method." |
| Downgrade with active ongoing slots | Warning: "You have X ongoing slots in use. These will remain active." |
