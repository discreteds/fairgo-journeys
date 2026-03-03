# R05 — Membership & Funding Rail

**Purpose:** Manage subscription tier, shared membership, and fund events.
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
  │ [Fund Event] (slot or $2)
  ▼
S14 Event Funding → funded
  │ [Spread Costs] (optional)
  ▼
S05 Event Dashboard → limits lifted, full access
```

### Shared Membership Path

```
S13 Membership → "Members" / "Invite"
  │ invite flow or member management
  ▼
S13 Membership → member added/removed
  │
  │ ···· governance action created ····
  │
  ▼
S17 Notifications → vote request received
  │ "Review Action ▸"
  ▼
S13 Membership → vote on action → action executes or expires
```

### Shared Membership Funding Path

```
S05 Event Dashboard → "Funding Status ▸"
  ▼
S14 Event Funding
  │ ⚡ GET /events/{eid}/funding/suggest → ranked memberships
  │ user selects recommended membership
  ▼
S14 Event Funding → funded from shared pool
  │ [Spread Costs] (optional)
  ▼
S05 Event Dashboard → limits lifted
```

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S14 → S13 | No subscription slots, user wants to upgrade | Detour to membership |
| S13 → S14 | After upgrading, return to fund event | Back to funding |
| S09 → S14 | Expense blocked by funding gate | Error link to funding |
| S14 → S05 | Event funded | Return to dashboard |
| S13 → S17 | Governance action created, notification sent to co-members | Co-member receives vote request |
| S17 → S13 | Co-member taps to review and vote on action | Deep link to governance view |
| S14 → S13 | User wants to check/manage shared membership before funding | Detour to membership management |

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

Shared membership funding path:

```
GET /events/{eid}/funding/suggest
POST /events/{eid}/funding {funding_type: "subscription_slot", slot_type: "one_off",
  membership_id: "shared-membership-id"}
POST /events/{eid}/funding/cost-spread   # optional
```

> **Free tier auto-creation (JF-4A):** On join, if the user has no membership tier, a free tier is auto-created server-side. No detour to S13 required before participating in events.

## Scenarios Using This Rail

- SC06 (Free User Hits Limits) — funding gate → per-event fee → cost spread
- SC06 variant — funding gate → upgrade membership → subscription slot
- SC10 (Quota Exhaustion Recovery) — quota hit → upgrade tier → fund event
- SC26 (Couple Shared Membership) — invite co-member → configure cost split → fund from shared pool
- SC27 (Membership Governance) — request removal → vote → dissolution
- SC28 (Smart Funding Suggestion) — overlap scoring → select membership → fund event
