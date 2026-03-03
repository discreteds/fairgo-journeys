# R10 — Calculator Conversion Rail

**Purpose:** Anonymous calculator to registered user with persisted data. The primary acquisition rail.
**Primary persona:** Anonymous visitor converting to registered user.

## Rail Path

```
S01 Calculator (anonymous)
  │ User enters people, items, splits
  │ Balance preview shows instantly
  │ (all client-side, no API calls)
  │
  │ ···· conversion trigger ····
  │ (Save & Share / 6th item / settle action)
  │
  ▼
S02 Register (inline modal over S01)
  │ "Create an account to save your split"
  │ ⚡ POST /auth/register
  │
  │ ···· data migration ····
  │ ⚡ POST /events
  │ ⚡ POST /events/{eid}/persons (× N)
  │ ⚡ POST /events/{eid}/transactions
  │ localStorage cleared
  │
  ▼
S05 Event Dashboard (data intact)
  │ [transfer to R02 Expense, R04 Invitation]
  │
  ▼
S06 Prepare & Share (optional next step)
  │ Send invite links to friends
```

### Existing User Path

```
S01 Calculator (anonymous)
  │ User clicks "Log in"
  │
  ▼
S02 Login
  │ ⚡ POST /auth/login
  │
  │ ···· same data migration ····
  │
  ▼
S05 Event Dashboard (data intact)
```

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S01 → S02 | User clicks "Save & Share" or hits free tier limit | Registration modal (inline) |
| S01 → S02 | User clicks "Log in" | S02 login tab (full page) |
| S01 → S02 | User clicks "I have a code" | S02 register tab with invite code in session |
| S02 → S05 | Registration + data migration successful | R01 continues from S05 |
| S05 → S06 | User taps "Share Event →" | R04 (Invitation Rail) |
| S05 → S09 | User taps "+ Add Expense" | R02 (Expense Rail) |

## Key Orchestration Sequence

The "anonymous to registered with data" path:

```
# 1. Register
POST /auth/register

# 2. Create event from calculator state
POST /events {name: "Quick Split", base_currency: "AUD"}

# 3. Create persons (one per calculator person, excluding self — auto-created)
POST /events/{eid}/persons {display_name: "Emma"}
POST /events/{eid}/persons {display_name: "Fiona"}
POST /events/{eid}/persons {display_name: "Greg"}

# 4. Create transaction with line items and splits
POST /events/{eid}/transactions {
  description: "Quick Split",
  line_items: [
    {description: "Food", amount: "125.00",
     payer_person_id: self_id,
     splits: [...]},
    {description: "Drinks", amount: "62.50",
     payer_person_id: self_id,
     splits: [...]}
  ]
}
```

**Person ID mapping:** Calculator uses client-generated `tempId` values. During migration, the frontend maps `tempId` → server-assigned person ID after each `POST /persons` response. The registering user's own person is created automatically by `POST /events` and mapped to their calculator `tempId`.

## Error Recovery

| Error | Recovery |
|-------|----------|
| Registration fails (409 email exists) | Show login option; calculator state preserved in localStorage |
| Network failure during migration | Show retry button; calculator state preserved in localStorage |
| Partial migration (event created, persons fail) | Show error; user can manually complete on S05 |
| Migration succeeds but S05 load fails | Redirect to S03 (Home); user can navigate to event |

## Scenarios Using This Rail

- SC29 (Dave Splits a Dinner Without Signing Up) — full anonymous-to-registered path
- SC29b (Dave Hits the Free Tier Limit) — registration triggered by 5-item limit
