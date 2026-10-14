# OPENINGS
>Yesterday, I worked on Ticket AB#34088 — "enabling managed identity-based telemetry for the DEMO environment", through PR #663.
>
>First, I configured the Monitoring Metrics Publisher role assignments for all five telemetry senders targeting "platform-backend-insights-dm0".
>
>These include the Knowledge Graph event processor, GitHub Activity Logger, Ontology event processor, EMS Plan Narration Function App, and the UI App Service.
>
>Then I validated all five Terraform plans through GitHub Actions. All passed successfully with zero resources to destroy.
>
>And During validation, I fixed two resource naming mismatches that could have caused unnecessary resource recreation: one involving the UI managed identity and another involving the GitHub Activity Logger storage account.
>
>I also addressed Soomin’s review comments.We separated the rollout into two phases: first, deploy the RBAC permissions; then, after allowing time for permission propagation, enable Entra ID authentication.
>

>
>Next steps are to merge the PR, apply the RBAC changes across all five stacks,, wait for permission propagation, and then enable Entra ID authentication
>
>No blockers from my side.
