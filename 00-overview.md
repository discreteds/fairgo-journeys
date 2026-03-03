# Fair Go Web UI — User Journey Design

**Date:** 2026-02-27
**Status:** Draft
**Author:** Claude Code + Nathaniel Ramm

## Purpose

Map the user journeys for a web UI sitting on top of the Fair Go expense-splitting backend (72 endpoints, complex domain model). The goal: make simple UI interactions trigger orchestrated sequences of backend operations with smart defaults, so the complex backend feels simple to end users.

## Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Primary format | Screen-centric specs | Maps directly to component specs for implementation; eliminates wireframe duplication |
| Secondary formats | Scenarios + Journey Rails | Cross-validate screen specs; scenarios test narrative completeness, rails test navigation coherence |
| Target platform | Responsive, mobile-primary | Stacked layouts that expand on desktop. Most expense-splitting apps go this route |
| User personas | Both admin and member | Role-aware UI: same screens, different capabilities visible |
| PFG exposure | "Settlement Groups" | Hide domain complexity behind a friendlier name with colour-coded icons |
| Default philosophy | Aggressive defaults | Auto-split equally, auto-assign to "everyone", single-screen expense entry. User corrects exceptions |

## Conventions

- **Screen specs** (S##) in `screens/` are the source of truth — detailed wireframes, orchestrations, states
- **Scenarios** (SC##) in `scenarios/` are lightweight narratives that reference screen IDs
- **Rails** (R##) in `rails/` are linear sequences of screen IDs with transfer points
- All three reference the same screen IDs, so they cross-validate
- Orchestration chains document exactly which API calls fire behind a single UI action

## Glossary

| UI Term | Backend Concept | Explanation |
|---------|----------------|-------------|
| Settlement Group | Primary Financial Group (PFG) | Who settles as one unit. Colour-coded in UI |
| Person | PERSON | Financial participant in an event (may or may not have an account) |
| Group | GROUP | Named collection of persons for consumption splitting |
| Expense | TRANSACTION + LINE_ITEM(s) | Envelope with one or more line items |
| Line Item | LINE_ITEM + LINE_ITEM_SPLIT(s) | Atomic financial unit with per-person weights |
| Balance | Position (PFG net) | How much a settlement group owes or is owed |
| FX Rate | FX_RATE | Exchange rate record stored per event. Used when expenses are recorded in a currency different from the event's base currency. Contains rate, source, and timestamp |
| Modification Request | MODIFICATION_REQUEST | A request from a member to make a change that requires admin approval (e.g., add person, edit expense). Created via `POST /modification-requests`, approved/rejected by admin with auto-apply |
| Audit Log | AUDIT_LOG | Chronological record of all significant actions in an event. Admin-only access via `GET /events/{eid}/audit-log` |

## Document Map

```
ui-journeys/
├── 00-overview.md                    ← you are here
├── 01-navigation-map.md              # Master screen graph + journey rails
│
├── screens/                          # PRIMARY: Screen-Centric specs
│   ├── S01-welcome.md
│   ├── S02-register-login.md
│   ├── S03-home.md
│   ├── S04-create-event.md
│   ├── S05-event-dashboard.md
│   ├── S06-invite-join.md
│   ├── S07-manage-people.md
│   ├── S08-manage-groups.md
│   ├── S09-add-expense.md
│   ├── S10-expense-detail.md
│   ├── S11-balances.md
│   ├── S12-settle-up.md
│   ├── S13-membership.md
│   ├── S14-event-funding.md
│   ├── S15-profile-settings.md
│   ├── S16-admin-moderation.md
│   └── S17-notifications.md
│
├── scenarios/
│   ├── SC01-alice-organizes-dinner.md
│   ├── SC02-bob-joins-via-invite.md
│   ├── SC03-family-holiday-shared-pfg.md
│   ├── SC04-housemates-monthly-bills.md
│   ├── SC05-settle-and-close.md
│   ├── SC06-free-user-hits-limits.md
│   ├── SC07-charlie-claims-person.md
│   ├── SC08-member-permission-walls.md
│   ├── SC09-multi-event-power-user.md
│   ├── SC10-quota-exhaustion-recovery.md
│   ├── SC11-complex-group-splits.md
│   └── SC12-dispute-modification.md
│
├── rails/
│   ├── R01-onboarding-rail.md
│   ├── R02-expense-rail.md
│   ├── R03-settlement-rail.md
│   ├── R04-invitation-rail.md
│   ├── R05-membership-rail.md
│   └── R06-admin-rail.md
│
└── appendix/
    ├── A01-orchestrations.md
    ├── A02-smart-defaults.md
    ├── A03-role-permissions-ui.md
    ├── A04-sc01-api-changes.md
    ├── A05-gap-closure-changelog.md
    └── A06-gap-analysis-matrix.md
```
