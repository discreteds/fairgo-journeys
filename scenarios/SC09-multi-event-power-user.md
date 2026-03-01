# SC09 — Power User Manages Multiple Events

**Persona:** Dave — manages 3 concurrent events (work lunch, weekend trip, housemates).
**Screens:** S03 (home with multiple events) → S05 → S15 → S17
**Rails:** R01 (Onboarding), R02 (Expense)

## Narrative

Dave is already registered (from SC04). He creates multiple events:

```
Dave's S03 Home → "Housemates" event already exists
    → taps "+ New Event"
S04 → "Work Lunch Friday"
    ⚡ POST /events → event created
    → back to S03

S03 Home → taps "+ New Event"
S04 → "Weekend Bay Trip"
    ⚡ POST /events → event created
    → back to S03
```

Dave's home screen shows all events with position summaries:

```
S03 Home
    ⚡ GET /events?include=my_position
    → 3 event cards:
    ┌──────────────────────────────────┐
    │ 🏠 Housemates        ongoing    │
    │    You are owed $45.00          │
    │    Last activity: 2h ago        │
    ├──────────────────────────────────┤
    │ 🍕 Work Lunch Friday  one_off   │
    │    No expenses yet              │
    │    Created just now             │
    ├──────────────────────────────────┤
    │ 🏖️ Weekend Bay Trip   one_off   │
    │    No expenses yet              │
    │    Created just now             │
    └──────────────────────────────────┘
```

Dave adds people to the work lunch (reusing names that exist in other events):

```
S05 (Work Lunch) → taps "People ▸"
S07 → adds Alice, Bob, Eve
    ⚡ POST /events/{eid}/persons × 3
    → each person created fresh (event-scoped isolation)
    → these are NOT the same person records as in Housemates
    → no cross-event person linking
```

Dave switches between events quickly:

```
S05 (Work Lunch) → taps ← back
S03 Home → taps "Housemates"
S05 (Housemates) → sees housemates context
    → adds expense "Electricity $180"
    ⚡ POST /events/{eid}/transactions
    → back to S03

S03 → taps "Weekend Bay Trip"
S05 (Bay Trip) → different event context loaded
```

Dave checks his profile and activity:

```
S03 → taps profile icon
S15 Profile & Settings
    ⚡ GET /users/me
    → Display name: Dave
    → Email: dave@example.com
    → Member since: ...
    ⚡ GET /users/me/persons
    → Shows all persons across all events:
      - Dave (Housemates) — admin
      - Dave (Work Lunch Friday) — admin
      - Dave (Weekend Bay Trip) — admin

S03 → taps notification bell
S17 Activity Feed / Notifications
    → Recent activity across all events:
      - "Electricity $180 added to Housemates" — 5m ago
      - "Alice joined Work Lunch Friday" — 10m ago
      - "Weekend Bay Trip created" — 15m ago
```

## Validates

- Event list with inline position summaries (`?include=my_position`, P1 gap)
- Context switching between events preserves correct state
- Person reuse across events creates separate records (event-scoped isolation)
- Activity feed (S17) aggregates across events (P3 gap)
- Profile management (S15) with cross-event person listing
- Multiple event creation and management by single user
- Home screen ordering (recent activity vs creation date)

## Key Principle Demonstrated

> **"Event-scoped isolation"**
>
> "Alice" in Work Lunch and "Alice" in Housemates are separate person records.
> Events are independent financial contexts. Cross-event person linking is a
> future feature (I3) — for now, each event is its own world.
