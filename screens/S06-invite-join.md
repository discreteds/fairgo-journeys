# S06 — Invite & Join

**Purpose:** Two-sided screen. Admin generates/shares invite codes. New user redeems code.
**Visible to:** Admin (generate side) / Unauthenticated or authenticated (redeem side).
**Rails:** R04 (Invitation)
**Scenarios:** SC02

## Wireframe — Generate Side (Admin)

Reached from S05 or S07 ("Send Personal Invite").

```
┌──────────────────────────────┐
│  ← Share Invite              │
├──────────────────────────────┤
│                              │
│  Invite Link                 │
│  ┌──────────────────────────┐│
│  │ fairgo.app/join/X7kQ2m  ││
│  │        [Copy Link]       ││
│  └──────────────────────────┘│
│                              │
│  Or send a personal invite   │
│  ┌──────────────────────────┐│
│  │ For: [Select person ▾]   ││
│  │ (placeholder persons     ││
│  │  only)                   ││
│  │                          ││
│  │ This link will auto-     ││
│  │ connect their account    ││
│  │ to their placeholder.    ││
│  │        [Create Link]     ││
│  └──────────────────────────┘│
│                              │
│  Active Codes                │
│  ┌──────────────────────────┐│
│  │ General · 4 uses left    ││
│  │ For Dave · unused        ││
│  └──────────────────────────┘│
└──────────────────────────────┘
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

## Orchestration — Page Load (Generate Side)

```
1. GET /events/{eid}/invite-codes    → list of active codes
```

## Orchestration — "Copy Link"

```
1. Copy default invite URL to clipboard
2. Show "Copied!" toast
```

No backend call — the invite code was already created at event creation time.

## Orchestration — "Create Personal Link"

```
1. POST /events/{eid}/invite-codes
     {target_person_id: selected_person_id, max_uses: 1}
   → person-targeted invite code created
2. UI generates shareable link: fairgo.app/join/{code}
3. Copy to clipboard or share sheet
```

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

- General invite auto-created at event creation — admin just copies the link
- Person-targeted invite dropdown only shows placeholder persons (not already-claimed)
- Personal invites auto-resolve identity on join — no manual merge needed
- Invite codes default to unlimited uses unless creating a personal invite (max_uses: 1)

## Error States

| Error | Display |
|-------|---------|
| Invalid/expired code | "This invite link isn't valid or has expired" |
| Already a member | "You're already in this event" → navigate to S05 |
| Event is closed | "This event is no longer accepting new members" |
| Max uses reached | "This invite link has been used too many times" |
