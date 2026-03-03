# SC22 — Offline / Poor Connectivity

> **Differentiator:** Expense splitting happens at campsites, restaurants, and remote lodges — not just places with reliable Wi-Fi. Fair Go queues mutations offline and syncs safely when connectivity returns, using idempotency keys to prevent duplicates.

**Persona:** Sam (admin) adding expenses at a remote campsite with no signal.
**Screens:** S05 → S09 → S11
**Rails:** R02 (Expense)

## Design Principle

**"Add now, sync later."**

Sam shouldn't have to remember expenses and enter them later. The app accepts input offline, calculates positions locally, and syncs transparently when signal returns. Idempotency keys (see A07) guarantee that retries and partial syncs never create duplicates.

## Participants

| Person | Role | Settlement Group |
|--------|------|-----------------|
| Sam | Admin, payer | Sam 🔵 |
| Jo | Member, payer | Jo 🟢 |

Two people, solo PFGs. Simple arithmetic to clearly demonstrate the offline pattern.

## Narrative

### 1. Event setup (online)

Sam created the event and added Jo earlier, while still in town.

```
Event: "Camping Trip 2026"
People: Sam (admin), Jo (member)
Status: 2 people, 0 expenses
```

### 2. Signal lost

Sam and Jo arrive at the campsite. Sam opens the app.

```
S05 Event Dashboard
    ┌──────────────────────────────────────────────────┐
    │  📡 Offline — changes will sync when connected   │
    │                                                  │
    │  Camping Trip 2026                               │
    │  2 people · 0 expenses                           │
    │                                                  │
    │  [+ Add Expense]                                 │
    └──────────────────────────────────────────────────┘

    Detection: fetch fails or navigator.onLine === false
    → UI shows offline banner
    → All mutations queue to local storage (IndexedDB)
```

### 3. Three expenses added offline

**Expense 1: Firewood**

```
S09 Add Expense (offline)
    Description: "Firewood"
    Amount: $30.00
    Paid by: Sam ← default
    Split: Everyone equally (Sam, Jo — $15.00 each)

    → taps "Save"
    → saved to local queue (not sent to server)
    → Idempotency-Key pre-generated: uuid-firewood

    Queue: [{
      id: "q1",
      endpoint: "POST /events/{eid}/transactions",
      payload: {description: "Firewood", line_items: [{amount: "30.00", ...}]},
      idempotency_key: "uuid-firewood",
      status: "queued",
      created_at: "2026-03-03T14:30:00Z"
    }]

S05 → badge: "1 pending"
```

**Expense 2: Camp food**

```
S09 Add Expense (offline)
    Description: "Camp food"
    Amount: $85.00
    Paid by: Sam
    Split: Everyone equally ($42.50 each)

    → saved to local queue
    → Idempotency-Key pre-generated: uuid-campfood

S05 → badge: "2 pending"
```

**Expense 3: Ice**

```
S09 Add Expense (offline)
    Description: "Ice"
    Amount: $12.00
    Paid by: Jo ← Sam changes payer to Jo
    Split: Everyone equally ($6.00 each)

    → saved to local queue
    → Idempotency-Key pre-generated: uuid-ice

S05 → badge: "3 pending"
```

### 4. Local position calculation

The app calculates positions locally from the queued expenses:

```
S05 Event Dashboard (offline, local calculation)

    3 expenses (local) · $127.00 total

S11 Balances (local)

    Person    Paid      Consumed    Net
    ──────    ────      ────────    ────
    Sam       $115.00   $63.50      +$51.50
    Jo        $12.00    $63.50      −$51.50
    ──────    ────      ────────    ────
    Total     $127.00   $127.00     $0.00 ✓

    Per-person consumed: ($30 + $85 + $12) / 2 = $63.50 each
    Sam paid: $30 + $85 = $115. Jo paid: $12.
```

### 5. Signal returns — sync

```
S05 Event Dashboard
    ┌──────────────────────────────────────────────────┐
    │  🔄 Syncing 3 expenses...                        │
    │  ████████░░░░░░░░ 1/3                            │
    └──────────────────────────────────────────────────┘

Sync process (sequential — order matters for position calculation):

    Request 1: POST /events/{eid}/transactions
       Headers: Idempotency-Key: uuid-firewood
       → 201 Created ✓
       → queue q1 status: "synced"

    Request 2: POST /events/{eid}/transactions
       Headers: Idempotency-Key: uuid-campfood
       → 201 Created ✓
       → queue q2 status: "synced"

    Request 3: POST /events/{eid}/transactions
       Headers: Idempotency-Key: uuid-ice
       → 201 Created ✓
       → queue q3 status: "synced"

    ┌──────────────────────────────────────────────────┐
    │  ✅ All changes synced                           │
    │                                    [Dismiss]     │
    └──────────────────────────────────────────────────┘
```

### 6. Server positions match local

```
S11 Balances (server-computed, after sync)

    Person    Paid      Consumed    Net
    ──────    ────      ────────    ────
    Sam       $115.00   $63.50      +$51.50
    Jo        $12.00    $63.50      −$51.50
    ──────    ────      ────────    ────
    Total     $127.00   $127.00     $0.00 ✓

    ← matches local calculation exactly ✓
```

### 7. Replay protection — partial sync recovery

If signal drops mid-sync (e.g. after expense 1 synced but before expense 2):

```
Signal drops after request 1 succeeds
→ queue state: q1: synced, q2: syncing (failed), q3: queued

Signal returns again → resume sync from q2

    Request 2 (retry): POST /events/{eid}/transactions
       Headers: Idempotency-Key: uuid-campfood (same key as before)
       → 201 Created
       → Idempotency-Replayed: false (first successful receipt by server)
       → queue q2 status: "synced"

    Request 3: POST /events/{eid}/transactions
       Headers: Idempotency-Key: uuid-ice
       → 201 Created ✓
       → queue q3 status: "synced"
```

If the server had actually received request 2 the first time (but the client didn't get the response):

```
    Request 2 (retry): POST /events/{eid}/transactions
       Headers: Idempotency-Key: uuid-campfood (same key)
       → 201 Created
       → Idempotency-Replayed: true ← server already processed this
       → no duplicate created
       → queue q2 status: "synced"
```

### 8. Conflict handling — concurrent edits

If Jo also added an expense while Sam was offline (from a different device with signal):

```
Expenses are additive (append-only):
→ On sync, Sam's local positions update to include Jo's expense
→ No conflict — Jo's expense simply appears alongside Sam's 3

S05 → now shows 4 expenses instead of 3
S11 → positions recalculate to include all 4
```

If Jo edited an expense that Sam also edited offline:

```
Server returns: 409 Conflict

    ┌──────────────────────────────────────────────────┐
    │  ⚠️ This expense was modified while you were      │
    │  offline.                                        │
    │                                                  │
    │  [View changes]  [Keep mine]  [Keep theirs]      │
    └──────────────────────────────────────────────────┘
```

### 9. Retry strategy on persistent failure

```
On network error during sync:
    Attempt 1: wait 1s ± jitter → retry with same Idempotency-Key
    Attempt 2: wait 2s ± jitter → retry
    Attempt 3: wait 4s ± jitter → retry

After 3 failures:
    ┌──────────────────────────────────────────────────┐
    │  ⚠️ Sync failed for "Camp food"                   │
    │  "Network unavailable"                           │
    │                                                  │
    │  [Retry]  [Discard]                              │
    └──────────────────────────────────────────────────┘

    [Retry] → resets attempt counter, tries again
    [Discard] → removes from queue (expense never created on server)
```

## Validates

- Offline detection and user-visible banner
- Local mutation queue with pre-generated idempotency keys
- Local position calculation matching server-computed positions
- Sequential sync preserving operation order
- `Idempotency-Replayed` header preventing duplicates on partial sync recovery
- Exponential backoff with jitter retry strategy (A07)
- Conflict handling for concurrent edits (409 Conflict)
- Graceful degradation — app remains usable without connectivity

## Key Principle Demonstrated

> **"Add now, sync later."** At a campsite with no signal, Sam adds three expenses. The app queues them locally, calculates positions instantly, and syncs transparently when signal returns. Pre-generated idempotency keys guarantee that even a partial sync followed by a retry never creates duplicates. The server positions match the local calculation exactly.
