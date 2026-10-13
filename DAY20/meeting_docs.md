# OPENINGS
>Yesterday, I worked on Ticket AB#34086 and AB#34087: DEMO Application Insights consolidation. we changed the approach after reviewing it with Soomin.
>
>Instead of creating a separate public Application Insights instance, we decided to use the existing "platform-backend-insights-dm0" as the single telemetry destination.
>
>The PR #653 was closed as superseded, and also worked on PR #655 is (open public ingestion on "platform-backend-insights-dm0").
>
>And PR #655, merged into main. We also kept the AMPLS ingestion mode set to PrivateOnly to preserve the security boundary for VNet-connected workloads.
>
>The Terraform apply completed successfully in three seconds, with zero resources added or destroyed and one resource changed.
-------------------------------------------
>
>Next, I am going to verify DEMO telemetry, and start work on Ticket AB#34088 for Entra ID authentication using managed identities.
>
>No blockers from my side.
