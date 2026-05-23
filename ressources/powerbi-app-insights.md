# Connect Application Insights to Power BI

Build the KPI dashboard from the agent's telemetry (`customEvents`).

## Prerequisites

- Power BI Desktop installed
- Access to your Application Insights resource in Azure

## Steps

1. **Azure Portal** → your Application Insights resource → **Configure → API Access** → copy the **Application ID**.
2. **Application Insights → Logs** → run your query → **Export → Export to Power BI (M query)** → downloads a `.txt` file.
3. **Power BI Desktop → Get Data → Blank Query → Advanced Editor**.
4. Open the `.txt`, copy the M query, paste it into the Advanced Editor → **Done**.
5. **Edit Credentials → Organizational account → Sign in** with your Azure account → **Connect**.
6. The `customEvents` table loads with: `timestamp`, `name`, `customDimensions`, `session_Id`, `user_Id`.
7. **Close & Apply**.

## Privacy

- Do **not** expose `user_Id` or `user_AccountId` in dashboard visuals.
- For public-facing reports, use only `name`, `timestamp`, and `session_Id`.
