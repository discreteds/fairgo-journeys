# A05 — Gap Closure Changelog

**Date:** 2026-02-28
**Backend branch:** main (commit range: post-d6a4e5f7b8c9)
**Gaps closed:** 10 of 12 identified (2 post-MVP deferred)
**Test suite:** 160 tests, 0 failures

## Summary

| Gap | Name | Screens Affected | Key API Change |
|-----|------|-----------------|----------------|
| I2 | Self-Merge Routing Verification | S05, S07 | Verified: non-admin merge endpoint reachable |
| P2 | Enriched Transactions | S10, A01 | `GET /transactions/{tid}` returns full tree (always-on) |
| P1 | Enriched Events List | S03, A01 | `GET /events?include=my_position` embeds position |
| I1 | User-Facing Match Suggestions | S05, S07 | New: `GET /events/{eid}/persons/my-matches` |
| F1 | Status-Based Soft-Delete | S08, S10, S12, R03, SC05, A01 | New: `POST .../cancel`, `POST .../void`, `POST .../archive` |
| F3 | Modifier Range Extension | S09 | Schema: modifier `le=1` -> `le=2` |
| F2 | Multi-Currency Penny Allocation | — | New: `currency_utils.py`, precision parameter on `distribute_with_remainder()` |
| I3 | Singleton Guard Auto-Group | S08, SC03 | `POST .../members` on singleton auto-creates new group |
| E1 | Event Type & Visibility | S03, S04, SC04 | New fields: `event_type`, `visibility` on Event |
| P3 | Activity Feed | S17, A01 | New: `GET /users/me/activity` endpoint + activity table |

---

## CR-015 through CR-020: Remaining Gaps Resolution (2026-03-06)

**Backend branch:** main (post-002-remaining-gaps-resolution)
**Gaps closed:** 23 functional requirements across 6 Change Records

| CR | Name | Key Changes |
|----|------|-------------|
| CR-015 | Settlement Refinements | `effectively_settled` boolean on positions (1¢/participant tolerance); `period_label` on settlements; over-settlement guard (101% tolerance, `OVER_SETTLEMENT` 422); settlement suggestions for both singular and ongoing events; idempotency scope refinement (state-transitions no longer need `Idempotency-Key`) |
| CR-016 | API Friction Reduction | Equal-split shortcut (`split_mode: "equal"`, `payer_id`, `split_among`); `display_name` optional on join (defaults to registration name); `UNFUNDED_LIMIT` error code with structured detail (`current_count`, `limit`, `requires_funding`) |
| CR-017 | Response Shape Normalisation (BREAKING) | Paginated `{items, total, limit, offset}` wrapper for list endpoints; field aliases (`split_type`/`side`, `primary_financial_group`/`pfg`, `display_name`/`slot_label`) |
| CR-018 | Template Data Completeness | PFG preset capture on save-as-template (`pfg_presets` field); template provenance on events (`source_template_id`, `source_template_version`); `settlement_count` on events |
| CR-019 | Operational Enhancements | `max_uses` on invite codes; invite deactivation endpoint (`POST .../deactivate`); batch person creation (`POST /events/{id}/persons/batch`); configurable `ACCESS_TOKEN_EXPIRE_MINUTES` |
| CR-020 | Error & HTTP Semantics | `DUPLICATE_MEMBER` 409 (was 400); `PENDING_MEMBER` 403 with contextual message; `INVITE_NOT_FOR_YOU` 403 for user-targeted invites; `target_user_id` on invite codes |

### New Error Codes

| Code | HTTP | When |
|------|------|------|
| `OVER_SETTLEMENT` | 422 | Settlement amount > 101% of outstanding balance |
| `UNFUNDED_LIMIT` | 422 | Resource limit hit on unfunded event |
| `PENDING_MEMBER` | 403 | Member with pending approval attempts restricted action |
| `DUPLICATE_MEMBER` | 409 | Adding person already in group |
| `INVITE_NOT_FOR_YOU` | 403 | Wrong user attempts to use user-targeted invite code |

---

## Deferred (Post-MVP)

| Gap | Name | Reason |
|-----|------|--------|
| I4 | Group Templates | High complexity; depends on stable event types (E1) |
| E2 | Event Linking & Settlement Cascades | Very high complexity; cross-event settlement requires careful design |
| Gap 5 | Bulk Invite Codes | Low priority convenience optimisation |

## Design Deviations

Decisions made during implementation that differ from the original gap cluster proposals:

### F1: Status-Based Lifecycle (not `deleted_at`)

**Original proposal:** Add `deleted_at: datetime | None` column to Transaction, Settlement, Group. Filter `WHERE deleted_at IS NULL`.

**Implemented:** Status-based transitions only. Transactions get `cancelled` status, settlements get `voided` status, groups get `archived` status (new column). No `deleted_at` timestamps. Action endpoints (`POST .../cancel`, `POST .../void`, `POST .../archive`) set the terminal status. Hard delete restricted to entities already in terminal status.

**Rationale:** Aligns with existing state machine patterns (entity-lifecycles.md). Status transitions are more expressive than a boolean deleted flag and integrate naturally with the existing status flow.

### P2: Always-On Enrichment (not `?include=` gated)

**Original proposal:** `GET /transactions/{tid}?include=line_items,splits` returns full tree when requested.

**Implemented:** The transaction endpoint always returns the full tree (line items with computed splits). No `?include=` parameter. The `TransactionOut` schema already had `line_items: list[LineItemOut]` with nested `splits` — the service layer just populates them on every call.

**Rationale:** The overhead of computing splits is negligible per-transaction. The `?include=` gate would add complexity without measurable benefit. Every consumer of the transaction detail needs the full tree.

### I3: New Group (not Singleton Conversion)

**Original proposal (Option B):** Auto-convert singleton to non-singleton on member add.

**Implemented:** Auto-create a *new* non-singleton group with both members. The original singleton remains untouched as the person's PFG. The new group is returned in the API response.

**Rationale:** Converting the singleton would break the invariant that every person has a singleton group. Creating a new group preserves all existing PFG assignments and lets the admin explicitly reassign PFGs to the new shared group.

### E1: Mutable Event Type (no Transaction Guard)

**Original proposal:** `event_type` only changeable before the first transaction is added. 409 if transactions exist.

**Implemented:** `event_type` mutable at any time via `PATCH /events/{eid}`.

**Rationale:** Simplicity. The guard can be added later if misuse is observed. Event type currently has no downstream behavioural effects in the backend — it's informational metadata for frontend display.

## Cross-References

- Gap cluster docs: `fairgo-central/03.user-journey-mods/02.Gaps/01-04.*.md`
- Design doc: `fairgo-backend/docs/plans/2026-02-28-journey-updates-post-gap-closure-design.md`
- Implementation plan: `fairgo-backend/docs/plans/2026-02-28-journey-updates-post-gap-closure.md`
- Backend implementation: `fairgo-backend/` main branch, 160 tests passing
