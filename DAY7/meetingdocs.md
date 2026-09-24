# OPENINGS
>Hi everyone, I am working on Azure Function QA Implementation Plan
>
>Yesterday, I completed and verified two tasks: #30166 and #30168.
>
>FOR **Task #30166, Diagnostic Settings:**
>
>The goal was to make sure all QA Function Apps have platform logging and metrics configured correctly. I followed a staged rollout. First i do "diag-helios-qa" diagnostic setting on "helios-device-telemetry-qa-func" and verified that the configuration was correctly bound to the "Function App" and pointing to the "helios-qa-logs"  Analytics workspace.
>
>After the pilot was validated, I rolled the same configuration out to the remaining apps. Then I performed a full "subscription-level" to the QA "Function Apps". The result is now 10 out of 10 apps is working, with active diagnostic settings. The configuration is collecting "FunctionAppLogs" and "AllMetrics" and routing them to "helios-qa-logs".
>
------------------------------------------------------------
>**For Task #30168**: "Durable Orchestrator Failure Alerting",
>
>The main issue was that normal "HTTP-based" monitoring does not fully cover Durable Functions. The initial HTTP request can return 202 Accepted, while the actual orchestration continues asynchronously and can fail later in the background.
>
>Then I created a Scheduled Query Rule to "appi-sopfactory-qa".
>
>The KQL specifically looks for failed "sop-Factory-Orchestrator" requests and orchestration operations. The rule evaluates every 5 minutes with a 5-minute lookback window, uses Severity 1, and routes notifications to the "ag-helios-qa-ops" Action Group. And "Auto-mitigation" is also enabled so the alert can automatically resolve when subsequent executions succeed.
>
>After That I verified the alert configuration live using Azure Monitor, including the target Application Insights resource, query, schedule, severity, action group, and auto-mitigation settings.
>
>So Both task #30166 and task#30168 are closed.
>
>No blockers from my side. Thats all form my side Thankyou
