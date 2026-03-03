# SC02 — Bob Joins via Invite

> **Replaces:** Original SC02 (simple join flow). Rework adds dispute flow and differentiator demonstration.

**Persona:** Bob — a non-admin recipient. Receives a personal invite link from Alice, sees his balance before creating an account, reviews his split, and challenges a line item.
**Screens:** S18 (Invite Landing) → S02 → S05 → S10 → S16 → S05
**Rails:** R04 (Invitation), R07 (Dispute)
**Prerequisite:** Follows directly from SC01-R. Alice has created Friday Dinner, entered two expenses, and shared Bob's personal invite link with him.

## The Situation

Bob gets a text from Alice: "Here's the split for Friday dinner." He taps the link. He doesn't have a Fair Go account yet. He lands on a screen showing exactly what he owes and why — before he's asked to create an account.

He owes $30. He thinks the wine charge is wrong: he had one glass, Alice had two. He disputes the wine line item. Alice adjusts the weights. Bob now owes $25.

This is the **most important scenario in the entire product** from a growth perspective. Every new user who isn't the admin starts here. If this experience is confusing, slow, or feels untrustworthy, the growth loop dies.

## Event Setup (from SC01-R)

Alice has entered two expenses for Friday Dinner:

| Line Item | Amount | Paid By | Split Between | Method |
|-----------|--------|---------|---------------|--------|
| Food | $45.00 | Alice | Alice, Bob, Carol | Equal 3-way |
| Wine | $30.00 | Alice | Alice, Bob | Equal 2-way |

**Before dispute — expected balances:**
| Person | Food | Wine | Total Consumed | Paid | Net |
|--------|------|------|----------------|------|-----|
| Alice | $15.00 | $15.00 | $30.00 | $75.00 | **+$45.00** (owed) |
| Bob | $15.00 | $15.00 | $30.00 | $0.00 | **−$30.00** (owes) |
| Carol | $15.00 | $0.00 | $15.00 | $0.00 | **−$15.00** (owes) |
| **Σ** | **$45.00** | **$30.00** | **$75.00** | **$75.00** | **$0.00 ✓** |

## Narrative

```
Bob receives personal invite link via text from Alice

→ taps link

S18 Invite Landing
    ┌────────────────────────────────────────────────────────┐
    │  Friday Dinner                                         │
    │  Alice thinks you owe $30.00                           │
    │                                                        │
    │  Food    $15.00  (your share of $45.00 ÷ 3)           │
    │  Wine    $15.00  (your share of $30.00 ÷ 2)           │
    │                                                        │
    │  [Create Account to Review Details]                    │
    │  ─── or ───                                            │
    │  [Just Pay Alice →]                                    │
    └────────────────────────────────────────────────────────┘

    Bob sees his total and the line-item breakdown BEFORE creating
    an account. This is the critical trust moment — if he has to
    register before seeing anything, he'll bounce.

    Bob thinks $15 for wine is wrong — he only had one glass,
    Alice had two. He wants to review and dispute.

    → taps "Create Account to Review Details"

S02 Register
    → Bob enters: display name "Bob", email, password
    ⚡ POST /auth/register → tokens stored, free tier created
    ⚡ POST /events/{eid}/invite-codes/{code}/claim
       → Bob's placeholder PERSON (pre-created by Alice in SC01-R)
         is linked to his new USER account
       → resolution_status: claimed
       → Bob auto-assigned "member" EVENT_ROLE
       → personal link bypasses approval — Bob is immediately active
         (Alice pre-approved him by generating a personal link)

S05 Event Dashboard (Bob's view)
    → role: member (active — personal invite bypasses approval queue)
    → sees 2 expenses totalling $75.00
    → his balance: owes $30.00
    → taps "Wine $30.00" expense to review

S10 Expense Detail (Wine)
    ┌────────────────────────────────────────────────────────┐
    │  Wine              $30.00                              │
    │  Paid by: Alice                                        │
    │  Split between: Alice, Bob                             │
    │                                                        │
    │  Alice    $15.00  (weight 1)                           │
    │  Bob      $15.00  (weight 1)  ← you                   │
    │                                                        │
    │  [Looks Right ✓]  [Dispute This →]                    │
    └────────────────────────────────────────────────────────┘

    Bob thinks the equal split is wrong. He had one glass;
    Alice had two. A 1:2 weight ratio would give him $10,
    not $15.

    → taps "Dispute This →"

S16 Admin Moderation (Bob's submission view)
    → target: "Wine" line item (line_item_id: wine_li_id)
    → current split: Alice $15, Bob $15 (weights 1:1)
    → Bob's message: "I only had one glass of wine,
      Alice had two. Weights should be Alice 2, Bob 1."
    → suggested weights: Alice 2, Bob 1
    ⚡ POST /events/{eid}/modification-requests
       {target_type: "line_item_split",
        target_id: wine_li_id,
        message: "I only had one glass, Alice had two. Suggest weights Alice 2, Bob 1.",
        suggested_changes: {
          consumption_splits: [
            {person_id: alice_id, weight: 2},
            {person_id: bob_id,   weight: 1}
          ]
        }}
    → "Request sent to Alice ✓"
    → status: pending

Alice sees the notification:

Alice's S05 → "⚠️ 1 modification request"
    → taps to review
    ⚡ GET /events/{eid}/modification-requests

S16 Admin Moderation (Alice's review view)
    ┌────────────────────────────────────────────────────────┐
    │  Modification Request from Bob                         │
    │  "Wine" line item                                      │
    │                                                        │
    │  Bob's note: "I only had one glass, Alice had two.    │
    │  Suggest weights Alice 2, Bob 1."                      │
    │                                                        │
    │  Current:  Alice $15.00 (w:1), Bob $15.00 (w:1)       │
    │  Proposed: Alice $20.00 (w:2), Bob $10.00 (w:1)       │
    │                                                        │
    │  Impact on balances:                                   │
    │    Alice:  +$45.00 → +$40.00  (Alice owes $5 more)    │
    │    Bob:    −$30.00 → −$25.00  (Bob saves $5)           │
    │    Carol:  −$15.00 (unchanged)                         │
    │    Checksum: $0.00 ✓                                   │
    │                                                        │
    │  [Approve]   [Reject]                                  │
    └────────────────────────────────────────────────────────┘

    Alice agrees — she did have two glasses to Bob's one.

    → taps [Approve]
    ⚡ POST /events/{eid}/modification-requests/{rid}/approve
       → status: approved
       → proposed weights auto-applied:
           Alice: weight 2  →  $30.00 × 2/3 = $20.00
           Bob:   weight 1  →  $30.00 × 1/3 = $10.00
       → splits recalculated
    → Bob notified: "Your modification request was approved ✓"

S05 Event Dashboard (both users)
    → updated balances visible to all members
```

## Arithmetic Verification

**Wine line item recalculation after 2:1 weight adjustment:**

| Person | Weight | Share of $30.00 |
|--------|--------|-----------------|
| Alice | 2 | $30.00 × 2/3 = **$20.00** |
| Bob | 1 | $30.00 × 1/3 = **$10.00** |
| Total | 3 | **$30.00 ✓** |

**After dispute — final balances:**

| Person | Food | Wine (adjusted) | Total Consumed | Paid | Net |
|--------|------|-----------------|----------------|------|-----|
| Alice | $15.00 | $20.00 | $35.00 | $75.00 | **+$40.00** (owed) |
| Bob | $15.00 | $10.00 | $25.00 | $0.00 | **−$25.00** (owes) |
| Carol | $15.00 | $0.00 | $15.00 | $0.00 | **−$15.00** (owes) |
| **Σ** | **$45.00** | **$30.00** | **$75.00** | **$75.00** | **$0.00 ✓** |

## API Call Summary

| Step | Method | Endpoint | Purpose |
|------|--------|----------|---------|
| Register | `POST` | `/auth/register` | Create Bob's account |
| Claim person | `POST` | `/events/{eid}/invite-codes/{code}/claim` | Link Bob's USER to his placeholder PERSON |
| View expense | `GET` | `/events/{eid}/transactions/{tid}` | Load expense detail with line-item breakdown |
| Raise dispute | `POST` | `/events/{eid}/modification-requests` | Submit weight change request |
| List requests | `GET` | `/events/{eid}/modification-requests` | Alice's moderation queue |
| Approve | `POST` | `/events/{eid}/modification-requests/{rid}/approve` | Approve and auto-apply split change |
| Refresh balances | `GET` | `/events/{eid}/positions/persons` | Updated positions for all members |

## Validates

- **Recipient's first experience** — sees balance and line-item breakdown before creating account (S18)
- **Invite link → account creation → person claim** — the full onboarding-via-link flow
- **Personal invite bypasses approval queue** — Bob has immediate member access, no pending state
- **Expense detail view with per-person breakdown** — transparency builds trust before any dispute
- **Modification request end-to-end** — raise (Bob) → review (Alice) → approve → auto-apply
- **Non-equal weighted split on a single line item** — 2:1 ratio, not just include/exclude
- **Impact preview before approval** — Alice sees what changes before committing
- **Auto-apply on approval** — no separate PATCH call; approval triggers recalculation
- **Recalculated balances with verified checksum** — proves the system handles mid-stream corrections
- **Audit trail** — modification request records who asked, what was proposed, who approved
- **The "Just Pay Alice" escape hatch on S18** — some recipients just want to settle; account creation is optional

## Variant: Bob Withdraws Before Alice Acts

If Bob changes his mind after submitting:

```
Bob's S10 → sees pending modification request indicator
    → taps [Withdraw Request]
    ⚡ POST /events/{eid}/modification-requests/{rid}/withdraw
    → status: withdrawn
    → removed from Alice's moderation queue
    → splits unchanged
    → original $30 balance restored
```

## Variant: Alice Rejects the Request

```
Alice's S16 → reviews Bob's request
    → taps [Reject]
    → adds note: "You had two glasses too, I counted."
    ⚡ POST /events/{eid}/modification-requests/{rid}/reject
       {reason: "You had two glasses too, I counted."}
    → status: rejected
    → Bob notified: "Your modification request was rejected"
    → Bob sees Alice's reason on S10
    → splits unchanged — Bob still owes $30.00
```

## The Differentiator Moment

Bob can see exactly why he owes $30 — two line items, each with his specific share — before he even creates an account. And when he thinks the split is wrong, he doesn't text Alice privately and hope she believes him. He files a structured request with a reason and a proposed correction. Alice sees the impact before deciding. The approval creates an audit trail: who asked, what was proposed, who approved, when.

In Splitwise, this conversation happens in iMessage. Someone edits the split, the other person has no idea it changed, and there is no record of why. In Fair Go, the dispute is the record. The final numbers are provably correct, and everyone can see how they arrived there.

> **"Disputes are conversations, not conflicts"**
>
> Bob can't unilaterally change the split, but he can make a case.
> Alice sees the impact before deciding. The whole flow is transparent —
> proposed changes, preview, approve or reject with notes. Financial
> fairness requires communication, and the app facilitates it.
