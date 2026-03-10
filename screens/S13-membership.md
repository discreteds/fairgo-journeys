# S13 — Membership

**Purpose:** View/manage subscription tier, quotas, and shared membership.
**Visible to:** Authenticated users (from S15 Profile).
**Rails:** R05 (Membership & Funding)
**Scenarios:** SC06, SC10, SC26, SC27

## Free Tier Auto-Creation (JF-4A)

Every new user automatically receives a free tier membership upon registration (see S02). No manual plan selection is required. The membership screen reflects this default state — new users arriving here for the first time will see the "FREE" plan already active. Upgrade options are always available from this screen.

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
│  │  Pay $2 per event     ││
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

## Wireframe — Shared Membership

When a user belongs to a shared membership (Standard with 2 members, or Premium with 4–6), the membership switcher appears at the top. The shared view shows aggregate pool usage and a member list.

```
┌──────────────────────────────┐
│  ← Membership                │
├──────────────────────────────┤
│                              │
│  ┌──────────────────────────┐│
│  │ [Solo ▾]  [Household ▾]  ││
│  └──────────────────────────┘│
│                              │
│  Current Plan                │
│  ┌──────────────────────────┐│
│  │  💎 PREMIUM  (shared)    ││
│  │  $19.99/mo · Monthly     ││
│  │  3 of 4 members          ││
│  │  Next billing: Mar 15    ││
│  └──────────────────────────┘│
│                              │
│  Pool Usage This Cycle       │
│  ┌──────────────────────────┐│
│  │ One-off:  12 / 20 used  ││
│  │ ████████████░░░░░░░░     ││
│  │                          ││
│  │ Ongoing:   4 / 10 active ││
│  │ ████░░░░░░░░░░░░         ││
│  └──────────────────────────┘│
│                              │
│  Members                     │
│  ┌──────────────────────────┐│
│  │ 👤 Mel (you)   3/5 ceil ││
│  │ 👤 Jake         2/3 ceil ││
│  │ 👤 Sam          1/4 ceil ││
│  │                          ││
│  │ [Invite Member]          ││
│  └──────────────────────────┘│
│                              │
│  Cost Split: Equal           │
│  ┌──────────────────────────┐│
│  │ [View Cost Ledger]       ││
│  │ [Governance Actions (1)] ││
│  │ [Manage Membership]      ││
│  └──────────────────────────┘│
└──────────────────────────────┘
```

> **Privacy:** Each member sees aggregate pool usage and their own per-member ceiling, but cannot see which specific events other members have funded. The member list shows only slot ceilings, not individual usage breakdown.

## Wireframe — Invite Member

Accessed via [Invite Member] button on the shared membership view.

```
┌──────────────────────────────┐
│  ← Invite Member             │
├──────────────────────────────┤
│                              │
│  Search by email or username │
│  ┌──────────────────────────┐│
│  │ 🔍 jake@example.com     ││
│  └──────────────────────────┘│
│                              │
│  ┌──────────────────────────┐│
│  │ 👤 Jake Miller           ││
│  │    jake@example.com      ││
│  │    [Send Invite]         ││
│  └──────────────────────────┘│
│                              │
│  Capacity: 1 of 2 members   │
│  ✅ Space available          │
│                              │
└──────────────────────────────┘
```

## Wireframe — Governance Action

Governance actions require consensus from existing members. Displayed as pending action cards with vote buttons.

```
┌──────────────────────────────┐
│  ← Governance Actions        │
├──────────────────────────────┤
│                              │
│  Pending Actions             │
│  ┌──────────────────────────┐│
│  │ ⚠️ Remove Jordan          ││
│  │ Requested by: Alex       ││
│  │ Expires in: 5 days       ││
│  │                          ││
│  │ [Approve]    [Reject]    ││
│  └──────────────────────────┘│
│                              │
│  ── 2-member variant ──      │
│                              │
│  ┌──────────────────────────┐│
│  │ ⚠️ Dissolve Membership    ││
│  │ Requested by: Alex       ││
│  │ Expires in: 5 days       ││
│  │                          ││
│  │ ℹ️ With 2 members,        ││
│  │ removal is not available. ││
│  │ Dissolution reverts both  ││
│  │ members to solo plans.    ││
│  │                          ││
│  │ [Approve]    [Reject]    ││
│  └──────────────────────────┘│
│                              │
└──────────────────────────────┘
```

## Wireframe — Cost Ledger

Shows how the subscription cost is split across members.

```
┌──────────────────────────────┐
│  ← Cost Ledger               │
├──────────────────────────────┤
│                              │
│  Cost Split Mode: Equal      │
│  [Change to Usage-Based ▾]   │
│                              │
│  ┌──────────────────────────┐│
│  │ Member    Ceil  Share    ││
│  │ ──────────────────────── ││
│  │ Mel       5     $6.66   ││
│  │ Jake      3     $6.66   ││
│  │ Sam       4     $6.67   ││
│  │ ──────────────────────── ││
│  │ Total           $19.99  ││
│  └──────────────────────────┘│
│                              │
│  ── usage-based variant ──   │
│                              │
│  ┌──────────────────────────┐│
│  │ Member    Used  Share    ││
│  │ ──────────────────────── ││
│  │ Mel       6     $9.99   ││
│  │ Jake      2     $3.33   ││
│  │ Sam       4     $6.67   ││
│  │ ──────────────────────── ││
│  │ Total    12     $19.99  ││
│  └──────────────────────────┘│
│                              │
└──────────────────────────────┘
```

## Orchestration — Page Load

```
GET /memberships/me
→ returns: tier, billing_cycle, quotas (one_off used/total, ongoing used/total),
   next_billing_date
```

## Orchestration — Page Load (Shared)

```
GET /memberships/{id}
→ returns: tier, billing_cycle, quotas, cost_split_mode, member_count, max_members

GET /memberships/{id}/members
→ returns: [{user_id, display_name, slot_ceiling, joined_at}, ...]

GET /memberships/{id}/ledger
→ returns: [{user_id, display_name, slots_used, fair_share_amount}, ...],
   cost_split_mode, total_cost
```

## Orchestration — "Upgrade" (Free → Standard)

```
POST /users/me/membership
  {tier: "standard", billing_cycle: "monthly"}
→ 201: membership created, quotas activated
→ 200: if membership already exists, upgrades tier if different
```

> **Idempotent upgrade:** `POST /users/me/membership` returns 200 (not 409) if a membership already exists and the requested tier is different — it upgrades the tier in place. If the tier is the same, it returns 200 with the existing membership unchanged.

## Orchestration — "Upgrade to Premium" (Standard → Premium)

```
PUT /memberships/me
  {tier: "premium"}
→ tier upgraded, quotas reset to premium levels
```

## Orchestration — "Change to Annual"

```
PATCH /users/me/membership
  {billing_cycle: "annual"}
→ billing cycle changed
→ accepts "monthly" or "annual" as billing_cycle values
```

## Orchestration — "Cancel Subscription"

```
1. Confirm dialog: "Cancel your subscription? You'll keep access until the end
   of your billing period. Your ongoing event slots will remain active."
2. PUT /memberships/me
     {tier: "free"}
   → downgraded to free at end of cycle
```

## Orchestration — Invite Member

```
POST /memberships/{id}/members
  {user_id: "..."}
→ invite sent, member added (pending acceptance)
→ 409 if at capacity, 404 if user not found, 409 if already member
```

## Orchestration — Update Ceiling

```
PATCH /memberships/{id}/members/{user_id}
  {slot_ceiling: 5}
→ per-member ceiling updated
```

## Orchestration — Change Cost Split Mode

```
PATCH /memberships/{id}
  {cost_split_mode: "usage_based"}
→ cost split mode changed, ledger recalculated
→ valid modes: "equal", "usage_based"
```

## Orchestration — Request Removal

```
POST /memberships/{id}/actions
  {type: "remove_member", target_user_id: "..."}
→ governance action created, notifications sent to other members
→ requires consensus from all remaining members
```

## Orchestration — Vote on Action

```
POST /memberships/{id}/actions/{action_id}/vote
  {vote: "approve"}
→ vote recorded
→ if consensus reached: action executes (member removed, or membership dissolved)
→ if rejected: action cancelled
```

## Orchestration — Leave Membership

```
DELETE /memberships/{id}/members/{self_user_id}
→ voluntary departure, no vote required
→ user reverts to solo membership
→ if only 1 member remains: membership auto-dissolves
```

## Tier Comparison

| Feature | Free | Registered | Standard | Premium |
|---------|------|------------|----------|---------|
| Per-event fee | $2 | $2 | $2 | $2 |
| One-off slots/month | 0 | 0 | 5 | 20 |
| Ongoing slots/month | 0 | 0 | 2 | 10 |
| Max members | 1 | 1 | 2 | 4–6 |
| Monthly price | $0 | $0 | $9.99 | $19.99 |

## Smart Defaults

- Slot usage shown as visual progress bars
- Upgrade path highlighted based on current usage (if near limits, suggest upgrade)
- Billing cycle defaults to monthly (lower commitment)
- Cancel confirmation explains what happens to existing events
- Membership switcher defaults to the most recently viewed membership
- Cost split mode defaults to "equal" when creating a shared membership
- Per-member ceiling defaults to pool total ÷ member count (even split of capacity)

## Error States

| Error | Display |
|-------|---------|
| Already cancelled | "Your subscription is already cancelled" (409) |
| Payment failure | "Payment couldn't be processed. Update your payment method." |
| Downgrade with active ongoing slots | Warning: "You have X ongoing slots in use. These will remain active." |
| Invite — at capacity | "This membership has reached its member limit. Upgrade to add more members." |
| Invite — user not found | "No user found with that email or username." |
| Invite — already member | "This user is already a member of this membership." |
| Governance — action expired | "This action has expired. Request a new one if needed." |
| Governance — already voted | "You have already voted on this action." |
| Governance — cannot remove in 2-member | "With only 2 members, use dissolution instead of removal." |
