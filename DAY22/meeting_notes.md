# OPENINGS 
>Yesterday, I completed Ticket AB#34088, which covers managed identity-based telemetry for the five DEMO Function Apps and UI senders.
>
>And I applied all five Terraform stacks with zero resource destructions, enabled Entra ID authentication, and verified the telemetry in Log Analytics.
>
>Then I validate the successful HTTP 200 traffic with zero HTTP 403 errors. Soomin approved the changes, and PR #668 was merged into main.
-------------------------------------------------
>Second
>I worked on Ticket AB#34089, the AKS OpenTelemetry sidecar Workload Identity migration.
>
>I completed the architecture investigation under AB#34092 and inventoried all 22 DEMO services using the "azmon-sidecar", mapped to 17 unique managed identities under AB#34093.
>
> The current collector version 0.120.0 does not support the required azure_auth configuration, so we need to upgrade to 0.140.0. This also allows us to remove the legacy transform/azmon-compat processor.
>
>And I also confirmed that Kubernetes webhook reinvocation supports token injection into the sidecar.
>
>Next, I’ll align with the team on Terraform role-assignment placement, prepare the RBAC changes for the 17 identities, and update the sidecar configuration.

>The proposed rollout starts with tariff-api as the canary before restarting the remaining workloads.

>No blockers from my side
