# OPENINGS
>Yesterday, I Worked on DEMO cost-ingestion telemetry.
>
>Ticket AB#34991 is completed. I resolved the DEMO "cost-ingestion" telemetry gap by merging PR #689 to connect the Function App to "platform-backend-insights-dm0" and PR #696 to add the automated deployment workflow.
>
>Both the Terraform apply and function deployment completed successfully, and the function package is deployed and active.
>
>Then I have been working on **Ticket AB#34970, the SRE tabletop exercise for SOC 2 CC7.**;
>
>I prepared the Incident Procedure v1.0, the scenario brief, and the live-notes template.
>
>The exercise covers two scenarios: an APIM HTTP 5xx spike and unauthorized Key Vault secret retrieval. The session is planned for Wednesday, October 21, from 10:15 to 11:15 AM.
>
>**For AB#34089,** Soomin confirmed Option A for the AKS Workload Identity implementation. We’ll define the 17 Monitoring Metrics Publisher role assignments in terraform/stacks/aks/main.tf, alongside the managed identities.
>
>The inventory of 22 services is complete, and the technical investigation confirmed that upgrading the OpenTelemetry Collector to version 0.140.0 enables native Entra ID authentication.
>
>Next i am going to work on raise the Terraform RBAC PR, create the dedicated collector-upgrade task, update the Azure DevOps work items.
>
>No Blocker from my side.
