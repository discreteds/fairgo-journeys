# SC10 — Quota Exhaustion & Recovery

**Persona:** Frank — Standard subscriber ($9.99/mo) who's been busy.
**Screens:** S05 → S14 → S13
**Rails:** R05 (Membership), R02 (Expense)

## Narrative

Frank is a Standard subscriber with 5 one-off + 2 ongoing slots per month:

```
⚡ GET /users/me/membership
→ {tier: "standard", billing_cycle: "monthly",
   slots: {one_off: {total: 5, used: 0}, ongoing: {total: 2, used: 0}}}
```

Frank creates several events, using up his one-off slots:

```
S04 → "BBQ at Dave's"
    ⚡ POST /events → created
S14 → fund with one-off slot
    ⚡ POST /events/{eid}/funding {funding_type: "subscription_slot", slot_type: "one_off"}
    → slots remaining: 4 one-off

S04 → "Office Morning Tea"       → fund → 3 remaining
S04 → "Birthday Dinner"          → fund → 2 remaining
S04 → "Team Building"            → fund → 1 remaining
S04 → "Client Lunch"             → fund → 0 remaining
```

Frank also uses his ongoing slots:

```
S04 → "Housemates Q2"
S14 → fund with ongoing slot
    ⚡ POST /events/{eid}/funding {funding_type: "subscription_slot", slot_type: "ongoing"}
    → ongoing slots remaining: 1

S04 → "Couples Budget"
S14 → fund with ongoing slot → 0 remaining
```

Frank tries to create and fund one more event:

```
S04 → "Weekend Hike"
    ⚡ POST /events → created (event creation is always free)
S14 Event Funding
    → ○ Use subscription slot — ⚠️ No slots available
    → ● Pay $4.99 one-time
    → [Upgrade your plan →] link to S13

    Current usage:
    ┌─────────────────────────────┐
    │ one-off:  █████░░░ 5/5 used │
    │ ongoing:  ██░░░░░░ 2/2 used │
    └─────────────────────────────┘
```

Frank falls back to per-event fee:

```
S14 → selects $4.99 per-event fee
    ⚡ POST /events/{eid}/funding {funding_type: "per_event_fee"}
    → funded ✓
    → anti-profiteering cap: per-event fee never exceeds subscription equivalent
```

Frank closes an event to release an ongoing slot:

```
S05 (Housemates Q2) → ⚙️ → [Close Event]
    ⚡ POST /events/{eid}/close
    → event closed
    → ongoing slot released

⚡ GET /users/me/membership
→ {slots: {ongoing: {total: 2, used: 1}}}
→ 1 ongoing slot available again!
```

## Variant: Frank Upgrades to Premium

```
S14 → taps [Upgrade your plan →]
S13 Membership
    Current: Standard ($9.99/mo) — 5 one-off, 2 ongoing
    → taps [Upgrade] on Premium ($19.99/mo)
    ⚡ POST /users/me/membership {tier: "premium"}
    → Premium: 20 one-off, 10 ongoing slots
    → existing funded events unchanged
    → new slots immediately available
    → back to S14 → subscription slots now available
```

## Validates

- Subscription slot accounting (5 one-off, 2 ongoing for Standard)
- Slot exhaustion — clear indication of 0 remaining
- Fallback to per-event fee when slots exhausted
- Ongoing slot release on event close
- Tier upgrade prompt from S14 → S13 → back to S14
- Anti-profiteering cap on per-event fees
- Event creation always succeeds (funding is separate from creation)
- Membership quota display with visual usage indicator

## Key Principle Demonstrated

> **"Speed bump, not wall"**
>
> Running out of slots doesn't prevent Frank from creating events. He always
> has the per-event fee fallback. The upgrade prompt is visible but not forced.
> Closing events releases ongoing slots, creating a natural management cycle.
