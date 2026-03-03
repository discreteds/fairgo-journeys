# S01 — Calculator

**Purpose:** Landing page AND working bill-splitting calculator. Zero-friction value demonstration — users split a real bill before creating an account.
**Visible to:** Everyone. Unauthenticated users see the calculator. Authenticated users redirect to S03 (Home).
**Rails:** R01 (Onboarding), R10 (Calculator Conversion)
**Scenarios:** SC01, SC02, SC07, SC29, SC29b

## Design Principle

**The calculator IS the landing page.** Fair Go proves its value in 30 seconds — before signup, before any API call. The user enters a real bill, sees who owes whom, and immediately understands why "fair ≠ equal." Registration is the reward gate: save your work, share with friends, settle up.

## Wireframe — Empty State

```
┌──────────────────────────────────────┐
│  Fair Go                    [Log in] │
│  Fair ≠ equal                        │
│  Split a bill right now.             │
├──────────────────────────────────────┤
│                                      │
│  Event name:  [Quick Split       ]   │
│                                      │
│  PEOPLE                              │
│  [+ Add person]                      │
│                                      │
│  ITEMS                               │
│  [+ Add item]                        │
│                                      │
│                                      │
│  ┌──────────────────────────────┐    │
│  │  Add people and items to     │    │
│  │  see who owes whom.          │    │
│  └──────────────────────────────┘    │
│                                      │
└──────────────────────────────────────┘
```

## Wireframe — In Progress (Simple Mode)

```
┌──────────────────────────────────────┐
│  Fair Go                    [Log in] │
│  Fair ≠ equal                        │
│  Split a bill right now.             │
├──────────────────────────────────────┤
│                                      │
│  Event name:  [Friday Dinner     ]   │
│                                      │
│  PEOPLE                              │
│  Alice · Bob · Carol · Dave          │
│  [+ Add person]                      │
│                                      │
│  ITEMS                [Split by item →] │
│  Total: [$125.00]                    │
│  Paid by: [Alice ▾]                  │
│  Split: Everyone equally             │
│                                      │
│  ── BALANCE PREVIEW ─────────────    │
│  🔵 Alice       +$93.75  (owed)     │
│  🔴 Bob         −$31.25  (owes)     │
│  🔴 Carol       −$31.25  (owes)     │
│  🔴 Dave        −$31.25  (owes)     │
│  Checksum: $0.00 ✓                   │
│                                      │
│  ┌──────────────────────────────┐    │
│  │  💾 Save & Share             │    │
│  └──────────────────────────────┘    │
│  I have a code · Log in             │
│                                      │
└──────────────────────────────────────┘
```

## Wireframe — In Progress (Detailed Mode)

```
┌──────────────────────────────────────┐
│  Fair Go                    [Log in] │
│  Fair ≠ equal                        │
│  Split a bill right now.             │
├──────────────────────────────────────┤
│                                      │
│  Event name:  [Friday Dinner     ]   │
│                                      │
│  PEOPLE                              │
│  Alice · Bob · Carol · Dave · Emma   │
│  [+ Add person]                      │
│                                      │
│  ITEMS                 [← Simple mode] │
│                                      │
│  ┌────────────────────────────────┐  │
│  │ 1. Food (mains)               │  │
│  │    Amount: [$125.00]           │  │
│  │    Paid by: [Dave ▾]           │  │
│  │    Split: Everyone equally     │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │ 2. Drinks (wine)              │  │
│  │    Amount: [$62.50]            │  │
│  │    Paid by: [Dave ▾]           │  │
│  │    Split: Dave, Emma, Fiona ▾  │  │
│  └────────────────────────────────┘  │
│  [+ Add item]  (3 of 5 free)        │
│                                      │
│  ── BALANCE PREVIEW ─────────────    │
│  🔵 Dave        +$141.67  (owed)    │
│  🔴 Emma        −$45.83   (owes)    │
│  🔴 Fiona       −$45.83   (owes)    │
│  🔴 Greg        −$25.00   (owes)    │
│  🔴 Hannah      −$25.00   (owes)    │
│  Checksum: $0.00 ✓                   │
│                                      │
│  ┌──────────────────────────────┐    │
│  │  💾 Save & Share             │    │
│  └──────────────────────────────┘    │
│  I have a code · Log in             │
│                                      │
└──────────────────────────────────────┘
```

## Wireframe — At Limit (5 items)

```
│  ITEMS                               │
│  ...                                 │
│  ┌────────────────────────────────┐  │
│  │ 5. Dessert                     │  │
│  │    Amount: [$35.00]            │  │
│  │    Paid by: [Alice ▾]          │  │
│  │    Split: Everyone equally     │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │ + Add item                     │  │
│  │ Register for unlimited items → │  │
│  └────────────────────────────────┘  │
```

## Calculator Modes

### Simple Mode (Default)

Single total amount + payer + list of people = equal split. Matches the "quick split" use case.

- One amount field (the total)
- One payer selector
- "Everyone equally" is the only split option
- Toggle: "Split by item →" switches to detailed mode

### Detailed Mode

Line items with per-item custom splits (weights). Toggle: "← Simple mode" switches back.

- Multiple line item rows, each with: description, amount, payer, split selector
- Per-item split selector: "Everyone equally" (default) or custom person picker
- Matches S09's detailed mode interface
- "+ Add item" button (disabled after 5 items in free tier)

## Actions

| Action | Target | Orchestration |
|--------|--------|--------------|
| Save & Share (primary CTA) | → S02 (register tab, inline modal) | Registration prompt — see Register-to-Save flow below |
| Log in (secondary) | → S02 (login tab) | Navigation only |
| I have a code (tertiary) | → S02 (register tab, invite code stored in session) | Navigation only |
| Split by item → | Toggle to detailed mode | Client-side state only |
| ← Simple mode | Toggle to simple mode | Client-side state only |
| + Add person | Add name to people list | Client-side state only |
| + Add item | Add line item row | Client-side state only; disabled at 5 items (free tier) |

## Backend Calls

**None for the calculator itself.** All split math runs client-side in the browser. No API calls until registration.

## Client-Side Data Model

Mirrors the server domain model for seamless migration on registration:

```
calculatorState = {
  eventName: string,            // default: "Quick Split"
  currency: "AUD",
  mode: "simple" | "detailed",
  people: [{
    name: string,
    tempId: string              // client-generated UUID
  }],
  // Simple mode:
  simpleTotal: number,          // decimal
  simplePaidBy: tempId,
  // Detailed mode:
  lineItems: [{
    description: string,
    amount: number,             // decimal
    paidBy: tempId,
    splits: [{
      personId: tempId,
      weight: number            // default 1
    }]
  }],
  // Computed (not stored):
  balances: [{
    personId: tempId,
    netPosition: number,
    owes: [{ to: tempId, amount: number }]
  }]
}
```

Stored in `localStorage` under key `fairgo_calculator_state`. Survives page refresh. Explicitly ephemeral — no server sync until registration.

## Balance Computation (Client-Side)

Uses the same formula as the server:

```
effective_weight[i] = weight[i]  // no modifiers in calculator
person_share[i]     = line_item.amount × (weight[i] / Σ weights)
person_net[i]       = Σ paid[i] − Σ consumed[i]
```

Balances update in real-time on every input change. Checksum (Σ net positions = $0.00) is always displayed.

In simple mode, one implicit line item is created from the total amount with all people having equal weight.

## Free Tier Limits (Client-Side Enforced)

| Limit | Value | Behaviour at Limit |
|-------|-------|-------------------|
| Transactions | 1 | The calculator IS the transaction — single transaction only |
| Line items | 5 | "+ Add item" disabled; soft message: "Register for unlimited items →" |
| Participants | Unlimited | No restriction — the calculator is the acquisition hook |
| Settlement | Preview only | Balances shown, no "Settle" action available |

These limits match the pricing model's unfunded event limits (see `01.mvp_planning/pricing-model/01.membership-tiers-and-event-funding.md`).

## Register-to-Save Flow (S01 → S02 → S05)

### Triggers

Registration is prompted when the user:
- Clicks "Save & Share" (primary CTA)
- Tries to add a 6th line item (soft prompt)
- Clicks any settlement action (if shown)

### Flow

1. Registration modal opens **inline** — calculator remains visible behind overlay
2. Modal shows S02 Register form (email + password) with context: "Create an account to save your split"
3. On successful registration:
   - JWT tokens stored
   - Client-side calculator state transformed into API payloads:
     - `POST /api/v1/events` creates the event
     - `POST /api/v1/events/{id}/persons` creates each person (as placeholders)
     - `POST /api/v1/events/{id}/transactions` creates the transaction with line items and splits
   - User lands on **S05 Event Dashboard** with their calculator data fully persisted
4. `localStorage` calculator state cleared after successful migration

### Error Handling

| Error | Behaviour |
|-------|-----------|
| Network failure during migration | Show error, keep calculator state, let user retry |
| Registration fails (email taken) | Show login option, preserve calculator state |
| API error during data migration | Show error with retry button; calculator state preserved in localStorage |

### Data Migration Mapping

| Calculator Field | API Entity | API Field |
|-----------------|------------|-----------|
| `eventName` | EVENT | `name` |
| `currency` | EVENT | `base_currency` |
| `people[].name` | PERSON | `display_name` (placeholder) |
| `lineItems[].description` | LINE_ITEM | `description` |
| `lineItems[].amount` | LINE_ITEM | `amount` |
| `lineItems[].paidBy` | LINE_ITEM_SPLIT | expense side, weight 1 |
| `lineItems[].splits[]` | LINE_ITEM_SPLIT | consumption side |

## States

| State | Behaviour |
|-------|-----------|
| Empty | Form shown with placeholder copy; no balance preview |
| In progress | Items and people added; live balance preview updating |
| At limit | 5 line items reached; soft upsell to register |
| Results ready | Full balance breakdown visible; CTAs prominent |
| Authenticated user visits `/` | Redirect to S03 (Home) |
| Deep link with invite code (`/join/{code}`) | Redirect to S02 with code pre-loaded |
| Returning visitor (localStorage has state) | Calculator restores previous state |

## Smart Defaults

- Event name defaults to "Quick Split" (editable)
- Currency defaults to "AUD"
- Mode defaults to "Simple"
- "Paid by" defaults to first person added
- New line items default to "Everyone equally" split
- If user arrives via deep link (`fairgo.app/join/X7kQ2m`), bypass calculator and go directly to S02 with invite code in session (same as before)

## Cross-References

- **S09 (Add Expense):** Calculator's detailed mode mirrors S09's multi-line-item interface
- **S02 (Register/Login):** Registration modal uses S02's form, rendered inline
- **S05 (Event Dashboard):** Migration target after registration
- **Pricing model:** Free tier limits from `01.mvp_planning/pricing-model/01.membership-tiers-and-event-funding.md`
- **R10 (Calculator Conversion rail):** Primary navigation rail for this screen
- **SC29 (Dave Splits a Dinner):** Primary scenario exercising this screen
