# SC01 — Alice Organizes a Dinner

**Persona:** First-time user, becomes event admin.
**Screens:** S01 → S02 → S03 → S04 → S05 → S07 → S09
**Rails:** R01 (Onboarding), R02 (Expense)

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
    ⚡ POST /events/{eid}/invite-codes → default invite code generated
S05 Event Dashboard → Alice sees her event, 1 person, 0 transactions
    → copies invite link, sends to group chat
    → taps "+ Add Expense" FAB
```

Alice realises there's nobody to split with yet — the empty state on S09 redirects her:

```
S09 Add Expense → "Add people first" empty state
    → taps "Add People →"
S07 Manage People → adds Bob, Carol, Dave as placeholders
    ⚡ POST /events/{eid}/persons × 3 (each creates person + singleton PFG)
    → back to S05
```

Now Alice adds the expense:

```
S05 → taps "+ Add Expense" FAB
S09 Add Expense
    → "Pizza" / $45.00
    → Who paid: You (Alice) ← default
    → Split between: Everyone (4) ← default
    → Split method: Equal ← default
    → taps "Save Expense"
    ⚡ POST /events/{eid}/transactions
       {description: "Pizza", line_items: [{amount: 45, splits: [1 expense + 4 consumption]}]}
S05 Event Dashboard → "Bob owes $11.25, Carol owes $11.25, Dave owes $11.25"
```

## Validates

- Full onboarding flow: S01 → S02 → S03 → S04 → S05
- Auto-creation of invite codes on event creation
- Placeholder person creation by admin
- S09 empty state (no people yet) → redirect to S07
- Aggressive defaults: 2 fields + 1 tap for simple expense
- Single API call creates transaction + line item + all splits

## Gap Identified

S09 needs an empty-state handler when event has ≤1 person. Should either redirect to S07 or allow adding people inline.
