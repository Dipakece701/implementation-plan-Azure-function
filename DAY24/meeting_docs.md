# OPENINGS
>Friday I worked on  DEMO environment.
>
>The telemetry gaps in the DEMO environment, particularly for "cost-ingestion" and "github-activity-logger".
>
> I found that the application was not connected to the shared "platform-backend-insights-dm0" component. I Created a PR #689 in "helios-infra" to enable the correct wiring, following the existing QA and PROD configuration.
>
> All four CI pipelines are green, including Terraform contract tests, security scanning, isolated plan validation, and drift and monitoring regression checks. The PR is now awaiting For Constantin’s review.
>
>**For github-activity-logger**:
>
>I confirmed that the managed identity, RBAC permissions, and Application Insights configuration are correct. The actual issue is a platform-level HTTP 503 caused by a missing application deployment package in site/wwwroot. Since the dm0 deployment workflow is not configured in the application repository. PROD is already healthy and returning HTTP 200.
>
> Then I also verified that "kg-event-processor", the ontology event processor, and EMS Plan Narration are reporting telemetry correctly.
>
>For AB#34089, the OpenTelemetry Workload Identity investigation and inventory are complete.I mapped 22 AKS services to 17 managed identities. I’m waiting for confirmation from Soomin on where to define the Terraform role assignments before raising the implementation PR.
>
>My next steps are to follow up on PR #689, progress the 17 role assignments once confirmed.
>
>No blocker from my side.
