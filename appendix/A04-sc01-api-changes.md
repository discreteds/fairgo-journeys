# A04 — SC01 API Changes Specification

Backend changes required to support the SC01 "Capture First, Share Last" reflow.

## Overview

Three backend changes are required. They are scoped to avoid breaking existing flows (SC02–SC06) and existing API consumers.

| # | Change | Type | Breaking? |
|---|--------|------|-----------|
| 1 | Transaction split-pending state | Schema + validation + query | No — additive field, existing requests unaffected |
| 2 | Auto-assign splits on person creation | New behavior on existing endpoint | No — only triggers when split-pending transactions exist |
| 3 | Position calculation exclusion | Logic change | No — pending transactions have no splits to sum anyway |
| 4 | Person `phone_hint` | Already exists | N/A |
| 5 | Bulk invite codes | Deferred | N/A |

---

## Change 1: Transaction Split-Pending State

### Current State

**Model** (`fairgo-backend/src/fairgo_backend/models/transaction.py`):
```python
class Transaction(TimestampMixin, Base):
    status: str  # "pending", "approved", "disputed", "resolved" — approval workflow only
    # No field for split assignment state
```

**Schema** (`fairgo-backend/src/fairgo_backend/schemas/transaction.py`):
```python
class LineItemCreate(BaseModel):
    splits: list[LineItemSplitCreate]  # REQUIRED — must have ≥1 expense + ≥1 consumption
```

**Validation** (`fairgo-backend/src/fairgo_backend/services/transaction_service.py`):
- Each line item MUST have at least one `expense` split AND one `consumption` split
- All `person_id` references must be active persons in the event
- Effective weights must be non-zero

### Required Changes

**A. New column on `Transaction` model:**

```python
# transaction.py
class Transaction(TimestampMixin, Base):
    ...
    splits_status: str = Column(String(20), default="assigned", nullable=False)
    # Values: "pending" | "assigned"
    # "pending"  = saved without splits, awaiting people
    # "assigned" = splits exist and are calculated (default, backwards-compatible)
```

**Migration:** Add column with `default="assigned"` — all existing transactions get `"assigned"`, no data backfill needed.

**B. Schema changes for transaction creation:**

```python
# TransactionCreate — updated
class TransactionCreate(BaseModel):
    description: str
    line_items: list[LineItemCreate]  # min_length=1 still applies
    splits_status: str = "assigned"   # NEW — optional, defaults to existing behavior
    date: datetime | None = None
    currency: str = "AUD"

# LineItemCreate — conditional validation
class LineItemCreate(BaseModel):
    description: str | None = None
    amount: Decimal  # gt=0
    splits: list[LineItemSplitCreate] = []  # Change: default to empty list
    # Validation: if parent splits_status == "assigned", splits must have ≥1 expense + ≥1 consumption
    # Validation: if parent splits_status == "pending", splits must be empty
```

**C. New validation rules in `transaction_service.py`:**

```python
# When splits_status == "pending":
#   - line_items[*].splits MUST be empty
#   - Skip person_id validation (no persons referenced)
#   - Skip expense/consumption side validation
#   - Transaction is saved with total_amount computed from line item amounts
#   - No position impact (see Change 3)

# When splits_status == "assigned" (default, existing behavior):
#   - Existing validation unchanged
#   - line_items[*].splits required with expense + consumption sides
```

**D. Response schema update:**

```python
# TransactionOut — add splits_status
class TransactionOut(BaseModel):
    ...
    splits_status: str  # NEW — "pending" | "assigned"
```

**E. Query filter support:**

```
GET /events/{eid}/transactions?splits_status=pending
GET /events/{eid}/transactions?splits_status=assigned
```

Add optional query parameter to the transactions list endpoint for filtering.

### Example: Creating a Split-Pending Transaction

**Request:**
```json
POST /events/{eid}/transactions
{
    "description": "Pizza",
    "splits_status": "pending",
    "line_items": [
        {
            "description": null,
            "amount": 45.00,
            "splits": []
        }
    ]
}
```

**Response:**
```json
{
    "id": "tx_abc123",
    "event_id": "evt_xyz",
    "description": "Pizza",
    "status": "pending",
    "splits_status": "pending",
    "total_amount": "45.00",
    "total_amount_base": "45.00",
    "line_items": [
        {
            "id": "li_def456",
            "description": null,
            "amount": "45.00",
            "amount_base": "45.00",
            "splits": []
        }
    ],
    "created_at": "2026-02-27T10:00:00Z"
}
```

---

## Change 2: Auto-Assign Splits on Person Creation

### Current State

**Endpoint:** `POST /events/{eid}/persons`

**Current behavior** (`fairgo-backend/src/fairgo_backend/services/person_service.py`):
1. Create person record
2. Create singleton group (name = display_name)
3. Create PFG (singleton group → person)
4. Return `PersonOut`

No interaction with transactions.

### Required Changes

**A. Extended behavior when split-pending transactions exist:**

After steps 1–3 (existing), add:

```python
# 4. Query transactions with splits_status="pending" in this event
# 5. For each pending transaction:
#    a. Get all active persons in the event (including the just-created person)
#    b. For each line item:
#       - Set the event creator as the sole expense-side split (weight=1, modifier=1)
#       - Set ALL active persons as consumption-side splits (weight=1, modifier=1 each)
#       - Recalculate shares using largest remainder method
#    c. Update splits_status to "assigned"
# 6. Return person + affected transactions
```

**B. Extended response schema:**

```python
# PersonCreateResponse — new response type for this endpoint
class PersonCreateResponse(BaseModel):
    person: PersonOut
    affected_transactions: list[TransactionOut]  # Transactions that had splits auto-assigned
    # Empty list when no split-pending transactions existed
```

**Note:** This is a response shape change. This is a pre-launch API, so breaking changes are acceptable. Existing clients will need to unwrap `person` from the response.

**C. Who-paid default for auto-assigned splits:**

When auto-assigning splits, the **expense side** defaults to the transaction creator (`created_by` user's person in the event). This matches the S09 smart default "Who paid: You (Alice)."

If the transaction creator's person can't be determined (edge case), fall back to the event admin's person.

### Example: Adding Bob (auto-assigns to Pizza)

**Request:**
```json
POST /events/{eid}/persons
{
    "display_name": "Bob"
}
```

**Response:**
```json
{
    "person": {
        "id": "per_bob",
        "display_name": "Bob",
        "status": "active",
        "resolution_status": "auto_created",
        "email_hint": null,
        "phone_hint": null,
        "pfg": {
            "group_id": "grp_bob",
            "group_name": "Bob",
            "is_singleton": true
        }
    },
    "affected_transactions": [
        {
            "id": "tx_abc123",
            "description": "Pizza",
            "splits_status": "assigned",
            "total_amount": "45.00",
            "line_items": [
                {
                    "id": "li_def456",
                    "amount": "45.00",
                    "splits": [
                        {"person_id": "per_alice", "side": "expense", "weight": 1, "modifier": "1.0", "effective_weight": "1.0", "share": "45.00"},
                        {"person_id": "per_alice", "side": "consumption", "weight": 1, "modifier": "1.0", "effective_weight": "1.0", "share": "22.50"},
                        {"person_id": "per_bob", "side": "consumption", "weight": 1, "modifier": "1.0", "effective_weight": "1.0", "share": "22.50"}
                    ]
                }
            ]
        }
    ]
}
```

**After adding Carol (3rd person):**

The same Pizza transaction's consumption splits become:
- Alice: $15.00, Bob: $15.00, Carol: $15.00

**After adding Dave (4th person):**
- Alice: $11.25, Bob: $11.25, Carol: $11.25, Dave: $11.25

---

## Change 3: Position Calculation — Exclude Split-Pending

### Current State

**Service** (`fairgo-backend/src/fairgo_backend/services/position_service.py`):
- Positions calculated from ALL transactions — no status filtering
- Line 37: queries all transactions for the event

### Required Change

Add filter to exclude split-pending transactions:

```python
# position_service.py — transaction query
transactions = db.query(Transaction).filter(
    Transaction.event_id == event_id,
    Transaction.splits_status != "pending"  # NEW — exclude pending
).all()
```

**Why this is safe:** Split-pending transactions have no splits, so they contribute $0 to positions anyway. But explicitly excluding them:
1. Avoids edge cases in the calculation loop
2. Makes the intent clear
3. Prevents any future bugs if the calculation changes

---

## Change 4: Person `phone_hint` — No Change Required

The person model already has:
```python
phone_hint: str | None  # max 50 chars
```

Both `PersonCreate` and `PersonOut` schemas already include this field. The S06 "Prepare & Share" screen can use the existing `PUT /events/{eid}/persons/{pid}` endpoint to update `phone_hint`.

**No backend changes needed.**

---

## Change 5: Bulk Invite Codes — Deferred

For V1, the "Create All Personal Links" feature on S06 will fire N parallel requests:

```
POST /events/{eid}/invite-codes {target_person_id: per_bob, max_uses: 1}
POST /events/{eid}/invite-codes {target_person_id: per_carol, max_uses: 1}
POST /events/{eid}/invite-codes {target_person_id: per_dave, max_uses: 1}
```

This is acceptable for events with <50 participants (the funded event limit). A bulk endpoint can be added later if needed.

---

## Files to Change in fairgo-backend

| File | Change |
|------|--------|
| `models/transaction.py` | Add `splits_status` column |
| `schemas/transaction.py` | Add `splits_status` to `TransactionCreate` and `TransactionOut`, make `LineItemCreate.splits` default to `[]` |
| `services/transaction_service.py` | Conditional validation based on `splits_status`, skip split validation when pending |
| `api/v1/transactions.py` | Add `splits_status` query parameter to list endpoint |
| `services/person_service.py` | Auto-assign splits to pending transactions on person creation |
| `schemas/person.py` | New `PersonCreateResponse` wrapping `PersonOut` + `affected_transactions` |
| `api/v1/persons.py` | Return `PersonCreateResponse` from create endpoint |
| `services/position_service.py` | Filter out `splits_status="pending"` transactions |
| `alembic/versions/` | Migration to add `splits_status` column with default `"assigned"` |

## Edge Cases

| Case | Behavior |
|------|----------|
| Split-pending transaction exists, then person is removed | If only 1 person remains, transaction stays/reverts to `splits_status: "pending"` |
| Split-pending transaction manually edited | Allow `PUT /transactions/{tid}` to add splits and change `splits_status` to `"assigned"` |
| Event has existing people when a split-pending transaction is created | Auto-assign immediately using existing people (splits_status transitions to "assigned" on save) |
| Multiple split-pending transactions when person added | All pending transactions get the new person added in a single database transaction |
| Split-pending transaction and event closure | Prevent `POST /events/{eid}/close` if any transactions have `splits_status: "pending"` — return 409 with message |
