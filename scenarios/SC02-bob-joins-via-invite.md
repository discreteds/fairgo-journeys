# SC02 — Bob Joins via Invite

**Persona:** Invited participant, new to app. Becomes event member.
**Screens:** S01 → S02 → S05 (pending) → S05 (active) → S09
**Rails:** R01 (Onboarding), R04 (Invitation), R02 (Expense)

## Narrative

Bob receives an invite link in a group chat: `fairgo.app/join/X7kQ2m`

```
→ Opens link in browser
S01 Welcome → deep link detected, redirects to S02 with code in session
S02 Register → creates account (display name: "Bob")
    ⚡ POST /auth/register → tokens stored
       → free tier membership auto-created (JF-4A)
    ⚡ POST /events/join {invite_code: "X7kQ2m"}
       → event_role created (status: pending_approval)
       → person auto-created (resolution_status: auto_created)
       → if admin has sponsorship enabled: Bob covered by sponsor's tier (NUP)
       → response: no potential_matches (Bob is new)
S05 Event Dashboard → "⏳ Awaiting Approval" banner
    Bob can see event name and people but can't add expenses
```

> **Note (JF-4A):** Bob's registration automatically creates a free tier membership. No tier selection is needed. If Alice (the event admin) has sponsorship enabled on her paid tier, Bob's participation in this event is covered by Alice's plan — Bob gets paid-tier benefits without upgrading.

Meanwhile, Alice (admin) sees the notification on her dashboard:

```
Alice's S05 → "⚠️ 1 pending action"
    → "Bob awaiting approval" → taps [Approve]
    ⚡ POST /events/{eid}/event-roles/{rid}/approve
    → Bob's role: active
```

Bob's view refreshes (or he pulls to refresh):

```
Bob's S05 → approval banner gone, full access
    → taps "+ Add Expense" FAB
S09 Add Expense → "Uber to restaurant $32"
    → Who paid: You (Bob) ← default
    → Split between: Everyone ← default
    → Save
    ⚡ POST /events/{eid}/transactions
S05 → balances update, everyone sees updated positions
```

## Validates

- Invite deep link → register → auto-join flow
- Pending approval state on S05
- Admin inline approval from S05 dashboard
- Member adding expenses after approval
- Auto-person creation on join (no admin action needed)

## Variant: Existing User Joins

If Bob already has an account:

```
→ Opens link → S02 Login tab
    ⚡ POST /auth/login → tokens stored
    ⚡ POST /events/join → same flow as above
```

## Variant: Person-Targeted Invite

If Alice created a personal invite for Bob's placeholder:

```
    ⚡ POST /events/join {invite_code: "personal_code"}
       → target_person_id matched → auto-claim
       → Bob linked to existing placeholder (no duplicate person)
       → resolution_status: claimed
```

## Variant: Self-Merge After Join (I1, I2)

If Alice pre-created a placeholder "Bob" and Bob joins without a personal invite:

```
Bob joins -> auto_created person "Bob" created alongside placeholder "Bob"
Bob's S05 Dashboard:
    -> "We found a match!" banner (from GET /events/{eid}/persons/my-matches)
    -> "Bob" placeholder matches Bob's display name (medium confidence)
    -> taps [This is me]
    ! POST /events/{eid}/persons/{placeholder_id}/merge {target_person_id: bob_person_id}
       -> authorize_self_merge() validates: source is placeholder, target belongs to Bob, names match
       -> merge executes: splits transferred, placeholder removed
       -> no admin intervention needed
```
