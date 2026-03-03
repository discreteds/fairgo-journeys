# A06 — Gap Analysis Matrix

**Date:** 2026-03-03
**Scope:** Journey documentation vs implemented API changes (CR-001, CR-002, Journey Extended Fixes, New User Paths)
**Purpose:** Per-file checklists cross-referenced to change sources, prioritised for implementation waves

---

## Change Source Registry

| ID | Name | Scope |
|----|------|-------|
| CR-001 | Idempotency & Robustness | `Idempotency-Key` header, `Idempotency-Replayed` response header, `recoverable` error field, frontend retry logic |
| CR-002 | Multi-Currency & Time | FX_RATE entity, `occurred_at` on transactions, `fx_rate_used`/`fx_source` on line items, cross-currency settlement fields, write-off entity |
| JF-1A | Split Reassignment & Filtering | `POST reassign-splits`, match suggestion filtering, person query `role` param |
| JF-1B | Settlement & Participant Guards | Settlement guard on closed events, 5-person unfunded limit, voided settlement visibility |
| JF-1C | Person Creation Access | Person creation admin-only, members use modification requests |
| JF-2A | Settlement Suggestions & Split Editing | Server-side settlement suggestions endpoint, dedicated split editing endpoint |
| JF-2B | Pending Members & Discovery | `GET /events/pending`, funding discovery (data not 404), event limits object, composite group creation |
| JF-3 | Modification Request System | 6 endpoints (create, list, get, approve, reject, withdraw), auto-apply on approve |
| JF-4A | Free Tier & Invite Tracking | Free tier auto-creation on register, invite code usage tracking |
| JF-4B | Activity Feed Improvements | New activity types, `event_name` field, fixed `since` param |
| JF-4C | Schema & Pagination | Currency alias, `person_count`/`transaction_count` summary fields, `limit`/`offset`/`sort` pagination |
| JF-5 | Response Polish | 201 status codes, error `suggestion` field, audit fields (`cancelled_at/by`, `voided_at/by`), display name propagation |
| JF-6 | Audit Log | New entity + `GET /events/{eid}/audit-log` admin-only endpoint |
| NUP | New User Paths | Path D (person-targeted invite auto-claim), sponsorship on join |

---

## Priority Definitions

| Level | Meaning |
|-------|---------|
| **CRITICAL** | Current documentation is factually wrong or misleading |
| **IMPORTANT** | New feature/behaviour not documented; docs are incomplete but not wrong |
| **NICE** | Polish, consistency, or enhancement |

---

## Per-Document Gap Matrix

### Screen Specs

#### S01 Welcome

No changes needed.

#### S02 Register / Login

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Document free tier auto-creation on register | JF-4A | IMPORTANT | [x] |
| 2 | Add `Idempotency-Key` to register/login orchestrations | CR-001 | NICE | [ ] |

#### S03 Home

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add pending events section (`GET /events/pending`) | JF-2B | IMPORTANT | [x] |
| 2 | Add `person_count`/`transaction_count` summary fields | JF-4C | NICE | [x] |

#### S04 Create Event

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add `Idempotency-Key` mention to create event orchestration | CR-001 | NICE | [ ] |
| 2 | Note 201 status code on successful creation | JF-5 | NICE | [ ] |

#### S05 Event Dashboard

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add multi-currency balance display variant | CR-002 | CRITICAL | [x] |
| 2 | Add event limits display for unfunded events | JF-2B | IMPORTANT | [x] |
| 3 | Add Audit Log link for admin | JF-6 | IMPORTANT | [x] |
| 4 | Document `voided_at`/`voided_by` audit fields on settlements | JF-1B, JF-5 | IMPORTANT | [x] |

#### S06 Prepare & Share

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add invite code usage tracking display | JF-4A | IMPORTANT | [x] |
| 2 | Add person-targeted invite auto-claim flow (Path D) | NUP | IMPORTANT | [x] |
| 3 | Document sponsorship on join | NUP | IMPORTANT | [x] |

#### S07 Manage People

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add member modification request path for person addition (members can't create directly, only suggest) | JF-1C, JF-3 | CRITICAL | [x] |
| 2 | Document merged persons excluded from match suggestions | JF-1A | IMPORTANT | [x] |
| 3 | Add split line-item editing endpoint | JF-2A | IMPORTANT | [x] |
| 4 | Add composite group creation (single endpoint) | JF-2B | NICE | [ ] |

#### S08 Manage Groups

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Update group creation to composite endpoint | JF-2B | IMPORTANT | [x] |

#### S09 Add Expense

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add `Idempotency-Key` to save expense (THE financial mutation) | CR-001 | CRITICAL | [x] |
| 2 | Add multi-currency FX support — current "not overridable" is wrong | CR-002 | CRITICAL | [x] |
| 3 | Add `occurred_at` backdating field | CR-002 | IMPORTANT | [x] |

#### S10 Expense Detail

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Update "Suggest Change" from "future feature" to implemented | JF-3 | CRITICAL | [x] |
| 2 | Add `occurred_at` display | CR-002 | IMPORTANT | [x] |
| 3 | Add FX rate display for multi-currency line items | CR-002 | IMPORTANT | [x] |
| 4 | Add `cancelled_at`/`cancelled_by` audit fields | JF-5 | IMPORTANT | [x] |
| 5 | Document split editing via dedicated endpoint | JF-2A | IMPORTANT | [x] |

#### S11 Balances

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add multi-currency balance display (assumes single currency currently) | CR-002 | CRITICAL | [x] |
| 2 | Add cross-currency settlement amounts | CR-002 | IMPORTANT | [x] |

#### S12 Settle Up

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Settlement suggestions now server-side (documented as "client-side") | JF-2A | CRITICAL | [x] |
| 2 | Add `Idempotency-Key` to settlement mutations (create/confirm/pay/void) | CR-001 | CRITICAL | [x] |
| 3 | Add settlement guard on closed events | JF-1B | IMPORTANT | [x] |
| 4 | Add `voided_at`/`voided_by` on voided settlements | JF-1B, JF-5 | IMPORTANT | [x] |
| 5 | Add cross-currency settlement fields | CR-002 | IMPORTANT | [x] |
| 6 | Add write-off entity for rounding remainders | CR-002 | NICE | [x] |

#### S13 Membership

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Document free tier auto-creation | JF-4A | IMPORTANT | [x] |

#### S14 Event Funding

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Funding discovery returns data not 404 | JF-2B | IMPORTANT | [x] |
| 2 | Add event limits object display | JF-2B | IMPORTANT | [x] |

#### S15 Profile / Settings

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Note display name propagation to event persons | JF-5 | NICE | [ ] |

#### S16 Admin Moderation

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add full modification request queue system (6 endpoints, auto-apply) | JF-3 | CRITICAL | [x] |
| 2 | Add audit log access | JF-6 | IMPORTANT | [x] |

#### S17 Notifications (Activity Feed)

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add new activity types from Phase 4B | JF-4B | IMPORTANT | [x] |
| 2 | Add `event_name` field | JF-4B | IMPORTANT | [x] |
| 3 | Fix `since` param documentation | JF-4B | IMPORTANT | [x] |

---

### Scenarios

#### SC01 Alice Organizes Dinner

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add `Idempotency-Key` mention on expense creation | CR-001 | NICE | [ ] |

#### SC02 Bob Joins via Invite

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Document free tier auto-creation on register | JF-4A | IMPORTANT | [x] |

#### SC03 Family Holiday Shared PFG

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Update composite group creation orchestration | JF-2B | IMPORTANT | [x] |

#### SC04 Housemates Monthly Bills

No changes needed.

#### SC05 Settle and Close

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Settlement suggestions now server-side | JF-2A | CRITICAL | [x] |
| 2 | Add settlement guard on closed events | JF-1B | IMPORTANT | [x] |

#### SC06 Free User Hits Limits

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Document free tier auto-creation | JF-4A | IMPORTANT | [x] |

#### SC07 Charlie Claims Person

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add `Idempotency-Key` mention on claim | CR-001 | NICE | [ ] |

#### SC08 Member Permission Walls

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Person creation admin-only enforcement (show member alternative) | JF-1C | IMPORTANT | [x] |
| 2 | Expand modification request flow with full 6-endpoint API | JF-3 | IMPORTANT | [x] |

#### SC09 Multi-Event Power User

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add pending events visibility | JF-2B | IMPORTANT | [x] |

#### SC10 Quota Exhaustion Recovery

No changes needed.

#### SC11 Complex Group Splits

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add composite group creation endpoint | JF-2B | NICE | [ ] |

#### SC12 Dispute and Modification

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Remove separate PATCH call after resolve — auto-apply does both | JF-3 | CRITICAL | [x] |
| 2 | Expand modification request API from 2 to 6 endpoints | JF-3 | IMPORTANT | [x] |

---

### Journey Rails

#### R01 Onboarding Rail

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add free tier auto-creation note | JF-4A | IMPORTANT | [x] |

#### R02 Expense Rail

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add `Idempotency-Key` note on expense save | CR-001 | NICE | [ ] |

#### R03 Settlement Rail

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add settlement guard on closed events | JF-1B | IMPORTANT | [x] |

#### R04 Invitation Rail

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add Path D (person-targeted auto-claim) | NUP | IMPORTANT | [x] |
| 2 | Add sponsorship on join | NUP | IMPORTANT | [x] |

#### R05 Membership Rail

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Note free tier auto-creation on join | JF-4A | NICE | [ ] |

#### R06 Admin Rail

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add modification request handling | JF-3 | IMPORTANT | [x] |
| 2 | Add audit log access | JF-6 | IMPORTANT | [x] |

---

### Appendix

#### A01 Orchestrations

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add `Idempotency-Key` requirement to ALL financial mutation orchestrations | CR-001 | CRITICAL | [x] |
| 2 | Add settlement suggestions endpoint to S12 page load | JF-2A | CRITICAL | [x] |
| 3 | Add modification request orchestrations (6 endpoints) | JF-3 | IMPORTANT | [x] |
| 4 | Add audit log orchestration | JF-6 | IMPORTANT | [x] |
| 5 | Add FX rate management orchestrations | CR-002 | IMPORTANT | [x] |
| 6 | Add split editing endpoint | JF-2A | IMPORTANT | [x] |
| 7 | Update S03 page load (add pending events call) | JF-2B | NICE | [x] |

#### A02 Smart Defaults

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Update S09 currency default — "not overridable" is now wrong for multi-currency | CR-002 | CRITICAL | [x] |
| 2 | Add S09 `occurred_at` default (default: now) | CR-002 | IMPORTANT | [x] |
| 3 | Update S12 settlement suggestions to "server-computed" | JF-2A | IMPORTANT | [x] |

#### A03 Role Permissions UI

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add "Request Add Person" row for members on S07 | JF-1C, JF-3 | CRITICAL | [x] |
| 2 | Add modification request submission for members on S10, S16 | JF-3 | IMPORTANT | [x] |
| 3 | Add audit log as admin-only | JF-6 | IMPORTANT | [x] |

#### A04 SC01 API Changes

No changes needed (historical document).

#### A05 Gap Closure Changelog

No changes needed (historical document).

---

### Root Documents

#### 00-overview.md

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Update document map (missing SC07–SC12, A04–A06) | — | IMPORTANT | [x] |
| 2 | Add glossary entries: FX_RATE, Modification Request, Audit Log | CR-002, JF-3, JF-6 | NICE | [x] |

#### 01-navigation-map.md

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Fix S17 entry — says "no endpoint yet" but endpoint is implemented | JF-4B | CRITICAL | [x] |
| 2 | Add SC07–SC12 to cross-validation matrix | — | IMPORTANT | [x] |

#### _sidebar.md

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add SC07–SC12 entries | — | IMPORTANT | [x] |
| 2 | Add A04, A05, A06 entries | — | IMPORTANT | [x] |

#### nav-map-interactive.md

No changes needed.

---

## Documents Confirmed No Changes Needed

| Document | Note |
|----------|------|
| S01 Welcome | — |
| SC04 Housemates Monthly Bills | — |
| SC10 Quota Exhaustion Recovery | — |
| A04 SC01 API Changes | Historical |
| A05 Gap Closure Changelog | Historical |
| nav-map-interactive.md | — |

---

## New Documents Needed

| Document | Reason | Source | Priority |
|----------|--------|--------|----------|
| **SC13: Person-Targeted Invite Auto-Claim** | Path D is undocumented — targeted invite skips merge | NUP | IMPORTANT |
| **SC14: Sponsorship on Join** | Sponsor brings dependents on join — undocumented flow | NUP | IMPORTANT |
| **A07 Idempotency Guide** (or section in A01) | Idempotency touches every mutation — needs reference | CR-001 | IMPORTANT |
| **A08 Multi-Currency Reference** (or section in A01) | FX rates, conversion, write-offs — complex feature | CR-002 | IMPORTANT |
| **Audit Log screen/section** (S18 or section in S16) | New entity + endpoint needs UI specification | JF-6 | IMPORTANT |

---

## Priority Summary

### CRITICAL — 12 items (factually wrong)

| # | Document | Issue | Source |
|---|----------|-------|--------|
| 1 | S05 | No multi-currency balance variant | CR-002 |
| 2 | S07 | Missing member modification request path for person addition | JF-1C, JF-3 |
| 3 | S09 | Multi-currency FX contradicts "not overridable" | CR-002 |
| 4 | S10 | "Suggest Change" listed as "future feature" but is implemented | JF-3 |
| 5 | S11 | Single-currency balance display, now supports multi-currency | CR-002 |
| 6 | S12 | Settlement suggestions documented as "client-side", now server-side | JF-2A |
| 7 | S16 | Missing entire modification request queue system | JF-3 |
| 8 | SC05 | Settlement suggestions documented as client-side | JF-2A |
| 9 | SC12 | Shows separate PATCH after resolve — auto-apply does both | JF-3 |
| 10 | A01 | No idempotency headers on any financial mutation orchestrations | CR-001 |
| 11 | A02 | S09 currency "not overridable" is now wrong | CR-002 |
| 12 | A03 | Missing member modification request alternative for person creation | JF-1C, JF-3 |
| — | 01-nav-map | S17 "no endpoint yet" is wrong | JF-4B |

### IMPORTANT — 35 items (missing features)

See per-document tables above. Major clusters:

- **Multi-currency display** (S05, S09, S10, S11, S12) — CR-002
- **Modification requests** (S07, S08, S10, S16, SC08, SC12, R06, A01) — JF-3
- **Free tier auto-creation** (S02, S13, SC02, SC06, R01) — JF-4A
- **New User Paths** (S06, R04) — NUP
- **Activity feed** (S17) — JF-4B
- **Pending events** (S03, SC09) — JF-2B
- **Audit log** (S05, S16, R06, A01) — JF-6
- **Structural** (_sidebar.md, 00-overview.md, 01-navigation-map.md)

### NICE — 15+ items (polish)

201 status codes, `recoverable` field, `suggestion` field, pagination params, display name propagation, currency aliases, composite group creation, idempotency mentions on non-financial calls.

---

## Recommended Implementation Waves

### Wave 1 — Critical Accuracy Fixes

Fix documentation that is factually wrong. Highest value, prevents misinformation.

**Targets:** S07, S09, S10, S11, S12, S16, SC05, SC12, A01 (idempotency), A02, A03, 01-nav-map

**Sources:** CR-001, CR-002, JF-2A, JF-3, JF-1C, JF-4B

### Wave 2 — Multi-Currency

Add multi-currency support across all affected screens and appendix.

**Targets:** S05, S09, S10, S11, S12 (cross-currency), A01 (FX orchestrations), new A08 or A01 section

**Sources:** CR-002

### Wave 3 — New Features

Document undocumented features and user paths.

**Targets:** S02, S03, S06, S13, S14, S17, R04, R06, new SC13, new SC14, new A07 or A01 section

**Sources:** JF-2B, JF-4A, JF-4B, JF-6, NUP

### Wave 4 — Structural

Update navigation, cross-references, and document registry.

**Targets:** _sidebar.md, 00-overview.md, 01-navigation-map.md (matrix)

### Wave 5 — Polish

All NICE-TO-HAVE items: 201 status codes, `recoverable` field, `suggestion` field, pagination params, display name propagation, currency aliases.

---

## Cross-References

- CR-001 spec: `fairgo-central/06.spec/CR-001-idempotency-robustness.md`
- CR-002 spec: `fairgo-central/06.spec/CR-002-multi-currency-time.md`
- Journey Extended Fixes design: `fairgo-backend/docs/plans/journey-extended-fixes/`
- New User Paths: `fairgo-central/03.user-journey-mods/new-user-paths.md`
- A05 Gap Closure Changelog: `appendix/A05-gap-closure-changelog.md`
- Backend API spec: `fairgo-central/06.spec/expense-splitter-api.md`
