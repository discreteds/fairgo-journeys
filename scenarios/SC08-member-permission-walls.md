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

Eve tries to add a person directly:

```
Eve's S05 → taps "People ▸"
→ Screen: S07
S07 Manage People → sees everyone (read access ✓)
    → [+ Add Person] is NOT shown for members
    → Instead sees [✉ Request Add Person]
    → Eve taps [✉ Request Add Person]
    → Shows inline form: display name, email hint (optional)
    → Eve enters: "Frank", no email
    ⚡ POST /events/{eid}/modification-requests
       {type: "add_person",
        proposed_changes: {display_name: "Frank"},
        message: "Frank shared our taxi"}
    → "Request sent to admin ✓"
    → status: pending
    → Frank is NOT created yet — awaits admin approval in S16
```

Eve tries to remove a person:

```
Eve's S05 → taps "People ▸"
→ Screen: S07
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
Step 1: Eve submits the request
Eve's S10 (Alice's expense) → taps [Suggest Change]
→ Screen: S10 (modification request form)
    → Type: "edit_expense"
    → Target: Restaurant $120
    → Message: "The restaurant bill should be $110 not $120 — receipt shows $110"
    ⚡ POST /events/{eid}/modification-requests
       {type: "edit_expense",
        target_type: "transaction",
        target_id: tid,
        proposed_changes: {amount: 110.00},
        message: "The restaurant bill should be $110 not $120 — receipt shows $110"}
    → "Change request sent to admin ✓"
    → status: pending
```

Alice sees the request in her moderation queue:

```
Step 2: Request appears in admin's queue
Alice's S05 → "⚠️ 1 modification request"
→ Screen: S16
    ⚡ GET /events/{eid}/modification-requests
    → Eve's request listed: "Edit Expense — Restaurant $120"

Step 3: Alice reviews the detail
    → taps [Review ▸]
    ⚡ GET /events/{eid}/modification-requests/{rid}
    → sees proposed changes, Eve's message, impact preview

Step 4a: Alice approves (auto-applies)
    → taps [Approve]
    ⚡ POST /events/{eid}/modification-requests/{rid}/approve
    → status: approved
    → changes auto-applied (amount updated to $110.00 — no separate PATCH needed)
    → Eve notified: "Your change request was approved ✓"

    --- OR ---

Step 4b: Alice rejects
    → taps [Reject]
    → adds reason: "Receipt is correct at $120, tip was included"
    ⚡ POST /events/{eid}/modification-requests/{rid}/reject
       {reason: "Receipt is correct at $120, tip was included"}
    → status: rejected
    → Eve notified: "Your change request was rejected"
    → Eve sees Alice's reason
```

Eve can also withdraw her own pending request:

```
Step 5 (alternative): Eve withdraws before admin action
Eve's S10 → sees her pending request indicator
    ⚡ POST /events/{eid}/modification-requests/{rid}/withdraw
    → status: withdrawn
    → request removed from admin's queue
    → no changes applied
```

## Validates

- 403 boundaries on update, delete, and approve actions
- Clear, role-aware error messages (not generic "forbidden")
- [Suggest Change] offered as alternative to direct edit
- Member person addition redirected to modification request (not direct creation)
- Full modification request lifecycle: create → list → review → approve/reject → withdraw
- 6 modification request endpoints: `POST` (create), `GET` (list), `GET /{id}` (detail), `POST /{id}/approve`, `POST /{id}/reject`, `POST /{id}/withdraw`
- Auto-apply on approval — no separate PATCH needed
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
