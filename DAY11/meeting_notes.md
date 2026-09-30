# OPENING
>I am working on Azure Functions PROD  remediation.
>
>I completed and verified the two tasks, #30848 and #30849.
>**For Task #30848: Configure diagnostic settings across all 8 PROD function apps**:
>
>The baseline was zero out of 8 PROD "Function Apps" with diagnostic settings.
>
>I followed the staged rollout approach, first piloting "diag-helios-prod" on "helios-prod-cost-ingestion", verified the configuration, and then rolled it out across the remaining 7 apps.
>
>Now all 8 out of 8 PROD "Function Apps" are streaming "FunctionAppLogs" and "AllMetrics" to the "helios-prod-logs" Log Analytics workspace.
------------------------------------------------
>**For Task #30849: Verify availability web tests in PROD for kg-event-processor and ontology-event-processor:**
>
>Maurice deployed the synthetic tests through Terraform, and I completed the live verification.
>
>Both "kg-event-processor-prod" and "helios-ontology-event-processor-func-prod" have their availability tests enabled, and the /api/health endpoints are returning HTTP 200.
>
>Both tasks are complete and closed.
>
>Next, I’ll move to Task #30847, wiring Application Insights for helios-prod-cost-ingestion.
>
>No blockers from my side.
