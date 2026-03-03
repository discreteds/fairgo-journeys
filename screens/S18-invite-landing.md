# S18 — Invite Landing

**Purpose:** Show a new user their balance and event summary before asking them to create an account. The critical growth-loop screen — convert a curious visitor into a registered user by showing value first.
**Visible to:** Unauthenticated users arriving via a personal invite link.
**Rails:** R04 (Invitation)
**Scenarios:** SC02

---

## Overview

When Bob taps Alice's personal invite link (`https://fairgo.app/join/abc123`), the app router checks his authentication state. He is not logged in, so he is shown this screen instead of being redirected straight to S02.

The design principle is **show value before asking for commitment.** Bob can see exactly what he owes and why without creating an account. The account creation barrier is placed at the point where Bob wants to *act* (pay, dispute, review detail) — not at the point where he first arrives.

This screen is read-only. No writes happen until Bob taps a CTA and completes registration.

---

## Wireframe — Default State (Valid Link, Not Logged In)

```
┌─────────────────────────────────┐
│                                 │
│         fair go                 │
│                                 │
├─────────────────────────────────┤
│                                 │
│  Friday Dinner                  │
│  Alice invited you              │
│                                 │
│  ┌─────────────────────────────┐│
│  │  Your share                 ││
│  │                             ││
│  │         $43.50              ││
│  │                             ││
│  │  Food    $17.50             ││
│  │  split between 4 people     ││
│  │                             ││
│  │  Wine    $26.00             ││
│  │  split between 3 people     ││
│  └─────────────────────────────┘│
│                                 │
│  ┌─────────────────────────────┐│
│  │  Create Account to Pay      ││  ← Primary CTA
│  └─────────────────────────────┘│
│                                 │
│  ┌─────────────────────────────┐│
│  │  View Full Event            ││  ← Secondary CTA
│  └─────────────────────────────┘│
│                                 │
│  No account needed to view.     │
│  Account required to pay or     │
│  dispute.                       │
│                                 │
└─────────────────────────────────┘
```

---

## Wireframe — Line-Item Breakdown Detail

Each line item shows the description, the invitee's calculated share, and a brief split summary. The full payer name and per-person calculation are shown so Bob understands where the numbers come from.

```
┌─────────────────────────────────┐
│  Your share                     │
│                                 │
│  ┌──────────────────────────┐   │
│  │ Food                     │   │
│  │ Your share: $17.50       │   │
│  │ Alice paid $70.00        │   │
│  │ Split 4 ways equally     │   │
│  ├──────────────────────────┤   │
│  │ Wine                     │   │
│  │ Your share: $26.00       │   │
│  │ Alice paid $78.00        │   │
│  │ Split 3 ways equally     │   │
│  └──────────────────────────┘   │
│                                 │
│  Total you owe: $43.50          │
└─────────────────────────────────┘
```

---

## Component Table

| Component | Type | Behaviour |
|-----------|------|-----------|
| App wordmark | Static | "fair go" logo — no navigation |
| Event name | Heading | Pulled from invite-code response (`event.name`) |
| Inviter attribution | Subheading | "Alice invited you" — pulled from `invited_by.display_name` |
| Share card | Card | Prominent dollar amount (`your_share_total`), colour-coded: red if owes, green if owed |
| Line-item list | List | One row per line item: description, `your_share`, payer name, split summary |
| Create Account CTA | Primary button | Stores invite code in session, navigates to S02 (Register tab) |
| View Full Event CTA | Secondary button | Navigates to S05 in read-only mode (no payment or dispute actions available) |
| Disclaimer text | Caption | Explains account is not required to view, but is required to pay or dispute |

---

## State Variants

| State | Trigger | Display |
|-------|---------|---------|
| **Default** | Valid code, user not logged in | Full wireframe as above — share amount, line items, both CTAs |
| **Expired** | `GET /invite-codes/{code}` returns `status: expired` | "This invite link has expired. Ask Alice to send you a new one." No CTAs. |
| **Claimed** | Code already redeemed by a different account | "This invite link has already been used. If you think this is wrong, contact Alice." No CTAs. |
| **Invalid** | Code not found (404) | "This invite link isn't valid. Check the link and try again." No CTAs. |
| **Logged in** | User has a valid session when they tap the link | Skip S18 entirely — router auto-joins and redirects to S05 (see Logged-In Path below) |
| **No share calculated** | Invite is a general (non-person-targeted) code | Show event name and inviter, but omit share card — show "Create Account to join this event" CTA only |

---

## Logged-In Path (Skip S18)

If the user is already authenticated when they open the invite URL, S18 is never shown. The app router handles the join flow directly:

```
Authenticated user opens fairgo.app/join/abc123
    │
    └── POST /events/join {invite_code: "abc123"}
        → event_role created (or existing role detected)
        → S05 Event Dashboard
```

This path is handled by the router, not by S18. S18 is exclusively for unauthenticated visitors.

---

## API Orchestration — Page Load

```
1. GET /invite-codes/{code}
   Response includes:
   - status: valid | expired | claimed | invalid
   - event.name
   - event.description (if set)
   - invited_by.display_name
   - target_person_id (if personal invite)
   - your_share_total (pre-calculated for target person)
   - line_items[] each with:
       - description
       - your_share (calculated for target person)
       - payer_display_name
       - total_amount
       - split_summary (e.g., "split 4 ways equally")

No authentication required for this call.
The backend calculates your_share only if target_person_id is set on the code.
General (non-person-targeted) codes return no share data.
```

This is the only API call on S18. The screen is read-only.

---

## API Orchestration — "Create Account to Pay" CTA

```
1. Store invite code in session (localStorage: pending_invite_code)
2. Navigate to S02 (Register tab)

On S02 — after successful registration:
3. POST /auth/register {display_name, email, password}
   → access_token + refresh_token stored
4. POST /invite-codes/{code}/claim
   → target_person_id detected (personal invite)
   → Bob's new USER linked to his placeholder PERSON record
   → event_role created: member (status: active — personal invites pre-approved)
5. → S05 Event Dashboard
   → Bob sees Friday Dinner with his balance: owes $43.50
   → his person record is now resolved (not a placeholder)
```

The claim step is what resolves Bob's identity — connecting his new user account to the placeholder person Alice created when she entered his name into the event. After this, Bob's expenses, splits, and balance are all correctly attributed to him.

---

## API Orchestration — "View Full Event" CTA (No Account)

```
1. Navigate to S05 Event Dashboard with read-only flag
   → GET /events/{eid} (unauthenticated, public read if event is shareable)
   → Event displayed in read-only mode
   → "Pay" and "Dispute" actions hidden
   → Banner: "Create an account to pay or dispute your share"
```

This path allows Bob to browse the event without committing to registration. If he later taps the banner, the invite code is still in the URL and the normal CTA flow applies.

---

## Navigation

### What leads to S18

| Source | How |
|--------|-----|
| Personal invite link (SMS / WhatsApp / email) | `fairgo.app/join/{code}` — router sends unauthenticated users here |
| General group invite link | `fairgo.app/join/{code}` — same entry point; S18 shown without share data |

### Where S18 leads

| Action | Destination | Notes |
|--------|-------------|-------|
| "Create Account to Pay" | S02 (Register tab) | Invite code stored in session; claim fires after registration |
| "View Full Event" | S05 (read-only) | No account created; actions disabled |
| After registration + claim | S05 (full access) | Bob is now a member with resolved identity |
| Expired / invalid / claimed error state | Dead end | User prompted to contact inviter |

---

## Smart Defaults

- **Share card is the hero element** — the dollar amount is the largest text on screen. It's what Bob came to see.
- **Line-item breakdown visible immediately** — no tap required to expand. Transparency is the default.
- **Payer name shown on each line item** — "Alice paid $78.00" gives context for the charge without Bob needing to navigate anywhere.
- **"Create Account" is the primary CTA** — primary button styling, positioned above "View Full Event".
- **"View Full Event" is low-commitment** — secondary styling signals it is a safe, non-committal action.
- **Invite code preserved through registration** — stored in session before navigating to S02 so it is not lost even if the user switches apps, checks a message, and returns.
- **Personal invites pre-approve membership** — Bob does not see an "awaiting approval" banner after joining via a personal link. The personal invite signals admin intent; approval is implicit.
- **No Fair Go marketing on this screen** — the screen is about Bob's split, not about selling the product. Trust comes from accuracy, not from brand copy.

---

## Error States

| Error | Display |
|-------|---------|
| Code expired | "This invite link has expired. Ask Alice to send you a new one." |
| Code already claimed | "This invite link has already been used. Contact Alice if you need a new one." |
| Code not found (404) | "This invite link isn't valid. Check the link and try again." |
| Network failure on load | "Couldn't load your invite. Check your connection and try again." with retry button |
| Registration fails after CTA tap | Error shown on S02; invite code remains in session for retry |
| Claim fails after registration | Registration succeeds; error shown on S05 with a "Retry joining" prompt |

---

## Cross-References

- **SC02** — Bob Joins and Disputes: the primary scenario using this screen. Bob arrives here from Alice's personal link, sees his $43.50 share, taps "Create Account", registers, and proceeds to dispute a line item.
- **S02** — Register / Login: destination after "Create Account to Pay". Invite code flows through to the join and claim orchestration.
- **S05** — Event Dashboard: destination after successful claim, and also reachable directly via "View Full Event" in read-only mode.
- **S06** — Prepare & Share: where Alice generates the personal invite link that routes Bob to this screen. S06 is the producer; S18 is the consumer.
- **R04** — Invitation Rail: S18 is the entry point of this rail for unauthenticated recipients.
