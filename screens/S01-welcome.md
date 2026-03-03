# S01 — Welcome

**Purpose:** First-time landing. Brand, value proposition, call to action.
**Visible to:** Unauthenticated users only.
**Rails:** R01 (Onboarding)
**Scenarios:** SC01, SC02, SC07

## Wireframe

```
┌─────────────────────────┐
│                         │
│      Fair Go            │
│   Split expenses,       │
│   not friendships       │
│                         │
│  ┌───────────────────┐  │
│  │   Get Started     │  │
│  └───────────────────┘  │
│  ┌───────────────────┐  │
│  │   I have a code   │  │
│  └───────────────────┘  │
│                         │
│   Already have an       │
│   account? Log in       │
└─────────────────────────┘
```

## Actions

| Action | Target | Orchestration |
|--------|--------|--------------|
| Get Started | → S02 (register tab) | Navigation only |
| I have a code | → S02 (register tab, invite code stored in session) | Navigation only |
| Log in | → S02 (login tab) | Navigation only |

## Backend Calls

None — pure navigation.

## Smart Defaults

- If user arrives via deep link (`fairgo.app/join/X7kQ2m`), bypass this screen and go directly to S02 with invite code in session.

## States

| State | Behaviour |
|-------|-----------|
| Authenticated user visits | Redirect to S03 (Home) |
| Deep link with invite code | Redirect to S02 with code pre-loaded |
