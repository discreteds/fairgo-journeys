# SC28 — Smart Funding Suggestion

**Persona:** Dave (from SC09, power user) — has solo + shared household membership.
**Screens:** S05 → S14 → S05
**Rails:** R05 (Membership & Funding)

## Design Principle

**The right membership should be obvious.** When a user has multiple memberships, the system recommends the best one based on participant overlap — because the whole point of a shared membership is funding events where co-members participate. No mental arithmetic required.

## The Situation

Dave manages multiple concurrent events (from SC09). He has:
- **Solo Standard** membership ($9.99/mo) — 2 one-off slots remaining
- **Household Premium** (shared with partner Sam) — 8 one-off slots remaining in the pool, Dave's ceiling: 4 remaining

## Step 1: Work Lunch — No Overlap with Shared Membership

→ Screen: S05 (Event Dashboard) → S14 (Event Funding)

```
Dave creates "Work Lunch Friday" — participants: Dave + 4 colleagues
None of Dave's colleagues are co-members of any membership.

S05 — "Work Lunch Friday"
    Status: ⚠️ Unfunded
    Dave taps "Funding Status ▸"

S14 Event Funding
    ⚡ GET /events/{eid}/funding/suggest
    → Returns ranked memberships:
      1. Solo Standard ← recommended
         Overlap: 1/5 (Dave only — he's the sole member)
         Slots remaining: 2
      2. Household Premium (shared)
         Overlap: 1/5 (Dave only — Sam is not a participant)
         Slots remaining: 8 pool / 4 ceiling

    Both have 1/5 overlap (just Dave), so solo ranks first
    (tie-breaking: solo preferred over shared when overlap is equal)

    Dave selects Solo Standard
    ⚡ POST /events/{eid}/funding
      {funding_type: "subscription_slot", slot_type: "one_off",
       membership_id: "solo-membership-id"}
    → Event funded, 1 one-off slot remaining on solo
```

→ Screen: S05 — limits lifted

## Step 2: Weekend BBQ — Overlap with Shared Membership

→ Screen: S05 (Event Dashboard) → S14 (Event Funding)

```
Dave creates "Weekend BBQ" — participants: Dave + Sam + 4 friends (6 total)
Sam is Dave's co-member on the Household Premium.

S05 — "Weekend BBQ"
    Status: ⚠️ Unfunded
    Dave taps "Funding Status ▸"

S14 Event Funding
    ⚡ GET /events/{eid}/funding/suggest
    → Returns ranked memberships:
      1. Household Premium (shared) ← recommended
         Overlap: 2/6 (Dave + Sam)
         Slots remaining: 8 pool / 4 ceiling
      2. Solo Standard
         Overlap: 1/6 (Dave only)
         Slots remaining: 1

    Household wins: higher overlap ratio (2/6 > 1/6)
    Badge: "Recommended — 2 participants share this membership"

    Dave selects Household Premium
    ⚡ POST /events/{eid}/funding
      {funding_type: "subscription_slot", slot_type: "one_off",
       membership_id: "household-membership-id"}
    → Event funded from shared pool
    → Dave's ceiling: 3 remaining
    → Pool: 7 one-off slots remaining
```

→ Screen: S05 — limits lifted

## Step 3: Dave Checks Shared Membership

→ Screen: S13 (Membership)

```
Dave navigates to S13 → switches to Household membership

S13 Membership — Household (shared)
    ⚡ GET /memberships/{id}
    ⚡ GET /memberships/{id}/members
    ⚡ GET /memberships/{id}/ledger

    Pool Usage: 13/20 one-off used
    Members: Dave (you) 3/4 ceiling, Sam 2/4 ceiling

    Dave sees aggregate pool usage but NOT which events Sam funded.
    Dave's own events are visible only in his event list (S03/S05).
```

## What This Scenario Validates

- **Overlap scoring:** Higher overlap ratio wins (shared membership recommended when co-members participate)
- **Tie-breaking:** Solo preferred over shared when overlap is equal (simpler, no shared pool impact)
- **Multiple membership selection:** User can override suggestion and pick any membership
- **Ceiling check:** Funding deducts from per-member ceiling within the shared pool
- **Fallback logic:** Per-event fee always available as alternative to subscription slots
- **Privacy model:** Dave sees aggregate pool usage on S13 but not Sam's individual event details
