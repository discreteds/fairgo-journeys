# A06 — Gap Analysis Matrix

**Date:** 2026-03-03
**Last updated:** 2026-03-03 — Wave 1 scenario cross-references added (SC01, SC02, SC03, SC04, SC06 reworked in place; SC13, SC17, SC18, SC19 new; S18 new screen)
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

#### S18 Invite Landing

*New screen — added Wave 1*

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Create S18 Invite Landing screen spec (balance preview, event summary, registration CTA) | NUP, Wave 1 | IMPORTANT | [x] |
| 2 | Document recipient onboarding flow: S18 → S02 Register → S05 Event Dashboard | NUP | IMPORTANT | [x] |

**Scenario coverage:** SC02 (reworked) references S18 as the entry point for invite recipients.

---

### Scenarios

#### SC01 Alice Organizes Dinner

*Reworked in place — Wave 1*

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Add `Idempotency-Key` mention on expense creation | CR-001 | NICE | [ ] |
| 2 | Add line-item splits with different consumption groups (food vs drinks) | Wave 1 | IMPORTANT | [x] |
| 3 | Add weight=0 for non-drinker Carol on drinks line item | Wave 1 | IMPORTANT | [x] |
| 4 | Add verified arithmetic with per-person breakdown and zero-sum checksum | Wave 1 | IMPORTANT | [x] |

**Differentiator coverage:** Line-item splits with different groups (primary), weight=0 exclusion (primary), aggressive defaults (primary).

#### SC02 Bob Joins via Invite

*Reworked in place — Wave 1*

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Document free tier auto-creation on register | JF-4A | IMPORTANT | [x] |
| 2 | Add dispute lifecycle (raise → review → modify → resolve via S18 → S02 → S05 path) | Wave 1, JF-3 | IMPORTANT | [x] |
| 3 | Add recipient onboarding flow referencing S18 Invite Landing | Wave 1, NUP | IMPORTANT | [x] |
| 4 | Document auto-apply on modification request approval (no separate PATCH needed) | JF-3 | IMPORTANT | [x] |

**Differentiator coverage:** Structured disputes with audit trail (primary), recipient onboarding / first experience (primary).

#### SC03 Family Holiday Shared PFG

*Reworked in place — Wave 1*

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Update composite group creation orchestration | JF-2B | IMPORTANT | [x] |
| 2 | Add verified arithmetic with per-person and per-couple breakdown | Wave 1 | IMPORTANT | [x] |
| 3 | Add unequal splits composing with PFGs | Wave 1 | IMPORTANT | [x] |
| 4 | Simplify PFG discovery — show value before 7-step creation flow | Wave 1 | IMPORTANT | [x] |

**Differentiator coverage:** Settlement group (PFG) magic (primary).

#### SC04 Housemates Monthly Bills

*Reworked in place — Wave 1*

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Fix arithmetic errors in original scenario | Wave 1 | CRITICAL | [x] |
| 2 | Add custom rent split (bigger room = higher proportion) | Wave 1 | IMPORTANT | [x] |
| 3 | Add dietary exclusion weight=0 (vegan excluded from meat-based grocery line items) | Wave 1 | IMPORTANT | [x] |
| 4 | Add verified arithmetic with zero-sum checksum | Wave 1 | IMPORTANT | [x] |

**Differentiator coverage:** Weight flexibility / unequal splits (secondary), weight=0 exclusion (secondary).

#### SC05 Settle and Close

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Settlement suggestions now server-side | JF-2A | CRITICAL | [x] |
| 2 | Add settlement guard on closed events | JF-1B | IMPORTANT | [x] |

#### SC06 Free User Hits Limits

*Reworked in place — Wave 1*

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Document free tier auto-creation | JF-4A | IMPORTANT | [x] |
| 2 | Restructure to show value (completed split with penny-exact checksum) before the funding gate | Wave 1 | IMPORTANT | [x] |
| 3 | Address cost-spread visibility — user sees what they are paying for before being asked to fund | Wave 1 | IMPORTANT | [x] |

**Differentiator coverage:** Aggressive defaults / value demonstration (secondary).

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

#### SC13 Income-Proportional Couple (Mel & Jake)

*New scenario — Wave 1*

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Create scenario file `scenarios/SC13-income-proportional-couple.md` | Wave 1 | IMPORTANT | [x] |
| 2 | Demonstrate 3:2 weighted splits (60/40) applied consistently across multiple expenses | Wave 1 | IMPORTANT | [x] |
| 3 | Demonstrate mixed payers within the same event using weighted splits | Wave 1 | IMPORTANT | [x] |
| 4 | Include verified arithmetic: three expenses with known 3:2 ratios and zero-sum net settlement | Wave 1 | IMPORTANT | [x] |

**Differentiator coverage:** Weight flexibility / unequal splits (primary), penny-exact arithmetic (secondary).

#### SC17 Work Lunch Penny-Exact

*New scenario — Wave 1 (renumbered from proposed "SC05" to avoid collision)*

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Create scenario file `scenarios/SC17-work-lunch-penny-exact.md` | Wave 1 | IMPORTANT | [x] |
| 2 | Demonstrate penny-exact arithmetic: $47.30 ÷ 3 = $15.77 + $15.77 + $15.76 = $47.30 exactly | Wave 1 | IMPORTANT | [x] |
| 3 | Demonstrate largest-remainder allocation algorithm (no rounding drift) | Wave 1 | IMPORTANT | [x] |
| 4 | Include conservation invariant checksum | Wave 1 | IMPORTANT | [x] |

**Differentiator coverage:** Penny-exact arithmetic with conservation invariant (primary), aggressive defaults (primary).

#### SC18 Weekend Away Multi-Payer

*New scenario — Wave 1 (renumbered from proposed "SC06" to avoid collision)*

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Create scenario file `scenarios/SC18-weekend-away-multi-payer.md` | Wave 1 | IMPORTANT | [x] |
| 2 | Demonstrate multiple payers across different expenses in the same event | Wave 1 | IMPORTANT | [x] |
| 3 | Demonstrate net settlement minimization — 4 expenses collapse to 2 payments | Wave 1 | IMPORTANT | [x] |
| 4 | Demonstrate line-item splits with different consumption groups across payers | Wave 1 | IMPORTANT | [x] |
| 5 | Include verified arithmetic with deterministic settlement netting | Wave 1 | IMPORTANT | [x] |

**Differentiator coverage:** Line-item splits with different groups (primary), net settlement minimization (primary), multiple payers (primary).

#### SC19 Birthday Shout (Weight=0)

*New scenario — Wave 1 (renumbered from proposed "SC07" to avoid collision)*

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Create scenario file `scenarios/SC19-birthday-shout-weight-zero.md` | Wave 1 | IMPORTANT | [x] |
| 2 | Demonstrate weight=0 exclusion pattern: Dave's food is shouted (weight 0) but he pays for his own drinks (weight 1) | Wave 1 | IMPORTANT | [x] |
| 3 | Demonstrate 5-person event with mixed weight assignments per line item | Wave 1 | IMPORTANT | [x] |
| 4 | Include before/after share comparison showing weight=0 impact ($89 → $19) | Wave 1 | IMPORTANT | [x] |
| 5 | Include verified arithmetic with zero-sum checksum | Wave 1 | IMPORTANT | [x] |

**Differentiator coverage:** Weight=0 exclusion (primary), weight flexibility (primary).

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

#### R07 Dispute Rail

*New rail — Wave 2 coherence update*

| # | Change | Source | Priority | Status |
|---|--------|--------|----------|--------|
| 1 | Create R07 Dispute Rail (raise → review → resolve lifecycle) | Wave 2, JF-3 | IMPORTANT | [x] |
| 2 | Update SC02 rail reference from R05 to R07 | Wave 2 | IMPORTANT | [x] |

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
| SC05 Settle and Close | — |
| SC07 Charlie Claims Person | — |
| SC08 Member Permission Walls | — |
| SC09 Multi-Event Power User | — |
| SC10 Quota Exhaustion Recovery | — |
| SC11 Complex Group Splits | — |
| SC12 Dispute and Modification | Adequate; gaps closed in earlier wave |
| A04 SC01 API Changes | Historical |
| A05 Gap Closure Changelog | Historical |
| nav-map-interactive.md | — |

---

## New Documents Needed

| Document | Reason | Source | Priority | Status |
|----------|--------|--------|----------|--------|
| ~~**SC13: Person-Targeted Invite Auto-Claim**~~ | ~~Path D is undocumented — targeted invite skips merge~~ | NUP | IMPORTANT | [x] Created Wave 1 as `scenarios/SC13-income-proportional-couple.md` (scope shifted to weighted splits; Path D covered by SC02 rework) |
| **SC14: Sponsorship on Join** | Sponsor brings dependents on join — undocumented flow | NUP | IMPORTANT | [ ] |
| **SC17: Work Lunch Penny-Exact** | Penny-exact arithmetic undocumented | Wave 1 | IMPORTANT | [x] Created `scenarios/SC17-work-lunch-penny-exact.md` |
| **SC18: Weekend Away Multi-Payer** | Multiple payers / net settlement minimization undocumented | Wave 1 | IMPORTANT | [x] Created `scenarios/SC18-weekend-away-multi-payer.md` |
| **SC19: Birthday Shout (Weight=0)** | Weight=0 exclusion pattern undocumented as primary scenario | Wave 1 | IMPORTANT | [x] Created `scenarios/SC19-birthday-shout-weight-zero.md` |
| **S18 Invite Landing** | Recipient onboarding entry screen needed for SC02 rework | Wave 1, NUP | IMPORTANT | [x] Created `screens/S18-invite-landing.md` |
| **A07 Idempotency Guide** (or section in A01) | Idempotency touches every mutation — needs reference | CR-001 | IMPORTANT | [ ] |
| **A08 Multi-Currency Reference** (or section in A01) | FX rates, conversion, write-offs — complex feature | CR-002 | IMPORTANT | [ ] |

---

## Differentiator Capability Coverage (Wave 1 State)

Cross-reference to `scenario-strategy.md` §6. Primary scenarios demonstrate the capability as their central purpose; secondary scenarios exercise it incidentally.

| Capability | Primary Scenarios | Secondary | Gap? |
|------------|-------------------|-----------|:----:|
| Line-item splits with different consumption groups | SC01 (reworked), SC18 (new) | SC11 | — |
| Penny-exact arithmetic with conservation invariant | SC17 (new) | SC01 (reworked), SC13 (new) | — |
| Weight=0 exclusion | SC19 (new) | SC01 (reworked), SC04 (reworked) | — |
| Weighted / unequal splits | SC13 (new) | SC04 (reworked), SC11 | — |
| Settlement group (PFG) magic | SC03 (reworked) | — | — |
| Net settlement minimization | SC18 (new) | SC05 | — |
| Structured disputes with audit trail | SC02 (reworked) | SC12 | — |
| Recipient onboarding (first experience) | SC02 (reworked), S18 (new) | — | — |
| Multiple payers | SC18 (new) | — | — |
| Aggressive defaults (2-field expense entry) | SC01 (reworked), SC17 (new) | All | — |
| Multi-currency (FX rates, cross-currency) | — | — | **GAP** |
| Undo/correct aggressive defaults | — | — | **GAP** |
| Scaling (10+ participants) | — | — | **GAP** |
| Partial settlement & reopening | — | — | **GAP** |
| Offline/poor connectivity | — | — | **GAP** |

**Wave 1 result:** All eight demonstrable differentiators (capabilities without API blockers) now have primary scenario coverage. Five remaining gaps require backend design decisions before scenarios can be specified.

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

### Wave 1 — Critical Accuracy Fixes + Differentiator Scenario Coverage

**Status: Complete (2026-03-03)**

Fix documentation that is factually wrong, and add scenario coverage for all demonstrable differentiators.

**Critical accuracy targets (complete):** S07, S09, S10, S11, S12, S16, SC05, SC12, A01 (idempotency), A02, A03, 01-nav-map

**Scenario coverage targets (complete):** SC01, SC02, SC03, SC04, SC06 reworked in place; SC13, SC17, SC18, SC19 created new; S18 Invite Landing screen spec created.

**Sources:** CR-001, CR-002, JF-2A, JF-3, JF-1C, JF-4B, Wave 1 scenario strategy

### Wave 2 — Multi-Currency

Add multi-currency support across all affected screens and appendix.

**Targets:** S05, S09, S10, S11, S12 (cross-currency), A01 (FX orchestrations), new A08 or A01 section

**Sources:** CR-002

### Wave 3 — New Features

Document undocumented features and user paths.

**Targets:** S02, S03, S06, S13, S14, S17, R04, R06, new SC14 (Sponsorship on Join), new A07 or A01 section

**Note:** SC13, SC17, SC18, SC19 were advanced to Wave 1 as part of differentiator scenario coverage. S18 was also completed in Wave 1.

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
