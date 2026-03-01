# SC08 — Member Hits Permission Walls

**Persona:** Eve — a member (not admin) trying to do things beyond her role.
**Screens:** S05 → S09 → S07 → S10 → S16
**Rails:** R06 (Admin)

## Design Principle

**"Roles protect, not prevent."** Eve can see everything a member should see and can request changes she can't make directly. Permission walls show what action is needed and who can do it — they never leave the user stuck.

## Narrative

Eve joins Alice's event and gets approved as a regular member:

```
S02 Register → creates account (display name: "Eve")
    ⚡ POST /auth/register → tokens stored
    ⚡ POST /events/join {invite_code: "..."}
       → event_role: pending_approval
    → Alice approves
    ⚡ POST /events/{eid}/event-roles/{rid}/approve
       → role: active, permission_level: member
```

Eve can add her own expenses (member permission):

```
Eve's S05 → taps "+ Add Expense"
S09 Add Expense → "Eve's taxi $22"
    → paid: Eve, split: Everyone
    ⚡ POST /events/{eid}/transactions → 201 Created ✓
```

Eve tries to edit someone else's expense:

```
Eve's S05 → taps on Alice's "Restaurant $120" expense
S10 Expense Detail → sees full breakdown (read access ✓)
    → taps [Edit]
    ⚡ PATCH /events/{eid}/transactions/{tid} → 403 Forbidden
    → Error: "Only the event admin or expense creator can edit this transaction"
    → [Request Change →] button shown
```

Eve tries to remove a person:

```
Eve's S05 → taps "People ▸"
S07 Manage People → sees everyone (read access ✓)
    → taps Bob → [Remove] is greyed out / hidden
    → tooltip: "Only admins can remove people"
    ⚡ DELETE /events/{eid}/persons/{bob_id} → 403 Forbidden
    → Error: "Admin permission required to remove persons"
```

Eve tries to close the event:

```
Eve's S05 → ⚙️ Event Settings
    → [Close Event] is greyed out / hidden
    → "Only the event admin can close this event"
```

Eve tries to approve a pending join request:

```
Eve's S05 → sees "⚠️ 1 pending action" (Frank wants to join)
    → taps notification
    → [Approve] / [Reject] buttons not shown for members
    → "Admin approval required"
```

Eve uses the modification request flow instead:

```
Eve's S10 (Alice's expense) → taps [Request Change →]
S16 Admin Moderation Queue (Eve's view: submission form)
    → Type: "Modify transaction"
    → Message: "The restaurant bill should be $110 not $120 — receipt shows $110"
    ⚡ POST /events/{eid}/modification-requests
       {target_type: "transaction", target_id: tid, message: "..."}
    → "Request sent to admin ✓"
    → status: pending
```

Alice sees the request in her moderation queue:

```
Alice's S16 Admin Moderation Queue
    → "Eve requested a change to Restaurant $120"
    → Message: "The restaurant bill should be $110 not $120"
    → [Approve & Edit] [Reject]
    → taps [Approve & Edit]
    ⚡ PATCH /events/{eid}/transactions/{tid} {amount: 110.00}
    ⚡ POST /events/{eid}/modification-requests/{rid}/resolve {status: "approved"}
    → Eve notified: "Your change request was approved ✓"
```

## Validates

- 403 boundaries on update, delete, and approve actions
- Clear, role-aware error messages (not generic "forbidden")
- [Request Change] offered as alternative to direct edit
- Modification request creation and submission
- Admin moderation queue (S16) with approve/reject actions
- Member can create own expenses but not edit others'
- Member has full read access to event data
- Grey-out / hide pattern for unavailable actions in UI

## Key Principle Demonstrated

> **"Roles protect, not prevent"**
>
> Eve is never stuck. Every permission wall offers a path forward:
> can't edit → request change. Can't approve → wait for admin.
> The 403 response tells her exactly what permission is needed
> and suggests the modification request alternative.
