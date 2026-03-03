# A02 — Smart Defaults Reference

Every place in the UI where we make an assumption on behalf of the user.

## Principle

> The common case should require zero configuration. Every default should be the thing
> most users would choose. Overrides exist but aren't prominent.

## Defaults by Screen

### S02 — Register / Login

| Context | Default | Override |
|---------|---------|----------|
| Tab selection (new visitor) | Register | Toggle to Login |
| Tab selection (returning) | Login | Toggle to Register |
| Invite code handling | Auto-join after auth if code in session | — |

### S03 — Home

| Context | Default | Override |
|---------|---------|----------|
| Event sort order | Most recent activity first | — (no override in MVP) |
| Position display | "You owe $X" / "You're owed $X" | — |
| Closed events | Shown at bottom, muted | — |

### S04 — Create Event

| Context | Default | Override |
|---------|---------|----------|
| Currency | AUD (locale-detected) | Dropdown |
| Description | Empty (optional) | Free text |
| Invite code | Auto-generated | Create additional codes on S06 |
| Creator's person | Auto-created from profile display_name | — |
| Creator's role | Admin | — |

### S07 — Manage People

| Context | Default | Override |
|---------|---------|----------|
| New person's PFG | Singleton (solo settlement group) | Change via "Change Settlement Group" |
| New person's status | Placeholder | Becomes auto_created/claimed on join |
| Email/phone hints | Not required | Optional fields on add |
| Person sort | Active first, then placeholders | — |

### S08 — Manage Groups

| Context | Default | Override |
|---------|---------|----------|
| "Everyone" group | Derived from all persons (implicit) | Create explicit groups |
| New group type | Non-singleton (consumption group) | — |

### S09 — Add Expense (Most Important)

| Context | Default | Override |
|---------|---------|----------|
| **Who paid?** | **Current user ("You")** | Pick any person or "Multiple" |
| **Split between** | **Everyone in event** | Pick a group or "Custom" |
| **Split method** | **Equal (weight: 1 each)** | Custom weights per person |
| Number of line items | 1 (single item mode) | "+ Add another line item" |
| Line item description | Same as transaction description | Separate descriptions per item |
| Currency | Event currency | Picker to select a different currency (multi-currency events); records FX rate when overridden |
| Date (occurred_at) | Current date/time ("Today") | Date picker to backdate expense to when it actually occurred |
| Modifier | 1.0 (hidden) | Future: child/dietary adjustments |

**Impact:** For the most common expense (one person paid, split equally among everyone), the user fills **2 fields** (description + amount) and taps **Save**. Everything else is defaulted.

### S11 — Balances

| Context | Default | Override |
|---------|---------|----------|
| View mode | By Settlement Group | Toggle to "By Person" |
| Balance display | Signed amounts (owes/owed) | — |
| "Settle Up" visibility | Only when user's PFG net < 0 | — |

### S12 — Settle Up

| Context | Default | Override |
|---------|---------|----------|
| Suggested settlements | Server-computed via `GET /events/{eid}/settlement-suggestions` (debt simplification) | Create custom settlement |
| From/to groups | Pre-filled from suggestion | Editable via custom |
| Amount | Pre-filled from net positions | Editable via custom |

### S13 — Membership

| Context | Default | Override |
|---------|---------|----------|
| Billing cycle | Monthly | Annual toggle |
| Upgrade suggestion | Based on current usage proximity to limits | — |

### S14 — Event Funding

| Context | Default | Override |
|---------|---------|----------|
| Funding method (no slots) | Per-event fee pre-selected | Switch to subscription slot |
| Funding method (has slots) | Subscription slot pre-selected | Switch to per-event fee |
| Cost spread | Offered immediately after funding | [Skip] |
| Slot type guidance | "One-off for single events, ongoing for trips" | — |

## Default Philosophy

### "Don't Make Me Think" Priority

1. **Zero-config start** — Creating an event requires one field (name). Person, role, invite code are automatic.
2. **One-field expense** — Description + amount is the minimum. Everything else defaults correctly.
3. **Invisible PFGs** — Users never see the term "Primary Financial Group". They see "settlement groups" with colours.
4. **Singleton by default** — New persons get solo settlement groups. The safe default. Sharing requires explicit action.
5. **Funding speed bump** — When hit, the path to fund is 2 taps. Cost spread is optional and immediate.

### Where We DON'T Default

Some things require explicit user choice:

- **Settlement group reassignment** — Too impactful to automate (affects who pays whom)
- **Settlement confirmation** — Admin must explicitly confirm (prevents premature payments)
- **Event closure** — Irreversible, requires confirmation
- **Account deletion** — Irreversible, requires confirmation
- **Merge persons** — Data integrity risk, requires explicit tap
