# OPENINGS
>Yesterday, I completed Ticket AB#35220, related to tuning the DEMO UI HTTP 5xx alert.
>
>I created a PR #702 that was merged, and the Terraform apply completed successfully. Then I verified in Azure that the alert threshold is now set to 25, with Severity 2 enabled.
>
>For Ticket AB#34089, I completed the technical spike and opened PR #703 for the AKS Workload Identity implementation.
>
>The change grants the Monitoring Metrics Publisher role to 17 managed identities. The Terraform plan passed with 17 resources to add and zero changes or destructions.
>
>The PR is currently awaiting Soomin’s review. And I also created Ticket AB#35215 to track upgrading the OpenTelemetry sidecar collector to version 0.140.0.
>
>Next I am going to work on follow up on PR #703 approval, and prepare the GitOps change for the collector upgrade.
>
>No  blockers from my side
