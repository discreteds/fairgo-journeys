# R04 — Invitation Rail

**Purpose:** Admin prepares and shares invites, recipient joins event.
**Primary persona:** Admin (prepare & share side), new/existing user (receive side).

## Rail Path — Two Parallel Tracks

```
SEND SIDE (Admin)               RECEIVE SIDE (Invitee)
─────────────────               ──────────────────────

S05 Event Dashboard             Opens invite link
  │ "Share Event →"                 │
  │ or "People ▸" →                │
  │ "Send Personal Invite"         │
  ▼                                ▼
S06 Prepare & Share           S01 Welcome (deep link)
  │ Person checklist              │
  │ [Copy Group Link]             ▼
  │ [Share via...]            S02 Register / Login
  │ [Create Personal Link]        │
  │ [Create All Links]            ├── (auto-join via session code)
  │                                │   ⚡ POST /events/join
  │                                │
  │                                ▼
  │                            S05 Event Dashboard
  │                                │ (pending approval)
  │                                │
  ▼                                │
S05 Event Dashboard ◄─── admin approves ───┘
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

## Three Invite Paths

### Path A: Group Link (Most Common)
```
Admin copies group link from S06 → shares externally → invitee opens link → S02 → S05
```

### Path B: Personal Invite (Identity Resolution)
```
Admin creates personal invite on S06 for placeholder Dave
→ Dave opens link → S02 → auto-claim (no duplicate person created)
```

### Path C: Direct Code Entry
```
Existing user on S03 → "Join with Code" → enters code → S05
```

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
