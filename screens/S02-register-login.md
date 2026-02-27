# S02 — Register / Login

**Purpose:** Authentication. Single screen with tab toggle.
**Visible to:** Unauthenticated users.
**Rails:** R01 (Onboarding)
**Scenarios:** SC01, SC02

## Wireframe — Register Tab

```
┌─────────────────────────┐
│  [Register]  [Log In]   │
├─────────────────────────┤
│                         │
│  Display Name  [______] │
│  Email         [______] │
│  Password      [______] │
│                         │
│  ┌───────────────────┐  │
│  │   Create Account  │  │
│  └───────────────────┘  │
│                         │
│  Forgot password?       │
└─────────────────────────┘
```

## Wireframe — Login Tab

```
┌─────────────────────────┐
│  [Register]  [Log In]   │
├─────────────────────────┤
│                         │
│  Email         [______] │
│  Password      [______] │
│                         │
│  ┌───────────────────┐  │
│  │      Log In       │  │
│  └───────────────────┘  │
│                         │
│  Forgot password?       │
└─────────────────────────┘
```

## Orchestration — "Create Account"

```
1. POST /auth/register {display_name, email, password}
   → user created, access_token + refresh_token returned
2. Store tokens locally (localStorage or secure cookie)
3. If invite code in session:
   3a. POST /events/join {invite_code: "..."}
       → event_role created (status: pending_approval)
       → if target_person_id on invite → auto-claim placeholder
       → response may include potential_matches
   3b. → S05 (event dashboard, "awaiting approval" banner)
4. Else:
   → S03 (home)
```

## Orchestration — "Log In"

```
1. POST /auth/login {email, password}
   → access_token + refresh_token returned
2. Store tokens locally
3. If invite code in session:
   3a. POST /events/join {invite_code: "..."}
   3b. → S05
4. Else:
   → S03 (home)
```

## Orchestration — "Forgot Password"

```
1. POST /auth/password-reset/request {email}
   → reset email sent (always returns 200 to prevent enumeration)
2. User receives email with reset link
3. POST /auth/password-reset/confirm {token, new_password}
   → password updated
4. → S02 (login tab)
```

## Smart Defaults

- If user arrived via invite link, registration flows straight into the event after account creation — no extra steps
- Display name field pre-focused on register tab
- Email field pre-focused on login tab
- Tab defaults to Register for new visitors, Login for returning (detected via localStorage flag)

## Error States

| Error | Display |
|-------|---------|
| Email already registered | "An account with this email already exists. Log in instead?" with link to login tab |
| Invalid credentials | "Email or password is incorrect" |
| Weak password | Inline validation before submit |
| Network error | "Couldn't connect. Check your internet and try again." |
| Join event fails (after register) | Register succeeds, join error shown on S05 with retry option |
