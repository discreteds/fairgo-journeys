# Interactive Navigation Map

Click any screen to jump to its specification.

```mermaid
flowchart TD
    S01["S01<br>Welcome"]
    S02["S02<br>Register / Login"]
    S03["S03<br>Home"]
    S04["S04<br>Create Event"]
    S05["S05<br>Event Dashboard"]:::hub
    S06["S06<br>Invite / Join"]
    S07["S07<br>Manage People"]
    S08["S08<br>Manage Groups"]
    S09["S09<br>Add Expense"]
    S10["S10<br>Expense Detail"]
    S11["S11<br>Balances"]
    S12["S12<br>Settle Up"]
    S13["S13<br>Membership"]
    S14["S14<br>Event Funding"]
    S15["S15<br>Profile / Settings"]
    S16["S16<br>Admin Moderation"]
    S17["S17<br>Notifications"]

    %% Entry flow
    S01 -->|"Get Started / Log In"| S02
    S02 -->|"No invite"| S03
    S02 -->|"Has invite"| S05

    %% Home
    S03 -->|"+ New Event"| S04
    S03 -->|"Tap event"| S05

    %% Create event
    S04 -->|"Event created"| S05

    %% Dashboard hub connections
    S05 -->|"Invite Link"| S06
    S05 -->|"People"| S07
    S05 -->|"Groups"| S08
    S05 -->|"Add Expense"| S09
    S05 -->|"Tap transaction"| S10
    S05 -->|"Balances"| S11
    S05 -->|"Settle Up"| S12
    S05 -->|"Funding Status"| S14
    S05 -->|"Pending actions"| S16

    %% Bottom nav (global)
    S05 -.->|"Bottom nav"| S03
    S05 -.->|"Bottom nav"| S17
    S05 -.->|"Bottom nav"| S15

    %% Expense flow
    S09 -->|"Saved"| S05
    S10 --> S11

    %% Settlement flow
    S11 -->|"Settle Up"| S12
    S12 -->|"Done"| S11

    %% Membership flow
    S15 -->|"Membership"| S13

    %% Admin flow
    S16 -->|"Review"| S10
    S16 --> S07
    S16 --> S08
    S16 --> S14

    %% Invitation return
    S06 -.->|"Invitee arrives"| S01

    %% Styles
    classDef hub fill:#42b983,stroke:#333,stroke-width:3px,color:#fff

    %% Click handlers - navigate to screen specs
    click S01 "#/screens/S01-welcome"
    click S02 "#/screens/S02-register-login"
    click S03 "#/screens/S03-home"
    click S04 "#/screens/S04-create-event"
    click S05 "#/screens/S05-event-dashboard"
    click S06 "#/screens/S06-invite-join"
    click S07 "#/screens/S07-manage-people"
    click S08 "#/screens/S08-manage-groups"
    click S09 "#/screens/S09-add-expense"
    click S10 "#/screens/S10-expense-detail"
    click S11 "#/screens/S11-balances"
    click S12 "#/screens/S12-settle-up"
    click S13 "#/screens/S13-membership"
    click S14 "#/screens/S14-event-funding"
    click S15 "#/screens/S15-profile-settings"
    click S16 "#/screens/S16-admin-moderation"
    click S17 "#/screens/S17-notifications"
```

**S05 Event Dashboard** (highlighted in green) is the central hub. All journey rails pass through it.

## Journey Rails

| Rail | Path | Description |
|------|------|-------------|
| [R01 Onboarding](/rails/R01-onboarding-rail.md) | S01 → S02 → S03 → S04 → S05 | New user signs up and creates first event |
| [R02 Expense](/rails/R02-expense-rail.md) | S05 → S09 → S05 → S10 → S11 | Add an expense and view balances |
| [R03 Settlement](/rails/R03-settlement-rail.md) | S11 → S12 → S11 → S05 | Settle debts between participants |
| [R04 Invitation](/rails/R04-invitation-rail.md) | S05 → S06 ··· S01 → S02 → S05 | Send invite, invitee joins event |
| [R05 Membership](/rails/R05-membership-rail.md) | S15 → S13 → S05 → S14 | Manage subscription and event funding |
| [R06 Admin](/rails/R06-admin-rail.md) | S05 → S16 → S07/S08/S10/S14 | Review and moderate pending actions |
