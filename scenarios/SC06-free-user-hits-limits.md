# SC06 — Free User Hits Limits and Funds

**Persona:** Dave (admin, free tier) discovers funding gates and resolves them.
**Screens:** S04 → S05 → S09 → S14 → S13
**Rails:** R02 (Expense), R05 (Membership & Funding)

## Narrative

Dave creates an event and starts using it within free limits:

```
S04 Create Event → "Team Lunch"
S05 → adds 4 people (within 5-person limit)
S09 → adds 1 transaction (within 1-transaction limit for unfunded events)
```

Dave tries to add a second expense:

```
S09 Add Expense → fills in "Dessert $28"
    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions → 403 Forbidden
    → Error: "This event needs funding to add more expenses"
    → [Fund This Event →] link
```

Dave navigates to funding:

```
S14 Event Funding
    Status: ⚠️ Unfunded
    Current limits: 1 transaction, 5 line items, 1 group, 5 participants, no settlements

    Options:
    ○ Use subscription slot (0 available — Dave is free tier)
    ● Pay $2 one-time

    → selects $2 → taps [Fund Event]
    ⚡ POST /events/{eid}/funding {funding_type: "per_event_fee"}
    → Status: ✅ Funded
    → Limits: 50 transactions, 500 line items, 20 groups, 50 participants, settlements enabled
```

Dave sees the cost spread option:

```
S14 → "Spread the $2 fee across all 5 participants ($1.00 each)?"
    → taps [Spread Costs]
    ⚡ POST /events/{eid}/funding/cost-spread
    → line item created spreading $2 across 5 people
    → Cost Spread: ✅ Active
```

Dave goes back and adds his expense:

```
S05 → taps "+ Add Expense" FAB
S09 → "Dessert $28", paid: Dave, split: Everyone
    ⚡ POST /events/{eid}/transactions → 201 Created
    → success!
```

## Variant: Dave Upgrades Instead

Instead of per-event fee, Dave explores membership:

```
S14 → "No subscription slots available" → [Upgrade your plan →]
S13 Membership
    Current: FREE
    → taps [Upgrade] on Standard ($9.99/mo)
    ⚡ POST /memberships {tier: "standard", billing_cycle: "monthly"}
    → 5 one-off + 2 ongoing slots available

    → back to S14
S14 → ○ Use subscription slot (5 available) ← now available!
    → selects subscription slot (one-off)
    ⚡ POST /events/{eid}/funding {funding_type: "subscription_slot", slot_type: "one_off"}
    → funded!
```

## Validates

- Funding gate error on S09 when limit exceeded
- Clear error message with direct link to funding
- Per-event fee flow on S14
- Cost spread mechanics
- Subscription slot unavailable for free users
- Membership upgrade path: S14 → S13 → back to S14
- Limits lifted after funding

## Key Insight for UI

The funding gate should feel like a **speed bump**, not a wall. The error message includes a direct action link. The funding flow is 2 taps (select method + fund). The cost spread is optional and immediately offered. The user gets back to what they were doing (adding expenses) as fast as possible.
