# OPENINGS
>yesterday I am working on  Observability part.
>The three foundational observability remediation items are now completed and closed in Azure Boards.
>
>**For AB#31392:Alert Slack relay returns NotFound**
>
>And PR #601 was merged into main. 
>
>All four Alert Slack Relay Logic Apps across QA and PROD to use "System-Assigned" Managed Identity and retrieve the Slack webhook secrets directly from Key Vault with least-privilege access.
>
>I also enabled secure input and output masking. I validated this with four synthetic alerts across QA and PROD, and all four were successfully delivered with HTTP 200 in under 1.2 seconds.
-----------------------------------------------
>**For AB#31394: Function apps send no telemetry: private-only App Insights without VNet **
>
>And PR #595 was also merged.
>
>The two non-VNet PROD Function Apps, "kg-event-processor-prod" and "helios-github-activity-logger-prod-func", are now sending telemetry through the public Application Insights ingestion endpoint using Entra ID authentication and managed identity.
>
> I verified that "AppRequests", "AppTraces", and "AppMetrics" are flowing into the central "helios-prod-logs" workspace.
-----------------------------------------------
>**For AB#31395: PROD UI App Service sends no telemetry**
>
> the PROD UI App Service telemetry configuration is also completed and closed, with telemetry confirmed in the production Log Analytics workspace.
>
>During the validation, I also identified the root cause of the GitHub Activity Logger 503 issue. The issue is related to "WEBSITE_RUN_FROM_PACKAGE" = "1" drift on the Linux Consumption app. I created AB#33231 for the follow-up remediation.
>
>For the next priority, I’m starting Ticket** AB#33180 — the Grafana DEMO Health Board, or the “One DEMO Board.” **
>
>My next steps are to create demo-health.json, register it in the Terraform monitoring stack, validate the dashboard rendering, and run the Terraform contract tests.
>
>No blockers from my side.
