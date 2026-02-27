# S07 — Manage People

**Purpose:** View/add/edit persons. Identity resolution (merge). Settlement group assignment.
**Visible to:** All (admin has extra actions).
**Rails:** R02, R06
**Scenarios:** SC01, SC03

## Wireframe

```
┌──────────────────────────────┐
│  ← People · Bali Trip        │
├──────────────────────────────┤
│                              │
│  ┌──────────────────────────┐│
│  │ 🟢 Alice (you)          ││
│  │ 🔵 Alice & Partner       ││
│  ├──────────────────────────┤│
│  │ 🟢 Bob                  ││
│  │ 🔴 Bob (solo)            ││
│  ├──────────────────────────┤│
│  │ 🟡 Dave  ⏳ placeholder  ││
│  │ 🔴 Dave (solo)           ││
│  ├──────────────────────────┤│
│  │ 🟢 Carol                ││
│  │ 🔵 Alice & Partner       ││
│  ├──────────────────────────┤│
│  │ 🟡 Eve  ⏳ placeholder   ││
│  │ 🔴 Eve (solo)            ││
│  ├──────────────────────────┤│
│  │ 🟢 Frank                ││
│  │ 🔴 Frank (solo)          ││
│  └──────────────────────────┘│
│                              │
│  Legend:                     │
│  🟢 Active  🟡 Placeholder  │
│  🔵 Shared settlement group  │
│  🔴 Solo settlement group    │
│                              │
│       ( + Add Person )       │  ← admin only
└──────────────────────────────┘
```

Each person row shows:
- **Line 1:** Status icon + display name + resolution badge
- **Line 2:** Settlement group colour + settlement group name

## Wireframe — Person Detail (Admin Tap)

```
┌──────────────────────────────┐
│  Dave  ⏳ placeholder         │
├──────────────────────────────┤
│                              │
│  Settlement Group            │
│  🔴 Dave (solo)              │
│  [Change Settlement Group ▸] │
│                              │
│  ⚠️ Potential match found    │
│  ┌──────────────────────────┐│
│  │ "Dave" matches a user    ││
│  │ who just joined.         ││
│  │ Confidence: High (email) ││
│  │ [Merge] [Dismiss]        ││
│  └──────────────────────────┘│
│                              │
│  [Send Personal Invite]      │
│  [Edit Name]  [Remove]       │
└──────────────────────────────┘
```

## Wireframe — Person Detail (Member Tap, Own Placeholder)

```
┌──────────────────────────────┐
│  Dave  ⏳ placeholder         │
├──────────────────────────────┤
│                              │
│  ┌──────────────────────────┐│
│  │ This looks like you!     ││
│  │ [This is me]             ││
│  └──────────────────────────┘│
│                              │
│  Settlement Group            │
│  🔴 Dave (solo)              │
└──────────────────────────────┘
```

## Orchestration — Page Load

```
1. GET /events/{eid}/persons         → person list with resolution_status
2. For each person (parallelised):
   GET /events/{eid}/persons/{pid}/pfg → PFG (settlement group) info
3. GET /events/{eid}/groups          → group list for settlement group names
```

## Orchestration — "Add Person" (Admin)

```
1. Show inline form: display name (required), email hint (optional), phone hint (optional)
2. POST /events/{eid}/persons
     {display_name, email_hint?, phone_hint?}
   → person created with resolution_status: placeholder
   → singleton PFG auto-created by backend
3. Refresh person list
```

## Orchestration — "Merge" (Admin)

When a potential match is shown:

```
1. POST /events/{eid}/persons/merge
     {source_person_id: placeholder_id, target_person_id: joined_person_id}
   → all LINE_ITEM_SPLITs transferred from source to target
   → all GROUP_MEMBERships transferred
   → all USER_PERSON records transferred
   → source person marked status: removed
2. Refresh person list
```

## Orchestration — "This is me" (Member Self-Merge)

```
1. POST /events/{eid}/persons/merge
     {source_person_id: placeholder_id, target_person_id: my_person_id}
   → same merge endpoint, authorize_self_merge() validates:
     - source is placeholder
     - target belongs to caller
     - display_name matches (case-insensitive)
2. Refresh person list
```

## Orchestration — "Change Settlement Group"

```
1. Show picker with options:
   a. Existing non-singleton groups (e.g. "Alice & Partner")
   b. "Create new shared group"
2. If reassigning to existing group:
   PUT /events/{eid}/persons/{pid}/pfg
     {group_id: existing_group_id}
3. If creating new shared group:
   a. Prompt for group name
   b. POST /events/{eid}/groups {name, is_singleton: false}
   c. PUT /events/{eid}/persons/{pid}/pfg {group_id: new_group_id}
4. Refresh person list (settlement group colours update)
```

## Orchestration — "Send Personal Invite" (Admin)

```
→ S06 (generate side, pre-selected person)
```

## Smart Defaults

- Colour-coded settlement groups make financial relationships instantly visible at a glance
- Merge suggestions surface proactively when matches exist — admin doesn't have to go looking
- Self-merge available to members who see their own placeholder ("This is me" button)
- Email/phone hints are optional on add — useful for identity resolution but not required
- Persons sorted: active first, then placeholders

## Error States

| Error | Display |
|-------|---------|
| Merge fails (name mismatch for self) | "Names don't match. Ask an admin to merge these records." |
| Participant limit reached (unfunded) | "This event needs funding to add more people" → S14 |
| Remove person with splits | "Can't remove — this person has expenses. Reassign first." |
| PFG reassignment to singleton | "Can't join a solo settlement group. Create a shared one instead." |
