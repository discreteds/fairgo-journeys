# S19 — My Templates

**Purpose:** Create, edit, and manage reusable group templates for quick event setup.
**Visible to:** All authenticated users (own templates + shared templates).
**Rails:** R08 (Template)
**Scenarios:** SC23

## Wireframe — Template List

```
┌──────────────────────────────┐
│  ← My Templates         [+] │
├──────────────────────────────┤
│                              │
│  Your Templates              │
│  ┌──────────────────────────┐│
│  │ Friday Pub Crew    v3    ││
│  │ 5 members · owner        ││
│  ├──────────────────────────┤│
│  │ McKinnon Household  v1   ││
│  │ 2 members · owner        ││
│  └──────────────────────────┘│
│                              │
│  Shared With You             │
│  ┌──────────────────────────┐│
│  │ Book Club           v2   ││
│  │ 6 members · member       ││
│  └──────────────────────────┘│
│                              │
│  [Events] [Activity] [Me]   │
└──────────────────────────────┘
```

## Wireframe — Empty State

```
┌──────────────────────────────┐
│  ← My Templates              │
├──────────────────────────────┤
│                              │
│                              │
│   No templates yet.          │
│                              │
│   Save a group from an       │
│   event, or create one here. │
│                              │
│  ┌────────────────────────┐  │
│  │   + Create Template     │  │
│  └────────────────────────┘  │
│                              │
│  [Events] [Activity] [Me]   │
└──────────────────────────────┘
```

## Wireframe — Template Detail / Edit

```
┌──────────────────────────────┐
│  ← Friday Pub Crew     v3   │
├──────────────────────────────┤
│                              │
│  Template Name               │
│  [Friday Pub Crew_______]    │
│                              │
│  Members                     │
│  ┌──────────────────────────┐│
│  │ 👤 Macca    wt: 1  mod: 1││
│  │    james@example.com      ││
│  ├──────────────────────────┤│
│  │ 👤 Davo     wt: 1  mod: 1││
│  │    david@example.com      ││
│  ├──────────────────────────┤│
│  │ 👤 Saz      wt: 1  mod: 1││
│  │    saz@example.com        ││
│  ├──────────────────────────┤│
│  │ 👤 Thommo   wt: 1  mod: 1││
│  │    ✉ thommo@example.com   ││
│  │    (placeholder)          ││
│  ├──────────────────────────┤│
│  │ 👤 New Sarah wt: 1 mod: 1││
│  │    ✉ sarah@example.com    ││
│  │    (placeholder)          ││
│  └──────────────────────────┘│
│                              │
│  [+ Add Member]              │
│                              │
│  PFG Presets                 │
│  ┌──────────────────────────┐│
│  │ (none — singletons on    ││
│  │  instantiation)           ││
│  └──────────────────────────┘│
│                              │
│  ┌────────────────────────┐  │
│  │   Save Changes          │  │
│  └────────────────────────┘  │
│                              │
│  🗑️ Delete Template          │  ← owner only
│                              │
└──────────────────────────────┘
```

## Wireframe — Create Template

```
┌──────────────────────────────┐
│  ← New Template              │
├──────────────────────────────┤
│                              │
│  Template Name               │
│  [_________________________] │  ← focused
│                              │
│  Members                     │
│  ┌──────────────────────────┐│
│  │ (you are added            ││
│  │  automatically)           ││
│  └──────────────────────────┘│
│                              │
│  [+ Add Member]              │
│                              │
│  Add members by searching    │
│  registered users or adding  │
│  placeholders with an email  │
│  hint.                       │
│                              │
│  ┌────────────────────────┐  │
│  │   Create Template       │  │
│  └────────────────────────┘  │
│                              │
└──────────────────────────────┘
```

## Wireframe — Add Member Modal

```
┌──────────────────────────────┐
│  Add Member                  │
├──────────────────────────────┤
│                              │
│  Search users:               │
│  [Search by name or email__] │
│                              │
│  ┌──────────────────────────┐│
│  │ 👤 Sarah (sarah@...)     ││
│  │ 👤 Sam (sam@...)         ││
│  └──────────────────────────┘│
│                              │
│  — or —                      │
│                              │
│  Add placeholder:            │
│  Display Name  [______]      │
│  Email Hint    [______]      │
│                              │
│  [Add as Placeholder]        │
│                              │
└──────────────────────────────┘
```

## Orchestration — Page Load

```
1. GET /templates
   → returns user's templates (owned + shared via access list)
   → each template includes: id, name, version, member_count, role (owner/member)
```

Single API call. The list includes both templates the user owns and templates shared with them (via the access array).

## Orchestration — View Template Detail

```
1. GET /templates/{template_id}
   → returns template with full member list, PFG presets, access list
   → includes pfg_presets: [{name, member_slots: [slot_label, ...]}]
```

## Orchestration — Create Template

```
1. POST /templates
   {name, members: [{user_id?, display_name, email_hint?, default_weight, default_modifier}],
    pfg_presets: [{name, member_slots: ["slot_label_1", "slot_label_2"]}]}
   → 201 Created
   → template created, creator becomes owner
   → registered users in member list get "member" access automatically
   → pfg_presets define settlement group structures to recreate on instantiation
2. → template detail view (stays on S19)
```

## Orchestration — Edit Template

```
1. PUT /templates/{template_id}
   {name, members: [...], pfg_presets: [...]}
   → 200 OK
   → full replacement of members and PFG presets
   → version auto-incremented
   → access list updated (new registered users get member access)
```

Owner only. Full replacement — the client sends the complete member list, not a diff.

## Orchestration — Delete Template

```
1. DELETE /templates/{template_id}
   → 204 No Content
   → template removed for all users (owner only)
```

## Orchestration — Template Row Tap

```
→ template detail/edit view (within S19)
```

## Actions

| Action | Who | Target | Orchestration |
|--------|-----|--------|--------------|
| [+] / Create Template | All | Create view (S19) | `POST /templates` |
| Template row tap | All | Detail/edit view (S19) | `GET /templates/{id}` |
| Save Changes | Owner | Stay on S19 | `PUT /templates/{id}` |
| Delete Template | Owner | Back to list (S19) | `DELETE /templates/{id}` |
| + Add Member | Owner | Add member modal | Client-side |
| Use This Template | All (with access) | → S04 | Navigation (pre-selects template) |

## Smart Defaults

- Template name focused on create
- Members default to weight 1.0, modifier 1.0
- No PFG presets by default — each member gets a singleton PFG on instantiation
- **PFG presets** (CR-018): When saving an event as template, existing non-singleton PFGs are captured as `pfg_presets` with member slot labels. On instantiation, these presets recreate the PFG structure in the new event.
- **Template provenance** (CR-018): Events created from templates record `source_template_id` and `source_template_version` for traceability
- Sort order matches entry order (drag to reorder)
- Templates split into "Your Templates" (owner) and "Shared With You" (member) sections
- Version badge shown on each template card (v1, v2, etc.)
- Creator is auto-added as first member
- Registered users in member list auto-receive "member" access (can instantiate but not edit)
- Placeholder members show email hint and "(placeholder)" label
- "Use This Template" shortcut visible on detail view for quick instantiation via S04

## Error States

| Error | Display |
|-------|---------|
| Empty template name | Inline: "Give your template a name" |
| No members added | "Add at least one member to your template" |
| Delete non-owned | Button hidden for non-owners |
| Template not found | "This template doesn't exist or you don't have access" → back to list |
| Network error | "Couldn't load templates. Try again." with retry button |
