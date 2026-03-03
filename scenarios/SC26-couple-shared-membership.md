# SC26 — Mel & Jake: Couple Creates Shared Membership

**Persona:** Mel (from SC13) — higher earner in a couple, wants to share subscription costs with Jake.
**Screens:** S15 → S13 → S13 → S05 → S14 → S05
**Rails:** R05 (Membership & Funding)

## Design Principle

**Sharing should be as simple as splitting.** Mel and Jake already split expenses together — sharing a subscription should feel just as natural. One invite, one acceptance, and costs are shared automatically. The membership model mirrors the expense model: fair shares, not equal shares, when that's what the couple wants.

## The Situation

Mel and Jake are an established couple (from SC13 — they already split household costs 60/40). Mel has been paying for a solo Standard membership ($9.99/mo). She wants to share it with Jake so they both get event slots from a shared pool, splitting the subscription cost.

## Step 1: Mel Views Current Membership

→ Screen: S15 (Profile) → S13 (Membership)

```
Mel taps "Membership" from her profile.

S13 Membership
    ⚡ GET /memberships/me
    → Solo Standard, $9.99/mo, 3/5 one-off used this cycle

    Current Plan: ⭐ STANDARD (solo)
    Slot Usage: 3/5 one-off, 1/2 ongoing

    Mel sees [Upgrade to Premium] — which now shows "Up to 6 members"
```

## Step 2: Mel Upgrades to Standard Shared (2 Members)

→ Screen: S13 (Membership)

Standard tier already supports 2 members — no upgrade needed. Mel taps [Invite Member].

```
S13 Membership → Invite Member view
    Mel searches: "jake@example.com"
    ⚡ user lookup → Jake Miller found

    Capacity: 1 of 2 members ✅ Space available

    Mel taps [Send Invite]
    ⚡ POST /memberships/{id}/members {user_id: "jake-id"}
    → Invite sent
```

## Step 3: Jake Accepts Invite

→ Screen: S17 (Notifications) → S13 (Membership)

```
Jake receives notification:
    "Mel invited you to share their Standard membership"
    [Review ▸]

Jake taps → sees membership details and [Accept] / [Decline]
    ⚡ POST /memberships/{id}/members/{jake-id}/accept
    → Jake is now a co-member

Jake's S13 now shows:
    Membership switcher: [Solo (Free)] [Household (shared)]
    Household selected → shared Standard view
    Members: Mel (owner), Jake (you)
    Pool: 3/5 one-off used (carried from Mel's usage)
```

## Step 4: Mel Configures Cost Split and Ceilings

→ Screen: S13 (Membership)

```
Mel navigates to Cost Ledger:
    ⚡ GET /memberships/{id}/ledger

    Default: Equal split → $4.99 each

    Mel taps [Change to Usage-Based]
    ⚡ PATCH /memberships/{id} {cost_split_mode: "usage_based"}
    → Ledger recalculated based on slots used

    Mel sets per-member ceilings:
    ⚡ PATCH /memberships/{id}/members/{mel-id} {slot_ceiling: 3}
    ⚡ PATCH /memberships/{id}/members/{jake-id} {slot_ceiling: 3}
    → Each can use up to 3 one-off slots from the pool of 5
```

> **Privacy:** Jake cannot see which specific events Mel has funded — only the aggregate pool usage (3/5 one-off used). Mel sees the same about Jake.

## Step 5: Jake Funds an Event from Shared Membership

→ Screen: S05 (Event Dashboard) → S14 (Event Funding)

```
Jake creates "Household BBQ" event, adds Mel + Jake + 4 friends.

S05 Event Dashboard — "Household BBQ"
    Status: ⚠️ Unfunded
    Jake taps "Funding Status ▸"

S14 Event Funding
    ⚡ GET /events/{eid}/funding/suggest
    → Suggestion: Household (shared) recommended
      Overlap: 2/6 participants (Mel + Jake are co-members)
      Pool: 2 one-off slots remaining
      Jake's ceiling: 3 remaining

    Jake selects Household membership
    ⚡ POST /events/{eid}/funding
      {funding_type: "subscription_slot", slot_type: "one_off",
       membership_id: "household-membership-id"}
    → Event funded from shared pool
    → Jake's ceiling: 2 remaining
```

→ Screen: S05 (Event Dashboard) — limits lifted, full access

## What This Scenario Validates

- **Shared membership creation:** Invite flow from solo → shared within same tier
- **Invite flow:** Search by email, capacity check, acceptance via notification
- **Cost split modes:** Equal (default) and usage-based with ledger recalculation
- **Per-member ceilings:** Individual limits within a shared pool
- **Smart funding suggestion:** Overlap scoring recommends shared membership
- **Privacy model:** Co-members see aggregate usage, not individual event details
