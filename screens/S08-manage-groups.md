# S08 — Manage Groups

**Purpose:** Create/edit consumption groups. View settlement groups. Expand groups into splits.
**Visible to:** All can view. Admin can create/edit.
**Rails:** R06
**Scenarios:** SC03

## Wireframe

```
┌──────────────────────────────┐
│  ← Groups · Bali Trip        │
├──────────────────────────────┤
│                              │
│  Consumption Groups          │
│  ┌──────────────────────────┐│
│  │ Everyone            (6)  ││
│  │ Alice, Bob, Carol,       ││
│  │ Dave, Eve, Frank         ││
│  ├──────────────────────────┤│
│  │ Drinkers            (4)  ││
│  │ Alice, Bob, Dave, Frank  ││
│  └──────────────────────────┘│
│                              │
│  Settlement Groups           │
│  ┌──────────────────────────┐│
│  │ 🔵 Alice & Partner  (2) ││
│  │ Alice, Carol             ││
│  ├──────────────────────────┤│
│  │ 🔴 Bob (solo)            ││
│  │ 🔴 Dave (solo)           ││
│  │ 🔴 Eve (solo)            ││
│  │ 🔴 Frank (solo)          ││
│  └──────────────────────────┘│
│                              │
│       ( + New Group )        │  ← admin only
└──────────────────────────────┘
```

The screen visually separates two types of groups:

- **Consumption groups** (top): Named by purpose. Used as shortcuts when splitting expenses. E.g. "Drinkers" = the people who share alcohol costs.
- **Settlement groups** (bottom): Colour-coded 🔵/🔴. Represent PFGs. Managed via S07 ("Change Settlement Group"), not directly here.

## Wireframe — Group Detail (Admin Tap)

```
┌──────────────────────────────┐
│  ← Drinkers                  │
├──────────────────────────────┤
│                              │
│  Members                     │
│  ☑ Alice                     │
│  ☑ Bob                       │
│  ☐ Carol                     │
│  ☑ Dave                      │
│  ☐ Eve                       │
│  ☑ Frank                     │
│                              │
│  [Save Changes]              │
│                              │
│  [Delete Group]              │
└──────────────────────────────┘
```

## Orchestration — Page Load

```
1. GET /events/{eid}/groups     → all groups with member lists
2. GET /events/{eid}/persons    → person list (for PFG mapping to separate
                                  consumption vs settlement groups)
```

Groups where a person's PFG points to them are settlement groups. All others are consumption groups.

## Orchestration — "+ New Group" (Admin)

```
1. Prompt: group name + select members (checkbox list of all persons)
2. POST /events/{eid}/groups {name, is_singleton: false}
3. For each selected member:
   POST /events/{eid}/groups/{gid}/members {person_id}
4. Refresh group list
```

## Orchestration — Edit Group Members (Admin)

```
1. Toggle member checkboxes
2. For newly checked:
   POST /events/{eid}/groups/{gid}/members {person_id}
3. For newly unchecked:
   DELETE /events/{eid}/groups/{gid}/members/{person_id}
4. Refresh group detail
```

## Orchestration — Delete Group (Admin)

```
1. Confirm dialog: "Delete 'Drinkers'? This won't affect existing expenses."
2. DELETE /events/{eid}/groups/{gid}
3. Refresh group list
```

## Smart Defaults

- An "Everyone" group is shown derived from all persons even if no explicit group exists — useful as default split target
- Groups appear as quick-select options on S09 (Add Expense) "Split between" picker
- Settlement groups can't be created/deleted here — they're managed via person PFG assignment on S07
- Settlement group section is read-only with a hint: "Manage settlement groups in People"

## Group Expand (Future Feature)

The backend supports `GET /events/{eid}/groups/{gid}/expand` which returns suggested person-level splits from group members. This could power a "Expand group" action on S09 that pre-fills per-person weights from a group selection.

## Error States

| Error | Display |
|-------|---------|
| Group limit reached (unfunded) | "This event needs funding to add more groups" → S14 |
| Delete group with active splits | Backend allows this — splits reference persons, not groups |
| Add member to singleton group | Blocked by backend — "Can't add members to a solo group" |
