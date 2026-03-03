# S04 — Create Event

**Purpose:** Single-screen event creation with aggressive defaults. Supports "from template" shortcut.
**Visible to:** All authenticated users. Creator becomes admin.
**Rails:** R01 (Onboarding), R08 (Template)
**Scenarios:** SC01, SC03, SC04, SC06, SC10, SC18, SC23, SC24

## Wireframe

```
+-----------------------------+
|  <- New Event               |
+-----------------------------+
|                             |
|  Event Name    [______]     |
|  Description   [______]     |
|  Currency      [AUD v]      |
|  Type    (*) One-off        |
|          ( ) Ongoing        |
|                             |
|  +---------------------+   |
|  |   Create Event       |   |
|  +---------------------+   |
+-----------------------------+
```

## Wireframe — From Template

When the user selects "From template", a template picker replaces the blank form:

```
+-----------------------------+
|  <- New Event               |
+-----------------------------+
|                             |
|  (•) Start fresh            |
|  ( ) From template          |
|                             |
|  [When "From template":]    |
|  Template: [Pub Crew v3 ▾]  |
|                             |
|  ┌─────────────────────┐    |
|  │ Members preview:     │    |
|  │ Macca, Davo, Saz,   │    |
|  │ Thommo, New Sarah    │    |
|  └─────────────────────┘    |
|                             |
|  Event Name    [______]     |
|  Currency      [AUD v]      |
|  Type    (*) One-off        |
|          ( ) Ongoing        |
|                             |
|  +---------------------+   |
|  |   Create Event       |   |
|  +---------------------+   |
+-----------------------------+
```

The member preview shows the template's members so the user confirms who will be included. The event name, currency, and type can still be customised.

## Orchestration — "Create from Template"

```
1. User selects "From template" → GET /templates (if not cached)
2. User picks template from dropdown → template members shown as preview
3. User fills event name, currency, type
4. POST /templates/{template_id}/instantiate
   {event_name, base_currency, event_type}
   → event created with persons, groups, PFGs from template
   → event.source_template_id and source_template_version set
5. → S05 (event dashboard, pre-populated with all template members)
```

## Orchestration — "Create Event"

```
1. POST /events {name, description, currency, event_type, visibility}
   Headers: Idempotency-Key: <uuid> (see A07)
   -> 201 Created
   -> event created (event_type defaults to "singular", visibility defaults to "public")
   -> user gets admin event_role (auto)
   -> user's person auto-created with display_name from profile
   -> default shareable invite code auto-generated
2. -> S05 (event dashboard)
```

The backend auto-creates the user's person and invite code on event creation. The frontend sends one call; the user filled one form.

## Smart Defaults

| Field | Default | Override |
|-------|---------|----------|
| Event Name | (required, focused) | — |
| Description | (optional, empty) | Free text |
| Currency | AUD (locale-detected) | Dropdown of supported currencies |
| Event Type | singular (One-off) | Toggle: One-off / Ongoing |
| Visibility | public | Not shown on creation — available in event settings |

- Single required field: event name
- Description is optional — most casual events don't need one
- Currency defaults to AUD based on locale; can be overridden but rarely is
- Creator's person uses their profile display_name automatically
- "Start fresh" selected by default — "From template" is an opt-in shortcut for returning users
- Template dropdown only shown when "From template" is selected — fetches `GET /templates` on first open
- Member preview updates immediately when a different template is selected

## Error States

| Error | Display |
|-------|---------|
| Empty event name | Inline: "Give your event a name" |
| Network error | "Couldn't create event. Try again." with retry button |
