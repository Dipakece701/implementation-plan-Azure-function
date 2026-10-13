# OPENINGS
>Yesterday, I worked on Ticket AB#34086 and AB#34087: DEMO Application Insights consolidation. we changed the approach after reviewing it with Soomin.
>
>Instead of creating a separate public Application Insights instance, we decided to use the existing "platform-backend-insights-dm0" as the single telemetry destination.
>
>The earlier PR #653 was closed as superseded, and also worked on PR #655 to enable public ingestion on the existing component.
>
>Soomin approved PR #655, and it has now been merged into main. We also kept the AMPLS ingestion mode set to PrivateOnly to preserve the security boundary for VNet-connected workloads.
>
>The Terraform apply completed successfully in three seconds, with zero resources added or destroyed and one resource changed.
-------------------------------------------
>Apart form that i also worked on  Non-AKS Azure Estate implementation plan for DEV Enviroment.
>
>I aligned with Constantin to prioritize "GAP-003" diagnostic settings and "GAP-010" Application Insights integration,implemented strictly through Terraform.
>
>Next, I am going to verify DEMO telemetry, and start work on Ticket AB#34088 for Entra ID authentication using managed identities.
>
>No blockers from my side.
