# SC06 — Free User Hits Limits

> **Rework:** Restructured to show product value before the funding gate. Original showed the wall before value.

**Persona:** Alice (admin, free tier) sees the product work before being asked to pay.
**Screens:** S04 → S05 → S09 → S05 → S09 → S14 → S09 → S05
**Rails:** R02 (Expense), R05 (Membership & Funding)

## Narrative

> **Free tier auto-creation (JF-4A):** Alice's free tier membership was automatically provisioned when she registered (see S02). She did not select a plan — the free tier is created server-side on account creation. Alice can create unlimited events; each event allows 1 transaction and 5 line items until funded.

Alice creates a new event for an upcoming trip:

```
S04 Create Event
    → name: "Camping Trip"
    ⚡ POST /events → event created
    → redirects to S05
```

Alice opens the event dashboard and adds people:

```
S05 Event Dashboard — Camping Trip
    ⚡ GET /events/{eid}
    → No expenses yet. No balances.
    → taps "People ▸"
    → adds: Alex, Sam, Casey
    ⚡ POST /events/{eid}/persons × 3
    → 3 participants total (within 5-person free limit)
    → returns to S05
```

Alice adds the first expense — within the free tier limit:

```
S05 → taps "+ Add Expense" FAB
S09 Add Expense
    → description: "Campsite Fees"
    → amount: $120.00
    → paid by: Alice (default — current user)
    → split: Everyone equally (default)
    ⚡ POST /events/{eid}/transactions
      body: {description: "Campsite Fees", amount: 120.00,
             payer_person_id: alice_pid, split: "equal"}
    → 201 Created
    → line items created: Alex $40.00, Sam $40.00, Casey $40.00
    → total: $120.00 ✓
    → returns to S05
```

Alice sees the split working — this is the value moment:

```
S05 Event Dashboard — Camping Trip
    ⚡ GET /events/{eid}?include=balances
    → Expenses:
      Campsite Fees          $120.00

    → Balances:
      Alex  owes Alice  $40.00
      Sam   owes Alice  $40.00
      Casey owes Alice  $40.00

    → Total tracked: $120.00
    → Checksum: $40.00 + $40.00 + $40.00 = $120.00 ✓

    Alice sees the complete split. Penny-exact. No manual arithmetic.
    The product has demonstrated its value on the first expense.
```

Alice tries to add a second expense — quota hit:

```
S05 → taps "+ Add Expense" FAB
S09 Add Expense
    → description: "Food & Supplies"
    → amount: $89.00
    → paid by: Alice
    → split: Everyone equally
    → taps [Save Expense]
    ⚡ POST /events/{eid}/transactions
    → 403 Forbidden
    → Error (inline, below form):
      "You've used your free expense for this event.
       Unlock unlimited expenses for $2. [Unlock This Event →]"
```

Alice taps the inline link and arrives at Event Funding:

```
S14 Event Funding — Camping Trip
    ⚡ GET /events/{eid}/funding
    → funding_status: "none"
    ⚡ GET /memberships/me
    → subscription slots: 0 available (free tier)

    ┌──────────────────────────────────────────────┐
    │  ← Funding · Camping Trip                    │
    ├──────────────────────────────────────────────┤
    │                                              │
    │  Status: ⚠️ Unfunded                         │
    │                                              │
    │  What you've already tracked                 │
    │  ┌──────────────────────────────────────────┐│
    │  │ Campsite Fees · $120.00 · split 3 ways   ││
    │  │ 1 expense · 3 line items                 ││
    │  └──────────────────────────────────────────┘│
    │                                              │
    │  Unlock unlimited expenses for this event    │
    │  ┌──────────────────────────────────────────┐│
    │  │ ○ Use subscription slot                  ││
    │  │   No slots available                     ││
    │  │                                          ││
    │  │ ● Pay $2 one-time                        ││
    │  │   No subscription needed.                ││
    │  │   Less than $0.67 per person.            ││
    │  │                                          ││
    │  │ [Unlock This Event]                      ││
    │  └──────────────────────────────────────────┘│
    └──────────────────────────────────────────────┘

    → $2 per-event fee pre-selected (no subscription slots available)
    → per-person anchor ("Less than $0.67 per person") frames $2 as trivial
      relative to the $120.00 already tracked
```

Alice taps [Unlock This Event]:

```
S14 → taps [Unlock This Event]
    ⚡ POST /events/{eid}/funding
      body: {funding_type: "per_event_fee"}
    → 200 OK
    → funding_status: "funded"
    → limits lifted: 50 transactions, 500 line items, 20 groups,
                     50 participants, settlements enabled

    → Status: ✅ Funded

    Cost Spread prompt appears immediately:
    ┌──────────────────────────────────────────────┐
    │  Cost Spread                                 │
    │  Spread the $2 fee across all 3 participants ││
    │  ($0.67 each)?                               │
    │                                              │
    │  This adds a Fair Go fee line item so        │
    │  everyone shares the tool cost.              │
    │                                              │
    │  [Spread Costs]  [Skip]                      │
    └──────────────────────────────────────────────┘
```

Alice taps [Spread Costs]:

```
S14 → taps [Spread Costs]
    ⚡ POST /events/{eid}/funding/cost-spread
      body: {spread_to: "all_participants"}
    → system line item created: "Fair Go — Camping Trip ($0.67 each)"
    → visually distinct from regular expenses (greyed label, app-fee badge)
    → Cost Spread: ✅ Active · $2 split across 3 people ($0.67 per person)
```

Alice returns to the expense form and enters the second expense:

```
S14 → taps ← back
S05 → taps "+ Add Expense" FAB
S09 Add Expense
    → description: "Food & Supplies"
    → amount: $89.00
    → paid by: Alice
    → split: Everyone equally
    → taps [Save Expense]
    ⚡ POST /events/{eid}/transactions
    → 201 Created
    → line items: Alex $29.67, Sam $29.67, Casey $29.66
    → returns to S05
```

Alice sees both expenses on the dashboard:

```
S05 Event Dashboard — Camping Trip
    ⚡ GET /events/{eid}?include=balances
    → Expenses:
      Campsite Fees          $120.00
      Food & Supplies         $89.00
      Fair Go fee ⬩            $2.00   ← app-fee badge; visually distinct

    → Balances:
      Alex  owes Alice   $70.33
      Sam   owes Alice   $70.33
      Casey owes Alice   $70.34

    → Total tracked: $211.00
    → Settlements: enabled
```

## Variant: Alice Upgrades Instead of Paying Per-Event

At S14, if Alice explores the subscription path:

```
S14 → "No subscription slots available" → [Upgrade your plan →]
S13 Membership
    Current: FREE
    → taps [Upgrade] on Standard ($9.99/mo)
    ⚡ POST /memberships {tier: "standard", billing_cycle: "monthly"}
    → 5 one-off + 2 ongoing slots available

    → back to S14
S14 → ○ Use subscription slot (5 available) ← now available
    → selects subscription slot (one-off)
    ⚡ POST /events/{eid}/funding
      body: {funding_type: "subscription_slot", slot_type: "one_off"}
    → funded
```

> **Note:** This upgrade variant is realistic for users already on their third or fourth event, not typically the first time hitting the gate. The $2 per-event path is the expected first-time conversion.

## Validates

- Free tier event creation (unlimited events, 1 transaction limit per event)
- Expense entry and equal split within free tier limit
- VALUE MOMENT: balances visible before quota gate is reached
- Penny-exact split checksum on first expense
- 403 quota error on second transaction attempt with inline upgrade prompt
- S14 funding screen: unfunded state with what-you've-tracked summary
- Per-person cost anchor framing the $2 fee as proportionate
- Per-event fee funding flow (2 taps: select → fund)
- Cost spread mechanics: system line item, visual distinction from real expenses
- Limits lifted after funding — second expense succeeds
- Both expenses visible on S05 dashboard post-funding
- Subscription slot unavailable for free users; upgrade path via S13

## Key Insight for UI

The funding gate should feel like a **speed bump after a win**, not a toll booth on the on-ramp.

Alice sees a complete, working split before the gate appears. The upgrade prompt anchors the $2 fee against what she's already got (a $120 split, done correctly, automatically) and what she's about to add. The per-person cost frame ("less than $0.67 per person") makes the fee feel proportionate to the event, not to the app. The cost spread converts the admin's payment into a shared cost — the tool pays for itself across participants.

The free tier differentiator: **even free users see penny-exact arithmetic before being asked to pay.**
