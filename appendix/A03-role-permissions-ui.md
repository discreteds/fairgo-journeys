# A03 — Role Permissions UI Reference

What admin sees vs what member sees on every screen.

## Principle

> Same screens, different capabilities. Members see a streamlined experience.
> Admins see the same thing plus management controls. Never separate apps.

## Per-Screen Breakdown

### S05 — Event Dashboard

| Element | Member | Admin |
|---------|--------|-------|
| Summary stats (total, people, txns) | ✅ Visible | ✅ Visible |
| Your Balance card | ✅ Visible | ✅ Visible |
| Settle Up button | ✅ Visible (when owing) | ✅ Visible (when owing) |
| Transaction list | ✅ Visible | ✅ Visible |
| People / Groups / Settlements links | ✅ Visible | ✅ Visible |
| Invite Link copy | ✅ Visible | ✅ Visible |
| + Add Expense FAB | ✅ Visible | ✅ Visible |
| ⚠️ Pending actions banner | ❌ Hidden | ✅ Visible (when pending items exist) |
| Inline Approve/Reject | ❌ Hidden | ✅ Visible |
| ⚙️ Event Settings | ❌ Hidden | ✅ Visible |
| 💳 Funding Status | ❌ Hidden | ✅ Visible |

### S07 — Manage People

| Element | Member | Admin |
|---------|--------|-------|
| Person list with status/PFG | ✅ Visible | ✅ Visible |
| Settlement group colours | ✅ Visible | ✅ Visible |
| + Add Person button | ❌ Hidden | ✅ Visible |
| Person detail (tap) | Read-only (name, settlement group) | Full controls |
| "This is me" (self-merge) | ✅ Visible (own placeholder only) | N/A (use admin merge) |
| Merge suggestion | ❌ Hidden | ✅ Visible |
| Change Settlement Group | ❌ Hidden | ✅ Visible |
| Send Personal Invite | ❌ Hidden | ✅ Visible |
| Edit Name | ❌ Hidden | ✅ Visible |
| Remove Person | ❌ Hidden | ✅ Visible |

### S08 — Manage Groups

| Element | Member | Admin |
|---------|--------|-------|
| Group list (consumption + settlement) | ✅ Visible | ✅ Visible |
| Group member list | ✅ Visible | ✅ Visible |
| + New Group button | ❌ Hidden | ✅ Visible |
| Edit group members | ❌ Hidden | ✅ Visible (checkbox toggles) |
| Archive group | ❌ Hidden | ✅ Visible |

### S09 — Add Expense

| Element | Member | Admin |
|---------|--------|-------|
| Full expense form | ✅ Visible | ✅ Visible |
| All fields editable | ✅ Yes | ✅ Yes |
| Save Expense | ✅ Enabled | ✅ Enabled |

Both roles can add expenses. The difference is what happens after — member-created transactions may require admin approval (depending on event settings).

### S10 — Expense Detail

| Element | Member | Admin |
|---------|--------|-------|
| Transaction details | ✅ Visible | ✅ Visible |
| Line items + splits | ✅ Visible | ✅ Visible |
| Per-person breakdown | ✅ Visible | ✅ Visible |
| [Edit] button | ❌ Hidden | ✅ Visible |
| [Approve] button | ❌ Hidden | ✅ Visible (when pending) |
| [Request Changes] | ❌ Hidden | ✅ Visible (when pending) |
| [Cancel] button | ❌ Hidden | ✅ Visible |

### S11 — Balances

| Element | Member | Admin |
|---------|--------|-------|
| All balance views | ✅ Visible | ✅ Visible |
| Settle Up button | ✅ Visible (when owing) | ✅ Visible (when owing) |

No admin-specific elements. Balances are the same for everyone.

### S12 — Settle Up

| Element | Member | Admin |
|---------|--------|-------|
| Suggested settlements | ✅ Visible | ✅ Visible |
| Create Settlement | ✅ Enabled | ✅ Enabled |
| [Confirm] button | ❌ Hidden | ✅ Visible (on proposed settlements) |
| [Void] button | ❌ Hidden | ✅ Visible (on proposed/confirmed) |
| [Mark as Paid] button | ✅ Own PFG only | ✅ Any settlement |

### S14 — Event Funding

| Element | Member | Admin |
|---------|--------|-------|
| Entire screen | ❌ Not reachable | ✅ Full access |

Funding is admin-only. Members never see this screen. If a member hits a funding limit (e.g. on S09), they see "Contact your event admin."

### S16 — Admin Moderation

| Element | Member | Admin |
|---------|--------|-------|
| Entire screen | ❌ Not reachable | ✅ Full access |

## Visual Differentiation

The UI should use subtle cues to communicate role:

| Cue | Meaning |
|-----|---------|
| ⚙️ gear icon on S05 | Admin controls available |
| ⚠️ pending badge | Admin has items to review |
| + Add buttons (people, groups) | Admin creation powers |
| [Approve] / [Reject] buttons | Admin moderation powers |
| No extra controls | Member view — clean, focused |

## Permission Enforcement

UI permissions are **optimistic** — the UI hides elements based on role, but the backend enforces permissions on every API call. If a member somehow sends an admin-only request, the backend returns 403 Forbidden.

```
UI layer:   Hide elements the user can't use (UX)
API layer:  Reject unauthorised requests (security)
```

Never rely on UI hiding alone for security.
