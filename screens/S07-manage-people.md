# S07 — Manage People

**Purpose:** View/add/edit persons. Identity resolution (merge). Settlement group assignment. Inline split preview on add.
**Visible to:** All (admin has extra actions).
**Rails:** R02, R06
**Scenarios:** SC01, SC03, SC04, SC07, SC08, SC09

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
│   ( ✉ Request Add Person )   │  ← member only
└──────────────────────────────┘
```

**Role-aware actions:**
- **Admin** sees `+ Add Person` — creates the person directly via `POST /events/{eid}/persons`
- **Member** sees `✉ Request Add Person` — submits a modification request via `POST /events/{eid}/modification-requests` with `type: "add_person"`. The request enters the admin's moderation queue (S16). The person is only created once an admin approves.

Each person row shows:
- **Line 1:** Status icon + display name + resolution badge
- **Line 2:** Settlement group colour + settlement group name

## Wireframe — Inline Split Preview (After Adding Person)

When a person is added and split-pending transactions are auto-assigned, an inline split summary appears showing the effect on all affected expenses:

```
┌──────────────────────────────┐
│  ← People · Friday Dinner    │
├──────────────────────────────┤
│                              │
│  ┌──────────────────────────┐│
│  │ 🟢 Alice (you)          ││
│  ├──────────────────────────┤│
│  │ 🟡 Bob  ⏳ placeholder   ││
│  └──────────────────────────┘│
│                              │
│  ┌──────────────────────────┐│
│  │ ✓ Bob added              ││
│  │                          ││
│  │ Updated splits:          ││
│  │ ┌────────────────────────┐│
│  │ │ Pizza $45.00        ▸  ││
│  │ │ → $22.50 each         ││
│  │ │   (Alice, Bob)         ││
│  │ ├────────────────────────┤│
│  │ │ Drinks $30.00       ▸  ││
│  │ │ → $15.00 each         ││
│  │ │   (Alice, Bob)         ││
│  │ └────────────────────────┘│
│  └──────────────────────────┘│
│                              │
│       ( + Add Person )       │
└──────────────────────────────┘
```

Each expense row in the split summary is tappable (▸) to adjust individual weights for that expense.

## Wireframe — Split Adjustment (Tapped Expense Row)

```
┌──────────────────────────────┐
│  ← Adjust Split · Pizza      │
├──────────────────────────────┤
│                              │
│  Pizza — $45.00              │
│                              │
│  Split method                │
│  ● Equal  ○ Custom weights   │
│                              │
│  ┌──────────────────────────┐│
│  │ Alice    [1_]  $22.50    ││
│  │ Bob      [1_]  $22.50    ││
│  └──────────────────────────┘│
│                              │
│  ┌──────────────────────────┐│
│  │        Save Split        ││
│  └──────────────────────────┘│
└──────────────────────────────┘
```

## Wireframe — Multiple Pending Expenses

When there are multiple affected expenses, a "Review All Splits" action appears:

```
│  ┌──────────────────────────┐│
│  │ ✓ Dave added             ││
│  │                          ││
│  │ Updated splits:          ││
│  │ ┌────────────────────────┐│
│  │ │ Pizza $45.00        ▸  ││
│  │ │ → $11.25 each (4)     ││
│  │ ├────────────────────────┤│
│  │ │ Drinks $30.00       ▸  ││
│  │ │ → $7.50 each (4)      ││
│  │ ├────────────────────────┤│
│  │ │ Taxi $20.00         ▸  ││
│  │ │ → $5.00 each (4)      ││
│  │ └────────────────────────┘│
│  │                          ││
│  │ [Review All Splits]      ││
│  └──────────────────────────┘│
```

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
1. GET /events/{eid}/persons         -> person list with resolution_status and embedded PFG
2. GET /events/{eid}/groups          -> group list for settlement group names
3. GET /events/{eid}/persons/my-matches -> match suggestions (member view only)
```

PFG data (group_id, group_name, is_singleton) is embedded in each person response via `_person_to_dict()`. No per-person PFG call needed.

**Note:** Merged persons (resolution_status: `merged`) are excluded from match suggestion results returned by `/persons/my-matches`. This prevents the person claim/match autocomplete from suggesting persons that have already been merged into another identity.

## Orchestration — "Add Person" (Admin)

```
1. Show inline form: display name (required), email hint (optional), phone hint (optional)
2. POST /events/{eid}/persons
     {display_name, email_hint?, phone_hint?}
   → Response: PersonCreateResponse
     {
       person: PersonOut,
       affected_transactions: [TransactionOut, ...]
     }
   → person created with resolution_status: placeholder
   → singleton PFG auto-created by backend
   → if split-pending transactions exist: splits auto-assigned, splits_status → "assigned"
3. If affected_transactions is non-empty:
   → Show inline split preview (see wireframe above)
   → Each expense row is tappable to adjust weights
4. Refresh person list
```

## Orchestration — "Add People" (Batch, Admin)

For events with many participants, the admin can add multiple persons at once:

```
POST /events/{eid}/persons/batch
  {
    persons: [
      {display_name: "Bob", email_hint: "bob@example.com"},
      {display_name: "Carol"},
      {display_name: "Dave", phone_hint: "+61412345678"}
    ]
  }
→ Response: {persons: [PersonOut, PersonOut, PersonOut], affected_transactions: [...]}
→ Each person created with resolution_status: placeholder
→ Singleton PFGs auto-created for each
→ If split-pending transactions exist: splits auto-assigned for all new persons
```

Returns 422 if the `persons` array is empty. Each person in the array follows the same validation as single creation. If the event's person limit would be exceeded, the entire batch is rejected with `UNFUNDED_LIMIT`.

## Orchestration — "Request Add Person" (Member)

Members cannot create persons directly. Instead they submit a modification request for admin approval.

```
1. Show inline form: display name (required), email hint (optional), phone hint (optional)
2. POST /events/{eid}/modification-requests
     {
       type: "add_person",
       proposed_changes: {
         display_name: "...",
         email_hint?: "...",
         phone_hint?: "..."
       },
       message?: "Reason for adding this person"
     }
   → status: pending
   → "Request sent to admin ✓"
3. Request appears in admin's S16 moderation queue
4. On admin approval: person is auto-created (no separate POST needed)
   → member is notified of approval
5. On admin rejection: member is notified with optional reason
```

## Orchestration — "Adjust Split" (Tapped Expense Row)

```
1. User taps an expense row in the split preview
2. Show split adjustment form (weight per person)
3. PATCH /events/{eid}/transactions/{tid}/line-items/{lid}/splits
     {splits: [{person_id, weight}, ...]}
   → dedicated split editing endpoint (see also S10)
4. Refresh split preview to show updated shares
```

## Orchestration — "Review All Splits"

```
1. User taps "Review All Splits" (visible when ≥2 affected expenses)
2. Show all affected expenses with their current splits in a scrollable list
3. Each expense row is individually adjustable (same as single expense tap)
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

Self-merge routing verified: non-admin members reach `authorize_self_merge()` via the try/except in the merge endpoint. The route-level auth (`require_event_role()`) permits members. Integration tested (Gap I2).

## Orchestration — "Claim as Dependent" (Member, CR-021)

Members can claim placeholder persons as dependents (children, partners) in addition to claiming as "self". This allows a parent to manage a child's participation without the child needing an account.

**Trigger:** Member taps a placeholder person card → Person Detail sheet shows "This is me" (existing) and a new "Claim as Dependent" button.

```
1. POST /events/{eid}/persons/{pid}/claim
   Body: { "role": "sponsor" }
   → 200: person with resolution_status "claimed"
2. Creates UserPerson with role="sponsor" (not "self")
3. Unlike "self" claiming, a user can sponsor multiple persons
   while having only one "self" person per event
```

**Person Detail wireframe update:** Add a second button below "This is me" labeled "Claim as Dependent". Both buttons call the same endpoint with different `role` values (`"self"` vs `"sponsor"`).

## PFG Discovery Prompt

After adding people, S07 surfaces a natural-language prompt to discover settlement groups — not a settings menu:

```
┌─────────────────────────────────────────────────────┐
│  💡 Do any of these people settle together?         │
│     (e.g. couples sharing a bank account)           │
│                                        [Add group]  │
└─────────────────────────────────────────────────────┘
```

Tapping "Add group" opens a 2-step flow (see SC03 for the full narrative):

1. **Select members** — multi-select from person list
2. **Name the group** — e.g. "Mark & Lisa" → tap "Done"

```
⚡ POST /events/{eid}/pfgs
    {name: "Mark & Lisa", members: [
      {person_id: mark_id, weight: "1.0", modifier: "1.0", is_financial: true},
      {person_id: lisa_id, weight: "1.0", modifier: "1.0", is_financial: true}
    ]}
    → creates non-singleton PFG with per-member weight, modifier, and is_financial fields in a single call
    → member_ids shorthand is still accepted for equal-weight defaults
```

This replaces the old 7-step flow with a simplified 2-step approach. The user never sees the words "PFG" or "singleton."

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

## Orchestration — "Create Composite Group"

```
POST /events/{eid}/groups {name: "Drinkers", type: "composite", member_group_ids: [group_a_id, group_b_id]}
→ creates a composite group from existing groups in a single call (JF-2B)
→ alternative to creating group + adding members individually
```

## Orchestration — "Send Personal Invite" (Admin)

```
→ S06 (Prepare & Share, pre-selected person)
→ S06 generates personal invite link → recipient opens → S18 Invite Landing
```

> **Cross-reference:** Personal invite recipients arrive at S18 (Invite Landing) where they see their balance preview before creating an account. See R04 Path B.

## Smart Defaults

- Colour-coded settlement groups make financial relationships instantly visible at a glance
- Merge suggestions surface proactively when matches exist — admin doesn't have to go looking
- Self-merge available to members who see their own placeholder ("This is me" button)
- Email/phone hints are optional on add — useful for identity resolution but not required
- Persons sorted: active first, then placeholders
- **Inline split preview** appears automatically after adding a person when split-pending transactions exist — no extra navigation needed
- **Auto-assigned splits default to equal** — admin can adjust individual expenses inline without leaving S07
- **"Review All Splits"** only appears when there are ≥2 affected expenses — keeps the UI clean for simple cases

## Error States

| Error | Display |
|-------|---------|
| Merge fails (name mismatch for self) | "Names don't match. Ask an admin to merge these records." |
| Participant limit reached (unfunded) | "This event needs funding to add more people" → S14 |
| Remove person with splits | "Can't remove — this person has expenses. Reassign first." |
| PFG reassignment to singleton | "Can't join a solo settlement group. Create a shared one instead." |
| Re-merge already merged person (`ALREADY_MERGED`, 409) | "This person has already been merged into another person" |
