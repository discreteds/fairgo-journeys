# S06 — Prepare & Share (Invite & Join)

**Purpose:** Two-sided screen. Admin prepares person checklist and shares invite codes. New user redeems code.
**Visible to:** Admin (generate side) / Unauthenticated or authenticated (redeem side).
**Rails:** R04 (Invitation)
**Scenarios:** SC01 (prepare & share), SC02 (join)

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
