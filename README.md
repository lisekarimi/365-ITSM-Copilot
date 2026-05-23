# 365 ITSM Copilot

A **Microsoft Copilot Studio agent connected to ServiceNow** (`ITSM Support Agent`) that acts as an internal IT Service Desk assistant.

## What this agent does

| Capability | Description |
|------------|-------------|
| 🔍 Resolve IT issues | Answers IT support questions **grounded** in the ServiceNow **Knowledge Base** (RAG), with citations — no external or general knowledge. |
| 🎫 Create a ticket | When the KB can't resolve the issue (or it isn't covered), creates a ServiceNow **incident**. **AI Builder** classifies the issue into short description, description, urgency and category before the ticket is raised. |
| 📋 View recent tickets | Lists the user's 5 most recent ServiceNow incidents. |

Scope is limited to IT topics (password resets, VPN, software installs, hardware, access requests); off-topic requests are politely declined.

### How it decides
1. User describes a problem → the issue description is captured.
2. Can the **Knowledge Base** answer it? **Yes →** guide the user through the KB solution.
3. Ask if it's resolved. **No / not in KB →** offer to create a ticket.
4. On confirmation → create the ServiceNow incident and return the ticket number.

On conversation start the agent looks the user up in the ServiceNow `sys_user` table (by M365 email) and stores their `sys_id`, which is used as `caller_id` for ticket creation and lookups.

## Prerequisites

- **Microsoft Power Platform / Power Apps** account → https://make.powerapps.com
- **Microsoft Copilot Studio** account → https://copilotstudio.microsoft.com
- **ServiceNow** free developer instance → https://developer.servicenow.com/dev.do
  *(Create an instance for free — the Knowledge Base and Incident tables come pre-populated with demo data automatically.)*

## Build order

The agent, the Power Automate flow, the AI Builder prompt and the connection references are all packaged together as a **solution** (note the `lkar_` publisher prefix), so it can be promoted across environments via a **Power Platform pipeline**. Build in this order:

1. **Power Apps** (https://make.powerapps.com) → develop in the **Dev** environment *(Dataverse-backed — required for AI Builder and the Dataverse connector used by the ticket flow).*
2. Create a **solution** in Dev.
3. Inside the solution, create the **agent** (+ ticket flow and AI Builder prompt).
4. Use a **pipeline** to deploy the solution from Dev → UAT → Prod.

### Environment topology

| Environment | Type |
|-------------|------|
| Host (pipeline host) | Production (Managed) |
| Dev | Sandbox (Managed) |
| UAT | Sandbox (Managed) |
| Prod | Production (Managed) |

## Setup

For the full step-by-step ServiceNow ↔ Copilot Studio configuration (OAuth app registry, client credentials, redirect URL, consent, adding the knowledge source), see:

📄 [`ressources/servicenow-copilot-setup.pdf`](ressources/servicenow-copilot-setup.pdf)

## Telemetry & analytics

The agent emits Application Insights events (`KBResolved`, `TicketCreated`) to measure deflection, resolution rate, and hours saved — surfaced in a Power BI KPI dashboard.

📊 [`ressources/powerbi-app-insights.md`](ressources/powerbi-app-insights.md) — connect App Insights to Power BI.

## Architecture & flows

See [`ressources/agent-schema.md`](ressources/agent-schema.md) for the user journey and technical architecture diagrams.
