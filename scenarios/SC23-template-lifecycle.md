# SC23 — Template Lifecycle

> **Differentiator:** Recurring groups shouldn't require re-adding 5 people every time. Templates let you save, edit, share, and reuse group setups across events.

**Persona:** Alex — organises recurring Friday pub dinners with the same group. Tired of manually adding members each week.
**Screens:** S05 → S03 → S19 → S19 → S03 → S04 → S05
**Rails:** R08 (Template)

## Design Principle

**"Set up once, reuse forever."** Templates capture the group structure — members, display names, weights, modifiers — so event creation becomes a single tap. When the group changes (someone leaves, someone joins), edit the template once and every future event reflects the new roster.

## Participants

| Person | Status | Role in Template |
|--------|--------|-----------------|
| Alex | Registered user, event admin | Template owner |
| Macca | Registered user | Member |
| Davo | Registered user | Member |
| Thommo | Registered user | Member (removed in edit) |
| Saz | Registered user | Member (shared access) |
| Sarah | Placeholder (email hint only) | Added in edit |

## Narrative

### 1. Save existing event as template (reverse instantiation)

Alex has an existing pub dinner event with 5 people. Instead of recreating this group next week, he saves it as a template.

```
S05 Event Dashboard ("Friday Dinner — Feb 28")
    Alex (admin), Macca, Davo, Thommo, Saz

    → taps overflow menu → "Save as Template"
    → modal: "Template name?" → types "Friday Pub Crew"

    ⚡ POST /events/{eid}/save-as-template
       {template_name: "Friday Pub Crew"}
       → 201 Created
       → template with 5 members, version 1

    Toast: "Template saved!" [View →]
```

### 2. Navigate to Templates and edit

Alex navigates to the template to update the roster. Thommo has moved away and Sarah is joining.

```
S03 Home → taps "My Templates"

S19 My Templates
    Your Templates:
    ┌──────────────────────────┐
    │ Friday Pub Crew    v1    │
    │ 5 members · owner        │
    └──────────────────────────┘

    → taps template

S19 Template Detail
    Name: Friday Pub Crew
    Members:
      👤 Alex    wt: 1  mod: 1  (you, owner)
      👤 Macca   wt: 1  mod: 1
      👤 Davo    wt: 1  mod: 1
      👤 Thommo  wt: 1  mod: 1
      👤 Saz     wt: 1  mod: 1

    → removes Thommo (swipe or tap remove)
    → taps "+ Add Member"
    → searches "Sarah" → no registered user found
    → adds as placeholder: display_name: "Sarah", email_hint: "sarah@example.com"

    → taps "Save Changes"

    ⚡ PUT /templates/{template_id}
       {name: "Friday Pub Crew",
        members: [
          {user_id: alex_id, display_name: "Alex", default_weight: 1, default_modifier: 1.0},
          {user_id: macca_id, display_name: "Macca", default_weight: 1, default_modifier: 1.0},
          {user_id: davo_id, display_name: "Davo", default_weight: 1, default_modifier: 1.0},
          {user_id: saz_id, display_name: "Saz", default_weight: 1, default_modifier: 1.0},
          {display_name: "Sarah", email_hint: "sarah@example.com", default_weight: 1, default_modifier: 1.0}
        ],
        pfg_presets: []}
       → 200 OK, version: 2
```

### 3. Template sharing

Saz (a registered member in the template) now has "member" access automatically. She can see and instantiate the template but cannot edit it.

```
Saz's view on S19:
    Shared With You:
    ┌──────────────────────────┐
    │ Friday Pub Crew    v2    │
    │ 5 members · member       │
    └──────────────────────────┘

    → can tap to view members
    → can tap "Use This Template" → S04
    → cannot edit or delete (owner only)
```

### 4. Instantiate template into new event (next week)

The following Friday, Alex creates a new event using the template.

```
S03 Home → taps "+ New Event"

S04 Create Event
    → selects "From template"
    → template picker: "Friday Pub Crew v2"

    ⚡ GET /templates (loads template list)

    Member preview shown:
      Alex, Macca, Davo, Saz, Sarah (placeholder)

    Event Name: "Friday Dinner — March 7"
    Currency: AUD (default)
    Type: One-off (default)

    → taps "Create Event"

    ⚡ POST /templates/{template_id}/instantiate
       {event_name: "Friday Dinner — March 7",
        base_currency: "AUD",
        event_type: "singular"}
       → 201 Created
       → event with 5 persons, default groups, singleton PFGs
       → event.source_template_id set
       → event.source_template_version = 2
```

### 5. Event dashboard — pre-populated

```
S05 Event Dashboard ("Friday Dinner — March 7")
    5 people · 0 expenses

    People:
      🔴 Alex (you, admin) — solo PFG
      🔴 Macca             — solo PFG
      🔴 Davo              — solo PFG
      🔴 Saz               — solo PFG
      🔴 Sarah (placeholder, ✉ sarah@example.com) — solo PFG

    Alex starts adding expenses immediately.
    No manual person creation needed.

    [transfer to R02 Expense Rail]
```

## Arithmetic Verification

This scenario focuses on the template lifecycle rather than expense splitting. No expenses are created within the scenario, so no arithmetic table is needed. The template instantiation produces:

| Entity | Count | Source |
|--------|-------|--------|
| Persons | 5 | Template members (4 resolved + 1 placeholder) |
| Groups | 1 | Default group from template name |
| PFGs | 5 | Singleton PFGs (no pfg_presets in template) |
| Transactions | 0 | None yet — event is ready for expenses |

## Validates

- **Reverse instantiation:** Save existing event group as template (`POST /events/{eid}/save-as-template`)
- **Template editing:** Remove member, add placeholder member, version auto-bump (v1 → v2)
- **Template sharing:** Registered members get "member" access automatically (view + instantiate, no edit)
- **Placeholder members:** Sarah exists as email hint only — will be resolved when she claims the person
- **Forward instantiation:** Template → event via `POST /templates/{id}/instantiate`
- **Pre-populated event:** Dashboard shows all members immediately, no manual setup needed
- **Template version tracking:** Event records which template version it was created from
- **Home → Templates navigation:** "My Templates" link on S03 navigates to S19

## Key Moment

The payoff is step 4: Alex taps "From template", picks "Friday Pub Crew v2", types a name, and the event is created with all 5 members pre-populated. No searching for users, no adding placeholders one by one, no recreating groups. **One form, one tap, five people.**

> **"Same crew, new night. One tap."**
