# OPENINGS
>Hi everyone, quick update on the Azure Functions SRE remediation:
>
>On DEV, all three tasks assigned to me under Ticket #29301 are now completed, verified, and closed.
>
>Maurice has also completed the related Terraform monitoring tasks Ticket #29302, #29304, and #29306, including availability tests across DEV, QA, and PROD.
>
>Before starting the QA and PROD rollout, I completed a read-only audit of both environments and updated the SRE Remediation Plans in ADO Wiki.
>
-------------------------------------------------------------

# For Foundry Model Implementation Plan 
>**For DEV,** I look Sam’s review feedback into the remediation plan.
>
>For the P0 authentication issue on ca-model-service-dev, I changed the approach to a 3-phase rollout — first the caller tokens, then deploy an auth-enabled canary, and finally enforce authentication and restrict the ingress.
>
>I also aligned the implementation plan with the existing ai_foundry Terraform module instead of introducing a separate deployment approach. I also updated the alerting design to use KQL Scheduled Query Rules for 429s, 5xx errors, and P95/P99 latency.
>
>**For QA,** authentication is already enabled, and I also addressed the model alias issue and confirmed the telemetry routing.
>
>**For PROD**, I identified two major issues: the main concern is the single-replica risk on "ca-model-service-prod". It is currently at "minReplicas = 1", so the plan is to move it to 2 replicas for better availability.
>
>I also identified a 10,000 TPM quota limitation on gpt-5.4-mini in PROD, which needs an expedited Azure quota increase before higher production usage.
>
>No blockers from my side. Thats all form my side Thankyou
