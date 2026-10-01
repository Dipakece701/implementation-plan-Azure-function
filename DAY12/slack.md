Hi team, update on the Azure Functions PROD remediation:

I Completed and verified **PROD Tasks #30847 and #30852**.

**#30847:** Wired Application Insights for "helios-prod-cost-ingestion" to "platform-backend-insights-prod" and verified the app is running with telemetry enabled.

**#30852:** Implemented the Durable Orchestrator failure alert on "func-orchestrator-sopfactoryprod1fd3k", with 5-minute evaluation and alerts routed to "ag-helios-prod-ops".
* Next: I am working on **— #30850 inbound access restrictions**.

No blockers from my side.
