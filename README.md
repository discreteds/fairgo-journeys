# Fair Go Journeys

User journey documentation for the **Fair Go** web SaaS expense-splitting application.

## Quick Start

| I want to... | Go here |
|---|---|
| Understand the project | [Project Overview](00-overview.md) |
| See how screens connect | [Interactive Navigation Map](nav-map-interactive.md) |
| Read screen specifications | [S05 Event Dashboard](screens/S05-event-dashboard.md) (hub screen) |
| Walk through a user story | [SC01 Alice Organizes Dinner](scenarios/SC01-alice-organizes-dinner.md) |
| Follow a navigation rail | [R01 Onboarding Rail](rails/R01-onboarding-rail.md) |
| Check API orchestrations | [A01 Orchestrations](appendix/A01-orchestrations.md) |

## Document Structure

### Screens (17)

Detailed specifications for each screen, including ASCII wireframes, component breakdowns, state variants, and API orchestration details. Organized by function:

| Group | Screens | Purpose |
|-------|---------|---------|
| Onboarding | S01-S04 | Welcome, auth, home, event creation |
| Core | S05-S08 | Dashboard (hub), invites, people, groups |
| Expenses | S09-S12 | Add expense, detail, balances, settle up |
| Account & Admin | S13-S17 | Membership, funding, profile, moderation, notifications |

### Scenarios (6)

End-to-end narratives following a specific user through a complete workflow. Good for stakeholder walkthroughs and cross-validating screen specs.

### Journey Rails (6)

Linear sequences of screen IDs representing the shortest path through a workflow. Useful for testing and implementation planning.

### Appendix (3)

Reference material covering orchestration chains, smart default rules, and role-based permission matrices.

## Glossary

| UI Term | Backend Concept | Description |
|---------|----------------|-------------|
| Settlement Group | Primary Financial Group (PFG) | Who settles as one unit |
| Person | PERSON | Financial participant in an event |
| Group | GROUP | Named collection of persons |
| Expense | TRANSACTION + LINE_ITEM(s) | Envelope containing line items |
| Line Item | LINE_ITEM + LINE_ITEM_SPLIT(s) | Atomic unit with per-person weights |
| Balance | Position (PFG net) | How much a settlement group owes or is owed |
