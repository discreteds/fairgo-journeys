# R04 — Invitation Rail

**Purpose:** Admin generates invite, recipient joins event.
**Primary persona:** Admin (send side), new/existing user (receive side).

## Rail Path — Two Parallel Tracks

```
SEND SIDE (Admin)               RECEIVE SIDE (Invitee)
─────────────────               ──────────────────────

S05 Event Dashboard             Opens invite link
  │ "Invite Link 📋"               │
  │ or "People ▸" →                │
  │ "Send Personal Invite"         │
  ▼                                ▼
S06 Share Invite               S01 Welcome (deep link)
  │ [Copy Link]                    │
  │ [Create Personal Link]         ▼
  │                            S02 Register / Login
  │                                │
  │                                ├── (auto-join via session code)
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
| S05 → S06 | Admin wants to invite | Generate side |
| S06 → S07 | Personal invite needs person selection | Detour to people list |
| S02 → S05 | Invitee registers with code | Auto-join, pending approval |
| S05 (admin) → S05 (invitee) | Admin approves | Invitee gains full access |

## Three Invite Paths

### Path A: Generic Invite (Most Common)
```
Admin copies link from S05 → shares externally → invitee opens link → S02 → S05
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

## Scenarios Using This Rail

- SC02 (Bob Joins via Invite) — generic invite, full flow
- SC03 (Family Holiday) — people added as placeholders, personal invites sent later
