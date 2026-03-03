# S15 — Profile & Settings

**Purpose:** User account management.
**Visible to:** Authenticated users.
**Rails:** R05 (Membership & Funding)
**Scenarios:** SC09

## Wireframe

```
┌──────────────────────────────┐
│  ← Profile                   │
├──────────────────────────────┤
│                              │
│  Alice Johnson               │
│  alice@example.com           │
│                              │
│  ┌──────────────────────────┐│
│  │ Display Name             ││
│  │ [Alice Johnson_________] ││
│  │ [Save Changes]           ││
│  └──────────────────────────┘│
│                              │
│  My Persons Across Events    │
│  ┌──────────────────────────┐│
│  │ Alice · Bali Trip    ▸   ││
│  │ Alice · Friday Dinner ▸  ││
│  │ Alice · House Bills   ▸  ││
│  └──────────────────────────┘│
│                              │
│  ┌──────────────────────────┐│
│  │ Membership            ▸  ││
│  │ Change Password       ▸  ││
│  │ Log Out                  ││
│  │ Delete Account            ││
│  └──────────────────────────┘│
│                              │
│  [Events] [Activity] [Me]   │
└──────────────────────────────┘
```

## Orchestration — Page Load

```
1. GET /users/me           → user profile (display_name, email)
2. GET /users/me/persons   → all persons across events (with event names)
```

## Orchestration — "Save Changes"

```
PUT /users/me {display_name}
→ profile updated
```

## Orchestration — "Change Password"

Opens inline or modal form:

```
1. Enter current password + new password + confirm
2. POST /auth/password-reset/confirm {current_password, new_password}
   (or dedicated change-password endpoint if available)
```

## Orchestration — "Log Out"

```
1. POST /auth/logout
2. Clear stored tokens
3. → S01 (Welcome)
```

## Orchestration — "Delete Account"

```
1. Confirm dialog: "Delete your account? This cannot be undone.
   Your financial data in events will be preserved but unlinked."
2. DELETE /users/me
3. Clear stored tokens
4. → S01 (Welcome)
```

## Actions

| Action | Target | Notes |
|--------|--------|-------|
| Person row tap | → S05 (that event's dashboard) | Quick navigation to any event |
| Membership ▸ | → S13 | Tier management |
| Change Password ▸ | Inline/modal | Password change form |
| Log Out | → S01 | Clear session |
| Delete Account | → S01 | Destructive, requires confirmation |

## Smart Defaults

- Display name pre-filled from current profile
- Persons listed with event name for context
- Person rows are tappable — quick way to jump to an event
- Delete Account is visually de-emphasised (text link, not prominent button)
- Log Out doesn't require confirmation

## Error States

| Error | Display |
|-------|---------|
| Display name empty | Inline: "Display name can't be empty" |
| Wrong current password | "Current password is incorrect" |
| Delete account with admin events | Warning: "You're admin of X events. Transfer admin rights first, or those events will have no admin." |
