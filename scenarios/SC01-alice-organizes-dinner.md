# SC01 — Alice Organizes a Dinner

**Persona:** First-time user, becomes event admin.
**Screens:** S01 → S02 → S03 → S04 → S05 → S09 → S05 → S07 → S05 → S06
**Rails:** R01 (Onboarding), R02 (Expense), R04 (Invitation)

## Design Principle

**Capture first, share last.** Alice records what happened (expenses), adds who was there (people), then shares the event when everything is ready. Expenses and people are equally available from S05 in either order, with expense as the prominent first option.

## Flow Diagram

```
S01 → S02 → S03 → S04 → S05 (Event Dashboard)
                                │
                    ┌───────────┴───────────┐
                    ↓                       ↓
              S09 Add Expense         S07 Add People
              (split-pending)         (inline split preview)
                    │                       │
                    └───────────┬───────────┘
                                ↓
                          S05 (Dashboard)
                                ↓
                          S06 Prepare & Share
                          (person checklist → share actions)
```

## Narrative

Alice downloads the app and sees the welcome screen.

```
S01 Welcome → taps "Get Started"
S02 Register → creates account (display name: "Alice", email, password)
    ⚡ POST /auth/register → tokens stored
S03 Home → empty state ("No events yet!")
    → taps "+ New Event"
S04 Create Event → types "Friday Dinner"
    ⚡ POST /events → event created, Alice auto-added as admin + person
    ⚡ POST /events/{eid}/invite-codes → default invite code generated (NOT shared yet)
S05 Event Dashboard → Alice sees her event, 1 person, 0 transactions
    → taps "+ Add Expense" FAB
```

Alice enters the expense right away — no need to add people first:

```
S09 Add Expense → "Pizza" / $45.00
    → No splits assigned (split-pending state)
    → Indicator: "Splits will be assigned when you add people"
    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Pizza", splits_status: "pending",
        line_items: [{amount: 45.00, splits: []}]}
S05 Event Dashboard → "Pizza $45 — splits pending ⏳"
    → taps "People"
```

Now Alice adds people — splits are auto-assigned as each person is added:

```
S07 Manage People → adds Bob
    ⚡ POST /events/{eid}/persons {display_name: "Bob"}
       → person created + auto-assigned to pending transactions
    → inline: "Pizza $45 → $22.50 each (Alice, Bob)"

    → adds Carol
    ⚡ POST /events/{eid}/persons {display_name: "Carol"}
       → auto-assigned
    → inline: "Pizza $45 → $15.00 each (Alice, Bob, Carol)"

    → adds Dave
    ⚡ POST /events/{eid}/persons {display_name: "Dave"}
       → auto-assigned
    → inline: "Pizza $45 → $11.25 each (Alice, Bob, Carol, Dave)"

    → back to S05
```

Dashboard now shows balances. Alice is ready to share:

```
S05 Event Dashboard
    → "Bob owes $11.25, Carol owes $11.25, Dave owes $11.25"
    → taps "Share Event →"
S06 Prepare & Share
    → Person checklist:
      Bob — placeholder, no contact info [Add phone / Create personal link]
      Carol — placeholder, no contact info [Add phone / Create personal link]
      Dave — placeholder, no contact info [Add phone / Create personal link]
    → Share options:
      [Copy Group Link] — clipboard (always available)
      [Share via...] — native share sheet (when available)
      [Create Personal Links] — generates per-person invite codes
```

## Validates

- Full onboarding flow: S01 → S02 → S03 → S04 → S05
- Auto-creation of invite codes on event creation (not shared until S06)
- **Expense entry without people** — split-pending state allows capturing costs immediately
- **Auto-assign splits on person add** — each person added recalculates all pending splits
- **Prepare & Share as final step** — Alice confirms identities and chooses share method
- Aggressive defaults: 2 fields + 1 tap for simple expense (when people exist)
- Single API call creates transaction + line item (splits deferred when pending)
- Person creation returns affected transactions with updated splits

## What Changed (from previous flow)

| Before | After | Why |
|--------|-------|-----|
| Alice copies invite link right after creating event | Invite link shared at end via S06 Prepare & Share | People should be attached before sharing |
| S09 blocks with "Add people first" when ≤1 person | S09 allows saving with no splits (split-pending) | Expenses are front-of-mind; capture them first |
| No pre-share preparation step | S06 expanded with person checklist + share options | Alice should confirm identities before sending invites |
| People must be added before expenses | Either order works; expense is the prominent first option | Flexibility — different admins think differently |
