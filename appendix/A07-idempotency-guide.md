# A07 — Idempotency Guide

**Source:** CR-001 (Idempotency & Robustness)
**Purpose:** Reference for frontend developers implementing safe retry logic for all financial mutations.

---

## Principle

> Every financial mutation must be safe to retry. Network failures, double-taps, and offline sync should never create duplicate records.

---

## Resource-Creation vs State-Transition Endpoints (CR-015)

Endpoints fall into two categories with different idempotency requirements:

### Resource-Creation Endpoints — Require `Idempotency-Key`

These endpoints create new records. Without idempotency protection, retries would create duplicates.

| Endpoint | Method | Rationale |
|----------|--------|-----------|
| `/events/{eid}/transactions` | POST | Duplicate expenses corrupt balances |
| `/events/{eid}/transactions/{tid}` | PUT | Re-applied edits could overwrite concurrent changes |
| `/events/{eid}/settlements` | POST | Duplicate settlements create false debts |
| `/events/{eid}/write-offs` | POST | Duplicate write-offs distort remaining balances |
| `/events/{eid}/funding` | POST | Duplicate funding charges user twice |
| `/events/{eid}/funding/cost-spread` | POST | Re-spreading costs doubles line items |
| `/auth/register` | POST | Prevents duplicate account creation on retry |

### State-Transition Endpoints — Naturally Idempotent (No Key Needed)

These endpoints move an entity from one status to another. Repeating the same transition is a no-op — the entity is already in the target state. No `Idempotency-Key` header is required.

| Endpoint | Method | Transition |
|----------|--------|-----------|
| `/events/{eid}/transactions/{tid}/approve` | POST | proposed → approved |
| `/events/{eid}/transactions/{tid}/cancel` | POST | * → cancelled |
| `/events/{eid}/settlements/{sid}/confirm` | POST | proposed → confirmed |
| `/events/{eid}/settlements/{sid}/pay` | POST | confirmed → paid |
| `/events/{eid}/settlements/{sid}/void` | POST | proposed/confirmed → voided |
| `/events/{eid}/event-roles/{rid}/approve` | POST | pending → active |
| `/events/{eid}/event-roles/{rid}/reject` | POST | pending → rejected |
| `/events/{eid}/invite-codes/{cid}/deactivate` | POST | active → deactivated |

State-transition endpoints return the current state of the entity. If the transition has already been applied, the server returns the entity in its current (post-transition) state — the client sees the same result either way.

---

## Key Generation

The client generates the idempotency key **before** sending the request:

```
const idempotencyKey = crypto.randomUUID();  // UUID v4

fetch('/events/{eid}/transactions', {
  method: 'POST',
  headers: {
    'Idempotency-Key': idempotencyKey,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(payload)
});
```

**Rules:**
- Generate a new UUID v4 for each distinct user action
- Store the key alongside the request in the operation queue (for offline sync)
- Never reuse a key across different operations
- Reuse the **same** key when retrying the **same** operation

---

## Request / Response Flow

```
┌──────────┐                              ┌──────────┐
│  Client   │                              │  Server   │
└─────┬─────┘                              └─────┬─────┘
      │                                          │
      │  POST /transactions                      │
      │  Idempotency-Key: abc-123                │
      │ ────────────────────────────────────────► │
      │                                          │
      │         ┌─────────────────────┐          │
      │         │ Check key store:    │          │
      │         │ abc-123 not found   │          │
      │         │ → execute operation │          │
      │         │ → store response    │          │
      │         └─────────────────────┘          │
      │                                          │
      │  201 Created                             │
      │  {transaction_id: "tx_42", ...}          │
      │ ◄──────────────────────────────────────── │
      │                                          │
      │  ─── network drops, client retries ───   │
      │                                          │
      │  POST /transactions                      │
      │  Idempotency-Key: abc-123  (same key)    │
      │ ────────────────────────────────────────► │
      │                                          │
      │         ┌─────────────────────┐          │
      │         │ Check key store:    │          │
      │         │ abc-123 FOUND       │          │
      │         │ → return cached     │          │
      │         │   response          │          │
      │         └─────────────────────┘          │
      │                                          │
      │  201 Created                             │
      │  Idempotency-Replayed: true              │
      │  {transaction_id: "tx_42", ...}          │
      │ ◄──────────────────────────────────────── │
```

The client receives the **same response** both times. The operation executed exactly once.

---

## `Idempotency-Replayed` Header

When the server detects a replayed idempotency key, it returns the cached response with an additional header:

```
Idempotency-Replayed: true
```

**Frontend behaviour on replay:**
- Treat the response as a success (the operation already completed)
- Do **not** trigger success side effects again (toasts, analytics events, navigation)
- Update local state with the response data (it's the canonical result)

```javascript
const response = await fetch(url, { method: 'POST', headers, body });
const isReplay = response.headers.get('Idempotency-Replayed') === 'true';

if (response.ok) {
  updateLocalState(await response.json());
  if (!isReplay) {
    showSuccessToast('Expense saved');
    trackAnalytics('expense_created');
  }
}
```

---

## `recoverable` Error Field

When a mutation fails, the error response includes a `recoverable` boolean:

```json
{
  "error": "service_unavailable",
  "message": "Database temporarily unavailable",
  "recoverable": true,
  "suggestion": "Please try again in a few seconds"
}
```

| `recoverable` | Meaning | Client Action |
|:--------------:|---------|---------------|
| `true` | Transient failure (timeout, 503, rate limit) | Retry with **same** idempotency key |
| `false` | Permanent failure (validation, 4xx) | Show error to user; do **not** retry automatically |

---

## Retry Strategy

When `recoverable: true`, the client retries with exponential backoff and jitter:

```
Attempt 1: wait 1s ± random(0-500ms)
Attempt 2: wait 2s ± random(0-500ms)
Attempt 3: wait 4s ± random(0-500ms)
```

**Rules:**
- Maximum 3 automatic retries
- Use the **same** `Idempotency-Key` for all retries of the same operation
- After 3 failures, stop retrying and surface the error to the user
- If the error has `recoverable: false`, do **not** retry — show the error immediately

```
┌────────────────────────────────────────────────────┐
│  After 3 failed retries:                           │
│                                                    │
│  ⚠️ Couldn't save expense                          │
│  "Database temporarily unavailable"                │
│                                                    │
│  [Retry]  [Discard]                                │
└────────────────────────────────────────────────────┘
```

---

## Offline Sync Integration

When the app is offline, mutations are queued locally with pre-generated idempotency keys:

```
Queue entry:
{
  id: "queue_1",
  endpoint: "POST /events/{eid}/transactions",
  payload: { ... },
  idempotency_key: "pre-generated-uuid-v4",
  status: "queued",        // queued → syncing → synced | failed
  created_at: "2026-03-03T10:15:00Z"
}
```

When connectivity returns:
1. Process queue entries **sequentially** (order matters for position calculations)
2. Each request uses its pre-generated `Idempotency-Key`
3. If server returns `Idempotency-Replayed: true` → mark as synced (already applied, e.g. from a previous partial sync)
4. If server returns an error with `recoverable: true` → apply retry strategy
5. If server returns `recoverable: false` → mark as failed, prompt user

See SC22 for the full offline/sync narrative.

---

## Cross-References

- **A01 Orchestrations** — endpoint table with idempotency requirements
- **S09 Add Expense** — `Idempotency-Key` on expense save
- **S12 Settle Up** — `Idempotency-Key` on settlement mutations
- **SC22 Offline/Poor Connectivity** — offline queue with pre-generated keys
