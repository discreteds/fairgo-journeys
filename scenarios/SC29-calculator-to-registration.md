# SC29 — Dave Splits a Dinner Without Signing Up

**Persona:** Dave — first-time user, heard about Fair Go from a friend. The loyalty maven for tonight — he paid for everything on his card.
**Screens:** S01 (Calculator) → S02 (Register, inline modal) → S05 (Event Dashboard) → S06 (Invite)
**Rails:** R10 (Calculator Conversion), R04 (Invitation)

## Design Principle

**Value before identity.** Dave proves Fair Go works before he gives it his email address. The calculator is the product demo, the hook, and the conversion trigger — in that order.

## The Situation

Dave paid $187.50 for dinner with 4 friends. Some had drinks, some didn't. He's tired of the "let's just split it equally" argument. He googles "fair bill split" and lands on fairgo.app.

| Line Item | Amount | Payer | Split |
|-----------|--------|-------|-------|
| Food (mains) | $125.00 | Dave | Everyone (5-way equal) |
| Drinks (wine) | $62.50 | Dave | Dave, Emma, Fiona (3-way equal) |
| **Total** | **$187.50** | | |

## Flow Diagram

```
fairgo.app (landing)
  │
  ▼
S01 Calculator (empty state)
  │ Dave adds 5 people
  │ Dave adds 2 line items
  │ Balance preview shows instantly
  │
  ▼
S01 Calculator (results ready)
  │ Dave clicks "Save & Share"
  │
  ▼
S02 Register (inline modal, calculator visible behind)
  │ Dave registers
  │ Calculator state migrates to server
  │
  ▼
S05 Event Dashboard (data intact)
  │ Dave taps "Share Event →"
  │
  ▼
S06 Prepare & Share
  │ Dave sends invite links
```

## Narrative

### Landing on the Calculator

Dave opens fairgo.app. There is no splash page, no "Get Started" button. The calculator is right there.

```
S01 Calculator (empty state)
    Event name: "Quick Split" (default)
    No people, no items
    Placeholder: "Add people and items to see who owes whom."
```

### Adding People

Dave types in the names of everyone at dinner. No accounts, no emails — just names.

```
    → adds "Dave" (himself)
    → adds "Emma"
    → adds "Fiona"
    → adds "Greg"
    → adds "Hannah"

    PEOPLE
    Dave · Emma · Fiona · Greg · Hannah
```

All client-side. No API calls. Five names stored in localStorage.

### Adding Line Items

Dave toggles to detailed mode to split by item.

```
    → taps "Split by item →"
    → mode switches to detailed

    → adds line item 1:
    ┌────────────────────────────────────────┐
    │ 1. Food (mains)                        │
    │    Amount: $125.00                      │
    │    Paid by: Dave                        │
    │    Split: Everyone equally              │
    │    Preview: $25.00 each                 │
    └────────────────────────────────────────┘
```

Now the drinks — not everyone drank. Dave uses the person picker to select only the drinkers.

```
    → adds line item 2:
    ┌────────────────────────────────────────┐
    │ 2. Drinks (wine)                       │
    │    Amount: $62.50                       │
    │    Paid by: Dave                        │
    │    Split: Custom ← Dave taps to change  │
    │                                         │
    │    Person picker:                        │
    │      ☑ Dave                             │
    │      ☑ Emma                             │
    │      ☑ Fiona                            │
    │      ☐ Greg   ← unchecked              │
    │      ☐ Hannah ← unchecked              │
    │                                         │
    │    Split: Dave, Emma, Fiona             │
    │    Preview: $20.83 each                 │
    └────────────────────────────────────────┘
```

### Balance Preview (Instant)

The moment Dave finishes, the balance preview updates in real-time:

```
    ── BALANCE PREVIEW ─────────────────────
    🔵 Dave        +$141.67  (owed)
    🔴 Emma        −$45.83   (owes)
    🔴 Fiona       −$45.83   (owes)
    🔴 Greg        −$25.00   (owes)
    🔴 Hannah      −$25.00   (owes)
    Checksum: $0.00 ✓
```

Dave immediately sees that Greg and Hannah owe $25 each (food only), while Emma and Fiona owe $45.83 each (food + drinks). No other app shows this distinction before you even create an account.

### Save & Share (Registration Trigger)

Dave wants to send everyone their share. He taps "Save & Share."

```
    → taps "Save & Share"

    Registration modal opens (inline, calculator visible behind):
    ┌──────────────────────────────────────┐
    │  Create an account to save your split │
    │                                       │
    │  Display name: [Dave             ]    │
    │  Email:        [dave@example.com ]    │
    │  Password:     [••••••••         ]    │
    │                                       │
    │  [Register & Save]                    │
    │                                       │
    │  Already have an account? [Log in]    │
    └──────────────────────────────────────┘
```

### Registration and Data Migration

Dave fills in the form and registers.

```
    → fills form, taps "Register & Save"
    ⚡ POST /auth/register → tokens stored

    Calculator state migrates to server:
    ⚡ POST /events
       {name: "Quick Split", base_currency: "AUD"}
       → event created, Dave auto-added as admin + person

    ⚡ POST /events/{eid}/persons {display_name: "Emma"}
    ⚡ POST /events/{eid}/persons {display_name: "Fiona"}
    ⚡ POST /events/{eid}/persons {display_name: "Greg"}
    ⚡ POST /events/{eid}/persons {display_name: "Hannah"}

    ⚡ POST /events/{eid}/transactions
       {description: "Quick Split",
        line_items: [
          {description: "Food (mains)", amount: "125.00",
           payer_person_id: dave_server_id,
           splits: [
             {person_id: dave_id, side: "expense", weight: 1},
             {person_id: dave_id, side: "consumption", weight: 1},
             {person_id: emma_id, side: "consumption", weight: 1},
             {person_id: fiona_id, side: "consumption", weight: 1},
             {person_id: greg_id, side: "consumption", weight: 1},
             {person_id: hannah_id, side: "consumption", weight: 1}
           ]},
          {description: "Drinks (wine)", amount: "62.50",
           payer_person_id: dave_server_id,
           splits: [
             {person_id: dave_id, side: "expense", weight: 1},
             {person_id: dave_id, side: "consumption", weight: 1},
             {person_id: emma_id, side: "consumption", weight: 1},
             {person_id: fiona_id, side: "consumption", weight: 1}
           ]}
        ]}

    localStorage calculator state cleared.
```

### Landing on the Dashboard

Dave lands on S05. Everything he entered is there — same numbers, same splits, now persisted on the server.

```
S05 Event Dashboard
    "Quick Split"
    5 people · AUD · Active
    1 expense, $187.50 total

    YOUR POSITION
    You are owed $141.67

    RECENT EXPENSES
    Quick Split — $187.50 — Dave paid
```

### Sharing with Friends

Dave taps "Share Event →" to send everyone their share.

```
    → taps "Share Event →"

S06 Prepare & Share
    → Person checklist:
      Emma   — placeholder [Add phone / Create personal link]
      Fiona  — placeholder [Add phone / Create personal link]
      Greg   — placeholder [Add phone / Create personal link]
      Hannah — placeholder [Add phone / Create personal link]
    → Share options:
      [Copy Group Link]
      [Share via...]

    Each recipient sees their specific balance:
      Emma's link   → "You owe Dave $45.83"
      Fiona's link  → "You owe Dave $45.83"
      Greg's link   → "You owe Dave $25.00"
      Hannah's link → "You owe Dave $25.00"
```

## Arithmetic Verification

| Person | Food Share (1/5 of $125) | Drinks Share (1/3 of $62.50) | Total Owed | Paid | Net Position |
|--------|-------------------------|------------------------------|------------|------|-------------|
| Dave | $25.00 | $20.83 | $45.83 | $187.50 | +$141.67 |
| Emma | $25.00 | $20.83 | $45.83 | $0 | −$45.83 |
| Fiona | $25.00 | $20.83 | $45.83 | $0 | −$45.83 |
| Greg | $25.00 | $0 | $25.00 | $0 | −$25.00 |
| Hannah | $25.00 | $0 | $25.00 | $0 | −$25.00 |
| **Checksum** | **$125.00** | **$62.50** | **$187.50** | **$187.50** | **$0.00 ✓** |

Rounding verification: $62.50 ÷ 3 = $20.8333... → rounded to $20.83. Three shares: $20.83 × 3 = $62.49. Rounding remainder ($0.01) assigned to first participant (Dave) by server rounding rules. Client preview shows $20.83 for all three; server applies penny-exact allocation.

## Validates

- **Anonymous calculator works without authentication** — no API calls until registration
- **Line-item-level splits with different participant sets** — food (5-way) vs drinks (3-way)
- **Live balance computation matches server arithmetic** — same formula, checksum = $0.00
- **Seamless data migration on registration** — calculator state → API entities in one flow
- **Free tier limits respected** — 2 line items, well within 5-item limit
- **Client-side calculation uses the same formula as server** — weight-based proportional splits
- **localStorage persistence** — calculator state survives page refresh
- **Registration modal is inline** — calculator remains visible, reducing abandonment
- **Deep link to S06** — sharing is the natural next step after saving

## The Differentiator Moment

Dave immediately sees that Greg and Hannah owe $25 each (food only), while Emma and Fiona owe $45.83 each (food + drinks). No other app shows this distinction before you even create an account. Fair Go proved its value in 30 seconds.

Under an equal split, everyone would pay $37.50. Greg and Hannah would overpay by $12.50 each. Emma and Fiona would underpay by $8.33 each. The calculator makes this visible instantly.

> **"Greg didn't drink. Hannah didn't drink. They're not splitting the wine. Fair go."**

---

## Variant: SC29b — Dave Hits the Free Tier Limit

Same persona, bigger dinner. Dave has a complex dinner with many components.

### The Situation

Dave's dinner has 6 line items: Entrees, Mains, Sides, Drinks, Dessert, Coffee. After adding the 5th item (Dessert):

```
    ITEMS
    1. Entrees    $45.00   Everyone equally
    2. Mains      $180.00  Everyone equally
    3. Sides      $35.00   Everyone equally
    4. Drinks     $95.00   Dave, Emma, Fiona
    5. Dessert    $42.00   Everyone equally

    ┌────────────────────────────────────┐
    │ + Add item                         │
    │ Register for unlimited items →     │
    └────────────────────────────────────┘
```

### What Happens

- The balance preview still shows correct totals for all 5 items
- Dave can see who owes whom based on the 5 items entered
- The 6th item (Coffee, $18.00) cannot be added until registration
- "Register for unlimited items →" links to the same registration flow
- After registering, the 5-item limit is lifted immediately (Registered tier allows unlimited line items per transaction)
- Dave can then add Coffee as a 6th line item on S09

### Key Difference from SC29

The registration trigger is the limit, not the "Save & Share" button. The soft upsell message is non-blocking — Dave can still use the 5-item calculator and see meaningful results. The 6th item is the carrot, not the stick.

## Cross-References

- **S01 (Calculator):** Screen spec for the calculator landing page
- **S02 (Register/Login):** Registration form used in inline modal
- **S05 (Event Dashboard):** Migration target after registration
- **S06 (Prepare & Share):** Next step after saving — sharing with friends
- **R10 (Calculator Conversion rail):** Navigation rail for this flow
- **Pricing model:** Free tier limits from `01.mvp_planning/pricing-model/01.membership-tiers-and-event-funding.md`
- **SC01 (Alice Organizes Dinner):** Comparable dinner scenario for authenticated user flow
