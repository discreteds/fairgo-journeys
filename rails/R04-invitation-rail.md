# R04 — Invitation Rail

**Purpose:** Admin prepares and shares invites, recipient joins event.
**Primary persona:** Admin (prepare & share side), new/existing user (receive side).

## Rail Path — Two Parallel Tracks

```
SEND SIDE (Admin)               RECEIVE SIDE (Invitee)
─────────────────               ──────────────────────

S05 Event Dashboard             Opens invite link
  │ "Share Event →"                 │
  │ or "People ▸" →                ├── (personal invite)
  │ "Send Personal Invite"         │        │
  ▼                                │        ▼
S06 Prepare & Share                │     S18 Invite Landing
  │ Person checklist               │        │ (balance preview)
  │ [Copy Group Link]              │        ▼
  │ [Share via...]                 │     S02 Register / Login
  │ [Create Personal Link]         │        │
  │ [Create All Links]             │        ▼
  │                                │     S05 Event Dashboard
  │                                │
  │                                ├── (group link)
  │                                │        │
  │                                │        ▼
  │                                │     S01 Welcome (deep link)
  │                                │        │
  │                                │        ▼
  │                                │     S02 Register / Login
  │                                │        │
  │                                │        ├── (auto-join via session code)
  │                                │        │   ⚡ POST /events/join
  │                                │        │
  │                                │        ▼
  │                                │     S05 Event Dashboard
  │                                │        │ (pending approval)
  │                                │        │
  ▼                                │        │
S05 Event Dashboard ◄─── admin approves ────┘
  "⚠️ Pending actions"
  [Approve] [Reject]
```

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S05 → S06 | Admin taps "Share Event →" | Prepare & Share |
| S07 → S06 | Admin taps "Send Personal Invite" for a person | Prepare & Share (pre-selected person) |
| S02 → S05 | Invitee registers with code | Auto-join, pending approval |
| S05 (admin) → S05 (invitee) | Admin approves | Invitee gains full access |

## Four Invite Paths

### Path A: Group Link (Most Common)
```
Admin copies group link from S06 → shares externally → invitee opens link → S02 → S05
```

### Path B: Personal Invite (Identity Resolution)
```
Admin creates personal invite on S06 for placeholder Dave
→ Dave opens link → S18 Invite Landing (balance preview) → S02 Register → S05 (auto-claim)
```

### Path C: Direct Code Entry
```
Existing user on S03 → "Join with Code" → enters code → S05
```

### Path D: Person-Targeted Auto-Claim (NUP)

Admin creates a person-targeted invite linked to a specific placeholder person. The invitee is **automatically claimed** as that person on join — no manual merge step needed.

```
S05 → S06 (admin creates person-targeted invite for "Dave")
         │
         │  POST /events/{eid}/invite-codes
         │    {target_person_id: dave_pid, max_uses: 1}
         │
         ▼
Dave receives link → opens fairgo.app/join/Dk9x2q
         │
         ▼
S18 Invite Landing (balance preview, event summary)
         │
         ▼
S02 Register / Login
         │
         ├── ⚡ POST /auth/register (or /auth/login)
         │
         ├── ⚡ POST /events/join {invite_code: "Dk9x2q"}
         │     → target_person_id detected
         │     → auto-claimed as "Dave" (no duplicate, no merge)
         │     → resolution_status: claimed
         │
         ▼
S05 Event Dashboard (auto-claimed as Dave)
```

Rail notation: `S06 → S02 (register) → S05 (auto-claimed)`

This path skips the manual person merge/claim step entirely. The invite code carries the identity mapping.

## Sponsorship on Join (NUP)

During the join flow (all paths), the system checks whether the inviter or event admin has **sponsorship** enabled:

```
POST /events/join
  │
  ├── Check: does inviter/admin have active sponsorship?
  │     │
  │     ├── YES → new user covered by sponsor's paid tier
  │     │         (no action needed from joining user)
  │     │
  │     └── NO → new user uses their own tier (free by default)
  │
  ▼
S05 Event Dashboard
```

- Sponsorship is automatic — no additional UI step for the joining user
- The sponsor's paid tier must have available sponsorship slots
- Sponsorship status visible to admin on S07 (Manage People)
- Applies to all invite paths (A, B, C, D)

## S06 Prepare & Share — Admin Flow

Before sharing, the admin sees a person-by-person checklist:

1. **Review people** — each placeholder person shows their resolution status
2. **Add contact info** — inline phone number field for future SMS feature
3. **Create personal links** — per-person or bulk creation of invite codes
4. **Share** — copy group link, use native share sheet, or share individual links

This ensures the admin has added people and confirmed identities before sending invites.

## Scenarios Using This Rail

- SC01 (Alice Organizes Dinner) — prepare & share as final step after expenses and people
- SC02 (Bob Joins via Invite) — generic invite, full flow
- SC03 (Family Holiday) — people added as placeholders, personal invites sent later
