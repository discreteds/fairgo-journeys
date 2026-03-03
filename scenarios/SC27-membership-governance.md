# SC27 — Governance: Removal & Dissolution

**Persona:** Alex (housemate admin) — managing a 3-person shared Premium membership.
**Screens:** S13 → S17 → S13 → S13
**Rails:** R05 (Membership & Funding)

## Design Principle

**No one gets removed without consensus.** Shared memberships involve shared money. Removing a member affects everyone's costs, so everyone gets a vote. The exception: you can always leave voluntarily — no one is trapped. And when only 2 members remain, removal becomes dissolution — because removing one of two people is the same as ending the membership.

## The Situation

Alex, Sam, and Jordan share a Premium membership ($19.99/mo, up to 6 members). They've been splitting it equally at $6.67 each. Jordan is moving out and needs to leave the membership.

## Step 1: Alex Requests Jordan's Removal

→ Screen: S13 (Membership — Governance Actions)

```
S13 Membership — Household (shared)
    Members: Alex (you), Sam, Jordan
    Pool: 8/20 one-off used this cycle

    Alex taps [Governance Actions]
    Alex taps [Request Action ▸]
    Action type: "Remove Member"
    Target: Jordan

    ⚡ POST /memberships/{id}/actions
      {type: "remove_member", target_user_id: "jordan-id"}
    → Governance action created
    → Notifications sent to Sam (only non-requesting, non-target members vote)

    Status: "Pending — waiting for Sam's vote"
```

## Step 2: Sam Receives Vote Notification

→ Screen: S17 (Notifications)

```
Sam's Activity feed:
    "Alex requested to remove Jordan from Household membership"
    "Your vote is required"
    Expires in: 7 days
    [Review Action ▸]
```

## Step 3: Sam Votes to Approve

→ Screen: S13 (Membership — Governance Actions)

```
Sam taps [Review Action ▸] → S13 Governance view

    ⚠️ Remove Jordan
    Requested by: Alex
    Expires in: 5 days

    Impact:
    - Jordan's slots (3 used this cycle) will be finalized
    - Cost ledger will be recalculated for 2 members
    - Jordan reverts to solo Free membership

    Sam taps [Approve]
    ⚡ POST /memberships/{id}/actions/{action-id}/vote
      {vote: "approve"}
    → Consensus reached (Alex requested + Sam approved)
    → Jordan removed from membership
    → Jordan's ledger finalized, reverts to solo Free
    → Cost split recalculated: $19.99 ÷ 2 = $9.99 each
```

## Step 4: The 2-Member Edge Case — Dissolution

→ Screen: S13 (Membership)

Months later, Alex and Sam are the only two members. Alex wants to end the shared membership.

```
S13 Membership — Household (shared)
    Members: Alex (you), Sam
    2 members remaining

    Alex taps [Governance Actions]
    → "Remove Member" is NOT available (2-member case)
    → Only "Request Dissolution" is shown

    ℹ️ "With only 2 members, individual removal is not available.
    Dissolution will end the shared membership and revert both
    members to solo plans."

    Alex taps [Request Dissolution]
    ⚡ POST /memberships/{id}/actions
      {type: "dissolve"}
    → Notification sent to Sam
```

## Step 5: Sam Approves Dissolution

→ Screen: S17 (Notifications) → S13 (Membership)

```
Sam receives notification:
    "Alex has requested to dissolve the Household membership"
    [Review Action ▸]

S13 Governance view:
    ⚠️ Dissolve Membership
    Requested by: Alex
    Impact: Both members revert to solo plans. Ledger finalized.

    Sam taps [Approve]
    ⚡ POST /memberships/{id}/actions/{action-id}/vote
      {vote: "approve"}
    → Membership dissolved
    → Both Alex and Sam revert to solo Free memberships
    → Final ledger settled
```

## Alternative: Jordan Leaves Voluntarily

Instead of Steps 1–3, Jordan could leave on their own without requiring a vote:

```
Jordan navigates to S13 → Household membership
    Jordan taps [Leave Membership]

    Confirm: "Leave this membership? You'll revert to a solo plan.
    Your share of the current billing cycle will be finalized."

    ⚡ DELETE /memberships/{id}/members/{jordan-id}
    → Jordan removed immediately (no vote needed for self-departure)
    → Jordan reverts to solo Free membership
    → Cost split recalculated for remaining 2 members
```

## What This Scenario Validates

- **Consensus removal:** All non-target, non-requesting members must vote
- **Notification flow:** Vote requests appear in S17 Activity with deep links to S13
- **2-member edge case:** Removal unavailable, only dissolution (removing 1 of 2 = ending the membership)
- **Voluntary departure:** Self-removal is instant, no governance vote required
- **Ledger finalization:** Removed/departing member's usage is finalized and cost share settled
- **Privacy throughout:** Members never see each other's individual event details during governance
