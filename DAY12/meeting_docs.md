# Openings
>I am Working on Azure Functions PROD remediation.
>
>Yesterday, I completed two tasks, #30847 and #30852,
>
>**For Task #30847: "Wire Application Insights on helios-prod-cost-ingestion":**
>
>I connected "helios-prod-cost-ingestion" to the production "platform-backend-insights-prod" instance using the "APPLICATION-INSIGHTS_CONNECTION_STRING".
>
>And I verified the configuration live and confirmed the "Function App" remained in a Running state with no downtime. The "cost-ingestion" workload now has runtime tracing and exception telemetry.
>
--------------------------------------------
> **For Task #30852: "Add a durable orchestrator failure alert on func-orchestrator-sop-factor-prod"**
>
> I addressed the asynchronous failure gap on "func-orchestrator-sop-factory-prod-1fd3k"
>
> And Durable Functions can return HTTP 202 while the actual orchestration continues in the background, standard HTTP failure monitoring can miss those failures.
>
> I deployed the Scheduled Query Rule "alert-sopfactory-orchestrator-failure-prod" on "appi-sopfactory-prod".
>
> It evaluates every 5 minutes, uses a 5-minute lookback, routes alerts to "ag-helios-prod-ops" at Severity 1, and has "auto-mitigation" enabled. I verified the alert configuration in Live Azure.
>
> so both task is completed and closed.
>
> Next, I’ll working on task #30850 for inbound access restrictions.
>
> No blockers from my side.
