# OPENINGS 
# For Foundry Models
>I completed the validation across DEV, QA, and PROD, and all three implementation plans are  approved by Sam.
>
>For DEV, I finalized the 3-phase approach for the P0 authentication issue — first audit the caller tokens, then deploy and validate an auth-enabled canary, and finally enforce authentication and ingress restrictions.
>
>And I also updated the Terraform implementation to use the "native model" and "scale structures" from the existing "ai_foundry" module and standardized the Container App health probe configuration.
>
>For QA, I identified an important caller dependency. The orchestrator calling the model service is running on a Consumption "Function App" without "VNet integration". So before making the Container App internal, we need to address the "VNet integration" first.
>
>For PROD, The main P0 is the "single-replica" risk on "ca-model-service-prod". The agreed change is to move minReplicas from 1 to 2, along with health probes and autoscaling.
>
>And I also corrected an important architecture point in PROD: "**ca-model-service-prod**" is using its own "**ais-sopfactory-prod**" account. The actual consumer of the "**central PROD AI Hub**" is "**ems-plan-narration-function-prod**", so the quota planning is now scoped to the correct service.
>
>
-----------------------------------------------------------------
# FOR AZURE FUNCTION IMPLEMNTATION 
>AND update on the Azure Functions SRE remediation for QA.
>
>Following the completion and verification of the DEV remediation, I’ve now move to the QA remediation work under Parent Story #30165, linked to the QA discovery story #26565.
>
>I have mapped all 8 identified QA gaps into ADO child tasks. Task #30174 for availability web tests is already closed, with Maurice’s Terraform deployment verified and both health endpoints returning HTTP 200.
>
>The remaining 7 tasks, #30166 through #30172, are created and ready for implementation. These cover "diagnostic settings", "inbound access restrictions", "Durable Orchestrator failure alerting", "Application Insights for the cost-ingestion app", "the Y1 Consumption tier decision", "Key Vault RBAC", and managed identity for the UUDRI apps.
>
>Sam added comment on Ticket #30165 so I'll work on it.
>
> There is No blockers from my side. That's all form my side Thankyou
