# S06 — Prepare & Share (Invite & Join)

**Purpose:** Two-sided screen. Admin prepares person checklist and shares invite codes. New user redeems code.
**Visible to:** Admin (generate side) / Unauthenticated or authenticated (redeem side).
**Rails:** R04 (Invitation)
**Scenarios:** SC01

## Wireframe — Prepare & Share (Admin)

Reached from S05 "Share Event →" or S07 "Send Personal Invite".

```
┌──────────────────────────────┐
│  ← Prepare & Share           │
├──────────────────────────────┤
│                              │
│  People to Share With        │
│  ┌──────────────────────────┐│
│  │ Bob                      ││
│  │ 🟡 placeholder · no info ││
│  │ Phone: [____________]    ││
│  │ [Create Personal Link]   ││
│  ├──────────────────────────┤│
│  │ Carol                    ││
│  │ 🟡 placeholder · no info ││
│  │ Phone: [____________]    ││
│  │ [Create Personal Link]   ││
│  ├──────────────────────────┤│
│  │ Dave                     ││
│  │ 🟡 placeholder · no info ││
│  │ Phone: [____________]    ││
│  │ [Create Personal Link]   ││
│  └──────────────────────────┘│
│                              │
│  Share Options               │
│  ┌──────────────────────────┐│
│  │  [📋 Copy Group Link]    ││
│  │  [📤 Share via...]       ││
│  │  [🔗 Create All Links]   ││
│  └──────────────────────────┘│
│                              │
│  Active Codes                │
│  ┌──────────────────────────┐│
│  │ General · 4 uses left    ││
│  │ For Dave · unused        ││
│  └──────────────────────────┘│
└──────────────────────────────┘
```

## Wireframe — Person Row States

Each person in the checklist can be in one of these states:

```
│  Bob                           │  ← Placeholder, no contact info
│  🟡 placeholder · no info      │
│  Phone: [____________]         │
│  [Create Personal Link]        │

│  Carol                         │  ← Placeholder, phone added
│  🟡 placeholder · 0412-xxx    │
│  [Create Personal Link]        │

│  Dave                          │  ← Personal link created
│  🟡 placeholder · link sent   │
│  fairgo.app/join/Dk9x2q   📋  │

│  Eve                           │  ← Resolved (joined via link)
│  🟢 active · claimed           │
│  ✓ Joined 2 hours ago         │
```

## Invite Usage Tracking (JF-4A)

The admin side displays usage tracking for each invite code. The `GET /events/{eid}/invite-codes` response includes usage data per code:

```
Active Codes
┌──────────────────────────────────────┐
│ General · 4 uses left                │
│   Used by: Eve, Frank                │
│   Created: 2 hours ago               │
├──────────────────────────────────────┤
│ For Dave · unused                    │
│   Target: Dave (placeholder)         │
│   Created: 1 hour ago                │
├──────────────────────────────────────┤
│ For Carol · claimed ✓                │
│   Used by: Carol · 30 min ago        │
└──────────────────────────────────────┘
```

Each code row shows:
- **Use count** — how many times the code has been redeemed vs max uses
- **Claimed by** — list of users who have joined using this code
- **Status** — unused / partially used / fully used / expired

The invite code list endpoint returns `usage_count`, `max_uses`, and a `claims[]` array with user display names and timestamps.

## Wireframe — Redeem Side (Deep Link)

Not a screen the user navigates to directly — handled by the app router when opening an invite URL.

```
fairgo.app/join/X7kQ2m
    │
    ├── Not logged in → S02 (register, code in session)
    │
    └── Logged in → auto-join flow:
        1. POST /events/join
        2. → S05 (event dashboard)
```

## Orchestration — Page Load (Prepare & Share)

```
1. GET /events/{eid}/persons         → person list (for checklist)
2. GET /events/{eid}/invite-codes    → list of active codes
```

## Orchestration — "Copy Group Link"

```
1. Copy default invite URL to clipboard
2. Show "Copied!" toast
```

No backend call — the invite code was already created at event creation time.

## Orchestration — "Share via..."

```
1. Trigger Web Share API / native share sheet with:
   - title: event name
   - text: "Join my event on FairGo"
   - url: default invite URL
2. Falls back to clipboard copy if share API not available
```

No backend call — uses the existing default invite code.

## Orchestration — "Create Personal Link" (Single Person)

```
1. POST /events/{eid}/invite-codes
     {target_person_id: selected_person_id, max_uses: 1}
   → person-targeted invite code created
2. UI generates shareable link: fairgo.app/join/{code}
3. Copy to clipboard + show "Link created!" toast
4. Person row updates to show the link
```

## Orchestration — "Create All Personal Links" (Bulk)

```
1. For each unresolved placeholder person (in parallel):
   POST /events/{eid}/invite-codes
     {target_person_id: person_id, max_uses: 1}
2. UI generates shareable links for each person
3. Show summary: "3 personal links created"
4. All person rows update to show their links
```

For V1, this fires N parallel requests (acceptable for events with <50 participants). A bulk endpoint can be added later if needed.

## Orchestration — "Add Phone" (Inline)

```
1. User enters phone number in the inline field
2. PUT /events/{eid}/persons/{pid}
     {phone_hint: "+61412345678"}
   → person phone_hint updated
3. Show confirmation: "Phone saved"
```

Uses the existing `phone_hint` field on the person model. No backend changes needed.

## Orchestration — Redeem (Logged-In User)

```
1. POST /events/join {invite_code: "X7kQ2m"}
   Response includes:
   - event_role (status: pending_approval or active)
   - person (auto-created or auto-claimed if target_person_id)
   - potential_matches (if identity matches detected)
2. If target_person_id set → auto-claim, no new person
3. If generic code → new person auto-created
4. If potential_matches found → show merge prompt on S07
5. → S05 (event dashboard)
```

## Invite Paths

### Path A: Group Link (Most Common)

Admin copies group link from S06 → shares externally → invitee opens link → S02 → S05.

### Path B: Personal Invite (Identity Resolution)

Admin creates personal invite on S06 for a placeholder person → invitee opens link → S02 → auto-claim (no duplicate person) → S05.

### Path C: Direct Code Entry

Existing user on S03 → "Join with Code" → enters code → S05.

### Path D: Person-Targeted Auto-Claim (NUP)

Admin creates an invite linked to a specific person record (`target_person_id`). When the invitee opens the link:

```
Admin on S06:
  [Create Personal Link] for "Dave" (placeholder)
    → POST /events/{eid}/invite-codes
        {target_person_id: dave_pid, max_uses: 1}
    → code tied to Dave's person record

Invitee (Dave) opens link:
  fairgo.app/join/Dk9x2q
    │
    ├── Not logged in → S02 (register, code in session)
    │     → POST /auth/register
    │     → POST /events/join {invite_code: "Dk9x2q"}
    │
    └── Logged in → POST /events/join {invite_code: "Dk9x2q"}

  Join response:
    → target_person_id detected
    → user AUTO-CLAIMED as Dave (no manual merge needed)
    → resolution_status: claimed
    → event_role created (status per event approval settings)
    → S05 Event Dashboard
```

Path D eliminates the manual merge step entirely. The invite code carries the identity mapping, so the system knows exactly which placeholder person the new user should become.

## Sponsorship on Join (NUP)

When a user joins an event via invite, the event admin (or inviter) can **sponsor** the new user:

- **Sponsorship** means the sponsor's paid tier covers the new user's participation in this event
- The joining user does not need to take any action — sponsorship is applied automatically during the join flow
- The sponsor must have an active paid tier with available sponsorship slots

```
Join flow with sponsorship:
  POST /events/join {invite_code: "X7kQ2m"}
    → system checks: does inviter/admin have sponsorship enabled?
    → if yes: new user's participation covered by sponsor's tier
    → new user gets paid-tier benefits without their own subscription
    → no additional UI step for the joining user
```

Sponsorship status is visible on S05 (event dashboard) and S07 (manage people) for the admin.

## Smart Defaults

- **Person checklist** shows only people the admin needs to act on (placeholders without links)
- **Resolved persons** (already joined) shown with a checkmark — no action needed
- General invite auto-created at event creation — admin just copies the link
- Person-targeted invite dropdown only shows placeholder persons (not already-claimed)
- Personal invites auto-resolve identity on join — no manual merge needed
- Invite codes default to unlimited uses unless creating a personal invite (max_uses: 1)
- **Phone field** appears inline — no separate screen needed for adding contact info
- **"Share via..."** uses native share sheet when available, clipboard fallback otherwise
- **"Create All Links"** only appears when there are unresolved placeholders

## Error States

| Error | Display |
|-------|---------|
| Invalid/expired code | "This invite link isn't valid or has expired" |
| Already a member | "You're already in this event" → navigate to S05 |
| Event is closed | "This event is no longer accepting new members" |
| Max uses reached | "This invite link has been used too many times" |
| Phone format invalid | Inline: "Enter a valid phone number" |
