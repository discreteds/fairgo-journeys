# SC07 — Charlie Claims Their Person (Identity Resolution)

**Persona:** Charlie — was added by name to Alice's event, later registers and self-claims.
**Screens:** S01 → S02 → S05 → S07 (match suggestion banner)
**Rails:** R01 (Onboarding), R06 (Admin)

## Narrative

Alice has an event with a placeholder person "Charlie" who hasn't registered yet:

```
Alice's S07 Manage People
    → "Charlie" exists as placeholder (resolution_status: placeholder)
    → no linked user, no email
    → Charlie's singleton PFG auto-created
```

Charlie decides to register and join the event via invite link:

```
→ Opens invite link: fairgo.app/join/K9pR3x
S01 Welcome → deep link detected
S02 Register → creates account (display name: "Charlie")
    ⚡ POST /auth/register → tokens stored
    ⚡ POST /events/join {invite_code: "K9pR3x"}
       → event_role created (status: pending_approval)
       → new person auto-created (resolution_status: auto_created)
       → response includes: potential_matches: [{person_id: placeholder_id, confidence: "medium", reason: "display_name_match"}]
```

After Alice approves, Charlie sees a match suggestion:

```
Alice's S05 → approves Charlie's join request
    ⚡ POST /events/{eid}/event-roles/{rid}/approve

Charlie's S05 Event Dashboard
    → "We found a match!" banner
    ⚡ GET /events/{eid}/persons/my-matches
       → [{source_person_id: placeholder_id, display_name: "Charlie", confidence: "medium"}]
    → "A person named 'Charlie' already exists in this event. Is this you?"
    → [This is me] [Not me]
```

Charlie taps "This is me" to self-claim:

```
Charlie's S05 → taps [This is me]
    ⚡ POST /events/{eid}/persons/{placeholder_id}/merge {target_person_id: charlie_person_id}
       Headers: Idempotency-Key: <uuid> (see A07)
       → authorize_self_merge() validates:
         - source is placeholder (resolution_status: placeholder)
         - target belongs to requesting user
         - display names match (confidence threshold met)
       → merge executes:
         - all splits from placeholder transferred to Charlie's person
         - placeholder removed
         - PFG reassigned (Charlie inherits placeholder's PFG or gets singleton)
       → resolution_status: claimed
    → banner disappears
    → Charlie sees correct balances reflecting transferred splits
```

## Variant: Admin-Initiated Merge

Alice merges on Charlie's behalf from the people management screen:

```
Alice's S07 Manage People
    → sees two "Charlie" entries (placeholder + auto_created)
    → ⚠️ "Possible duplicate detected"
    → taps [Merge] on the placeholder
    → picker: "Merge into which person?" → selects Charlie's auto_created person
    ! POST /events/{eid}/persons/{placeholder_id}/merge {target_person_id: charlie_person_id}
       → admin merge: no self-merge validation, admin permission check instead
       → same merge mechanics: splits transferred, placeholder removed
```

## Variant: No Match (Different Names)

If Charlie registered as "Charles" (no name match):

```
Charlie's S05 → no match banner shown
    → no potential_matches returned (name similarity below threshold)
    → Alice must manually identify the duplicate from S07
    → or Charlie exists as a separate person (no merge needed if truly different)
```

## Validates

- Self-claim endpoint with merge mechanics
- Match suggestions via `GET /events/{eid}/persons/my-matches`
- Display name matching with confidence levels
- Merge flow: splits transferred, placeholder removed, PFG reassigned
- Duplicate detection on join (potential_matches in join response)
- Admin-initiated merge as alternative to self-claim
- Resolution status transitions: placeholder → claimed

## Key Principle Demonstrated

> **"Person ≠ User"**
>
> Charlie existed as a person (financial participant) before registering as a user.
> The identity resolution flow bridges this gap — merging the placeholder with
> the authenticated identity when Charlie finally joins.
