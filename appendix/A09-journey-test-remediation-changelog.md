# A09 — Journey Test Remediation Changelog

**Date:** 2026-03-09
**Backend branch:** `004-journey-test-remediation`
**Spec:** `fairgo-central/specs/004-journey-test-remediation/`
**Change Record:** CR-021
**Issues addressed:** 56 (28 API bugs, 10 missing features, 7 test spec errors, 11 improvements)
**Backend tests:** 407 passing

## Summary

| Stage | Name | Screens Affected | Key Changes |
|-------|------|-----------------|-------------|
| US1 | Security & Data Integrity | S05, S16 | Password hash exposure fixed; modification approval auto-applies proposed_changes (not cancel); tier downgrade guard; duplicate pending modification guard |
| US2 | Pending Member Access | S05, S07 | Pending members see event metadata with contact redaction; Tier 3 display_name matching prevents duplicate persons on join |
| US3 | Event Links & Decomposition | S12 | Event link referential integrity + auth + dedup; decompose endpoint validation tightened |
| US4 | Missing Features | S05, S07, S12, S16, S17 | Partial payment lifecycle; sponsor claiming; pending_modification_count; event-scoped activity; modification preview; invite preview financial summary |
| US5 | API Consistency | S14, S16, S17 | Funding gates → 403 UNFUNDED_LIMIT; modification status filter; invite error codes; activity event_id filter |
| US6 | Permissions | S08, S12, S16 | Members create groups; debtors propose settlements; auto-approve targeted invites |

## New Error Codes

| Code | HTTP | When |
|------|------|------|
| UNFUNDED_LIMIT | 403 | Any resource limit hit on unfunded/locked event (replaces 422) |
| INVITE_NOT_FOR_YOU | 403 | Wrong user attempts person-targeted invite |
| DUPLICATE_MODIFICATION | 409 | Pending modification already exists for target entity |
| OVER_QUOTA | 400 | Tier downgrade blocked — funded events exceed target tier limit |

## Screen Spec Changes

### S05 — Event Dashboard
- **pending_modification_count badge** on Admin Extras card
- **Event-scoped activity** card with `GET /events/{eid}/activity`
- **Contact redaction** note on Pending Approval State wireframe

### S07 — Manage People
- **"Claim as Dependent"** orchestration — `POST /claim` with `role: "sponsor"`

### S08 — Manage Groups
- **Member group creation** — permission loosened from admin-only to any member

### S12 — Settle Up
- **New wireframe: Partially Paid Settlement** — progress bar, payment history, Pay Remaining / Pay Other Amount buttons
- **Partial payment orchestration** — `POST /pay` with optional `amount` field
- **Updated settlement status flow** — confirmed → partially_paid → paid
- **Debtor settlement proposal** — members create settlements from own PFG

### S16 — Admin Moderation
- **Modification preview** — `GET /preview` with current/proposed diff and position deltas
- **Status filter** — `?status=` query parameter, segmented control UI
- **Auto-approve targeted invites** — skip pending queue when target_person_id matches

### S17 — Notifications / Activity
- **Event-scoped feed** — `GET /events/{eid}/activity` with page/since params
- **Cross-event event_id filter** — `?event_id=` on existing feed

## Design Deviations

### Funding Gate HTTP Status
**Original:** 422 VALIDATION_ERROR for all limit enforcement.
**Implemented:** 403 UNFUNDED_LIMIT with `suggestion` field pointing to funding endpoint.
**Rationale:** Limits are authorization boundaries (you don't have permission to exceed free tier), not validation failures. 403 is semantically correct and lets the frontend distinguish "fix your input" (422) from "upgrade your plan" (403).

### Partial Payment (New State)
**Original spec:** Settlement lifecycle was proposed → confirmed → paid → voided.
**Implemented:** Added `partially_paid` between confirmed and paid.
**Rationale:** Real-world group settlements are often paid in installments. The `paid_total` and `remaining` fields track progress without requiring multiple settlement records.

## Cross-References

- **Backend implementation:** `fairgo-backend` branch `004-journey-test-remediation`
- **Test spec corrections:** `fairgo-journey-tests` branch `004-journey-test-remediation` (7 files)
- **Central spec:** `fairgo-central/specs/004-journey-test-remediation/`
- **Change Record:** `fairgo-central/05.changes/active/CR-021-journey-test-remediation.md`
