# R08 — Template Rail

**Purpose:** Create, manage, and instantiate group templates into events.
**Primary persona:** Returning user with recurring groups (e.g. weekly pub dinner, housemates).

## Rail Path

```
S03 Home / S15 Profile
  │
  ├──── "My Templates" ────→ S19 My Templates
  │                            │
  │                     ┌──────┼──────┐
  │                     │             │
  │               (create new)   (manage existing)
  │                     │             │
  │                     ▼             ▼
  │               S19 Create    S19 Detail/Edit
  │               Template      Template
  │                     │             │
  │                     └──────┬──────┘
  │                            │
  │                     "Use This Template"
  │                            │
  │                            ▼
  │                     S04 Create Event
  │                     (template pre-selected)
  │                            │
  │                            ▼
  │                     S05 Event Dashboard
  │                     (pre-populated with
  │                      template members)
  │                     [transfer to R02, R04]
  │
  ├──── S05 → "Save as Template" ──→ S19 (template created)
  │     (reverse instantiation)
  │
  └──── S04 → "From template" ──→ S05
        (direct instantiation from create event)
```

## Reverse Instantiation Path

Admin users can save an existing event's group structure as a template:

```
S05 Event Dashboard (admin)
  │
  ├──── overflow menu → "Save as Template"
  │     ⚡ POST /events/{eid}/save-as-template {template_name}
  │     → template created with event's persons, groups, PFG structure
  │     → toast: "Template saved!" [View →]
  │
  └──── [View →] → S19 (new template detail)
```

## Transfer Points

| From | Condition | To |
|------|-----------|-----|
| S03 → S19 | User taps "My Templates" | Template management |
| S15 → S19 | User taps "My Templates" (profile) | Template management |
| S19 → S04 | User taps "Use This Template" | R01 continues — event creation with template pre-selected |
| S04 → S05 | "From template" → create event | R01 continues — event dashboard pre-populated |
| S05 → S19 | Admin taps "Save as Template" | Reverse instantiation |

## Key Orchestration Sequence

The "template to event" path triggers these backend calls:

```
GET /templates                                # 1. List user's templates (S19 or S04 picker)
POST /templates/{id}/instantiate              # 2. Create event from template
  {event_name, base_currency, event_type}
  → event + persons + groups + PFGs created
```

Two API calls. User picked a template and typed an event name. Everyone else is automatic.

## Scenarios Using This Rail

- SC23 (Template Lifecycle) — full path: save from event → edit → share → instantiate
