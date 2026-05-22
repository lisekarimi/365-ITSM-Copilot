# Schema — 365 ITSM Copilot

## User journey

```mermaid
flowchart TD
    U([User signs in]) --> D[Describes IT issue]
    D --> KB{In Knowledge Base?}
    KB -->|Yes| ANS[Agent guides through solution]
    ANS --> R{Resolved?}
    R -->|Yes| DONE([Anything else?])
    R -->|No| OFFER
    KB -->|No| OFFER[Offer to create a ticket]
    OFFER -->|User accepts| TKT[Ticket created<br/>returns INC number]

    U -.->|"My recent tickets?"| LIST[Show 5 latest tickets]
```

## Agent flow (topics)

Routing is **generative orchestration** — the agent follows its instructions and chooses the topic; this is not a hard-wired switch.

```mermaid
flowchart TD
    Start([Conversation start]) --> Lookup[ConversationStart:<br/>sys_user lookup → UserSysId]
    Lookup --> Orch{{Orchestrator routes by intent}}

    Orch --> Cap[Capture Issue<br/>→ UserIssueDescription]
    Orch --> Know[Knowledge<br/>→ grounded answer]
    Orch --> Create[Create Ticket<br/>flow + AI Builder → INC]
    Orch --> Recent[Get Recent Tickets<br/>→ top 5]

    Cap -.->|UserIssueDescription| Create
```

## Technical architecture

```mermaid
flowchart LR
    User([User]) --> Agent

    subgraph CS[Copilot Studio - ITSM Support Agent]
        direction TB
        Agent[Agent orchestration]
        KBsrc[Knowledge source]
        Flow[Create Ticket<br/>+ AI Builder]
        Agent --- KBsrc
        Agent --- Flow
    end

    Agent --> Conn
    KBsrc --> Conn
    Flow --> Conn

    Conn{{ServiceNow connector<br/>OAuth2}}

    Conn --> KB
    Conn --> INC
    Conn --> USR

    subgraph SN[ServiceNow]
        direction TB
        KB[(Knowledge Base)]
        INC[(Incidents)]
        USR[(sys_user)]
    end
```

**Notes**
- On start, the agent looks up the user in `sys_user` (by M365 email) → stores `sys_id` as `caller_id`.
- Ticket creation runs a Power Automate flow: **AI Builder** turns the free-text issue into `short_description`, `description`, `urgency`, `category`, then creates the incident.
- Recent tickets = ServiceNow query on `incident` filtered by `caller_id`, newest first, top 5.

## Ticket creation flow

The `CreateServiceNowIncident` Power Automate flow turns the user's free-text issue into a structured incident.

```mermaid
flowchart TD
    IN[/UserIssueDescription + UserSysId/] --> PROMPT[AI Builder prompt<br/>LLM classifies the issue]
    PROMPT --> OUT[["Structured output:<br/>short_description<br/>description<br/>urgency<br/>category"]]
    OUT --> CREATE[ServiceNow CreateRecord<br/>incident · caller_id = UserSysId]
    CREATE --> NUM[/Returns INC number/]
```
