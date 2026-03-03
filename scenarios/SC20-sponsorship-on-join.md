# SC20 — Sponsorship on Join

> **New User Path:** A sponsor joins an event and brings dependents (children) who settle under their account — one settlement group for the family.

**Persona:** Sarah (new user), joining Marcus's group holiday with her two kids.
**Screens:** S06 → S18 → S02 → S05 → S07 → S09 → S11 → S12
**Rails:** R01 (Onboarding), R04 (Invitation), R02 (Expense), R03 (Settlement)

## Design Principle

**"Bring your family, settle as one."**

Sarah's kids participate in activities and consume their share, but Sarah is the one who pays. Fair Go's settlement groups (PFGs) roll up the family's balances into a single number — Sarah sees one amount to pay, not three separate debts.

## Participants

| Person | Role | Settlement Group | Notes |
|--------|------|-----------------|-------|
| Marcus | Admin, payer | Marcus 🔵 | Event organiser |
| Sarah | Member, payer, sponsor | Sarah's Family 🟢 | Joins via invite, sponsors kids |
| Tom | Dependent | Sarah's Family 🟢 | Age 12, claimed by Sarah |
| Emma | Dependent | Sarah's Family 🟢 | Age 9, claimed by Sarah |
| Liam | Member, payer | Liam 🟡 | Another adult attendee |
| Jade | Member | Jade 🔴 | Another adult attendee |

## Narrative

### 1. Marcus creates event and adds placeholders

Marcus sets up the holiday event and adds person placeholders for everyone he expects to attend, including Sarah's kids.

```
S04 Create Event → "Beach Holiday 2026"
    ⚡ POST /events {name: "Beach Holiday 2026", currency: "AUD"}

S07 Manage People → adds placeholders
    ⚡ POST /events/{eid}/persons × 5
       names: "Sarah", "Sarah's kid 1", "Sarah's kid 2", "Liam", "Jade"
```

### 2. Marcus creates a person-targeted invite for Sarah

```
S06 Prepare & Share
    → selects "Sarah" placeholder → taps [Create Personal Invite]
    ⚡ POST /events/{eid}/invite-codes
       {type: "personal", target_person_id: sarah_placeholder_id}
    → generates invite link with embedded target
    → shares link with Sarah via message
```

### 3. Sarah receives invite and registers

```
Sarah opens invite link in browser

S18 Invite Landing
    ┌──────────────────────────────────────────────────┐
    │  🏖️ Beach Holiday 2026                           │
    │  Organised by Marcus                              │
    │  6 participants · 0 expenses so far              │
    │                                                  │
    │  You're invited as: Sarah                        │
    │                                                  │
    │  ┌────────────────────────────────┐              │
    │  │     Join this event →          │              │
    │  └────────────────────────────────┘              │
    └──────────────────────────────────────────────────┘

    → taps "Join this event"
    → invite code stored in session

S02 Register
    → Sarah creates account
    ⚡ POST /auth/register {display_name: "Sarah", email, password}
       Headers: Idempotency-Key: <uuid> (see A07)
       → account created, free tier auto-provisioned
    ⚡ POST /events/join {invite_code: "..."}
       → auto-claims "Sarah" placeholder (target_person_id match)
       → Sarah is now a member with her person linked

    → S05 Event Dashboard
```

### 4. Sarah renames and claims her kids

Sarah sees the placeholder names Marcus created and renames them to the actual names.

```
S05 → taps "People ▸" → S07 Manage People

    Sarah sees: Sarah (you), Sarah's kid 1, Sarah's kid 2, Liam, Jade, Marcus

    → taps "Sarah's kid 1" → renames to "Tom"
    ⚡ PATCH /events/{eid}/persons/{tom_placeholder_id}
       {display_name: "Tom"}

    → taps "Sarah's kid 2" → renames to "Emma"
    ⚡ PATCH /events/{eid}/persons/{emma_placeholder_id}
       {display_name: "Emma"}
```

### 5. Sarah creates a family settlement group

S07 surfaces the PFG discovery prompt after Sarah renames the kids:

```
S07 Manage People
    ┌─────────────────────────────────────────────────────┐
    │  💡 Do any of these people settle together?         │
    │     (e.g. a parent paying for children)             │
    │                                        [Add group]  │
    └─────────────────────────────────────────────────────┘

    → Sarah taps [Add group]
    → selects Sarah, Tom, Emma
    → names the group "Sarah's Family"

    ⚡ POST /events/{eid}/pfgs
       {name: "Sarah's Family", member_ids: [sarah_id, tom_id, emma_id]}
       → creates non-singleton PFG
       → all three persons now share a settlement group 🟢
```

### 6. Expenses

Three expenses are added over the holiday:

**Expense 1: Accommodation** — Marcus pays, split equally across all 6 people.

```
S09 Add Expense (Marcus)
    Description: "Beach House"
    Amount: $1,200.00
    Paid by: Marcus
    Split: Everyone equally (6 people, $200.00 each)

    ⚡ POST /events/{eid}/transactions
       Headers: Idempotency-Key: <uuid>
       {line_items: [{amount: "1200.00", payer_person_id: marcus_id,
         splits: [{marcus, expense, 1}, {marcus, consumption, 1},
                  {sarah, consumption, 1}, {tom, consumption, 1},
                  {emma, consumption, 1}, {liam, consumption, 1},
                  {jade, consumption, 1}]}]}
```

**Expense 2: Activities for all** — Liam pays, split equally across all 6.

```
S09 Add Expense (Liam)
    Description: "Kayak Hire"
    Amount: $300.00
    Paid by: Liam
    Split: Everyone equally (6 people, $50.00 each)

    ⚡ POST /events/{eid}/transactions
       Headers: Idempotency-Key: <uuid>
       {line_items: [{amount: "300.00", payer_person_id: liam_id,
         splits: [{liam, expense, 1}, {all 6, consumption, 1}]}]}
```

**Expense 3: Kids' activity** — Sarah pays, split between Tom and Emma only.

```
S09 Add Expense (Sarah)
    Description: "Kids Surf Lesson"
    Amount: $120.00
    Paid by: Sarah
    Split: Tom, Emma only ($60.00 each)

    ⚡ POST /events/{eid}/transactions
       Headers: Idempotency-Key: <uuid>
       {line_items: [{amount: "120.00", payer_person_id: sarah_id,
         splits: [{sarah, expense, 1},
                  {tom, consumption, 1}, {emma, consumption, 1}]}]}
```

### 7. Per-person positions

```
S11 Balances

    Person    Paid        Consumed    Net
    ──────    ────        ────────    ────
    Marcus    $1,200.00   $250.00     +$950.00
    Sarah     $120.00     $250.00     −$130.00
    Tom       $0.00       $310.00     −$310.00
    Emma      $0.00       $310.00     −$310.00
    Liam      $300.00     $250.00     +$50.00
    Jade      $0.00       $250.00     −$250.00
    ──────    ────        ────────    ────
    Total     $1,620.00   $1,620.00   $0.00 ✓

    Per-person consumed:
      Marcus: $200 (house) + $50 (kayak) = $250
      Sarah:  $200 (house) + $50 (kayak) = $250
      Tom:    $200 (house) + $50 (kayak) + $60 (surf) = $310
      Emma:   $200 (house) + $50 (kayak) + $60 (surf) = $310
      Liam:   $200 (house) + $50 (kayak) = $250
      Jade:   $200 (house) + $50 (kayak) = $250
```

### 8. PFG rollup — Sarah settles for her family

```
S11 Balances (PFG view)

    Settlement Group      Net
    ────────────────      ────
    Marcus 🔵             +$950.00
    Sarah's Family 🟢    −$750.00   (Sarah −$130 + Tom −$310 + Emma −$310)
    Liam 🟡              +$50.00
    Jade 🔴              −$250.00
    ────────────────      ────
    Checksum              $0.00 ✓
```

Sarah sees one number: her family owes $750 total. She doesn't see three separate debts.

### 9. Settlement

```
S12 Settle Up → server-computed suggestions

    Settlement 1: Sarah's Family 🟢 → Marcus 🔵   $700.00
    Settlement 2: Jade 🔴 → Marcus 🔵              $250.00
    Settlement 3: Sarah's Family 🟢 → Liam 🟡     $50.00
                                                   ─────────
    Total transferred:                              $1,000.00

    Verification:
      Marcus receives: $700 + $250 = $950 ✓ (net: +$950)
      Sarah's Family pays: $700 + $50 = $750 ✓ (net: −$750)
      Liam receives: $50 ✓ (net: +$50)
      Jade pays: $250 ✓ (net: −$250)
```

Sarah makes 2 payments totalling $750. Tom and Emma don't need accounts or payment methods — Sarah handles everything.

## Validates

- Person-targeted invites with auto-claim on join (NUP Path D)
- Sponsor claiming and renaming dependent placeholders
- PFG creation grouping sponsor + dependents into one settlement unit
- PFG rollup: 3 individual debts → 1 family debt
- Dependents participate in splits without needing accounts
- Settlement suggestions use PFGs, not individuals
- Zero-sum checksum at both person and PFG level

## Key Principle Demonstrated

> **"Bring your family, settle as one."** Sarah's kids eat, play, and consume their fair share — but Sarah is the one who pays. The PFG rolls up three individual positions into one family balance. No separate accounts for kids, no manual addition, no confusion about who owes what.
