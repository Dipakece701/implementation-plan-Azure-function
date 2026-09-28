# OPENINGS
> I am working on Azure Functions QA remediation.
>
> Yesterday, I completed the two tasks, #30172 and #30170.
>
> **For Task #30172, Managed Identity:**
>
> I resolved the identity gap on the two "UUDRI" "Function App": "UUDRI-bill-processor-qa-01" and "UUDRI-Function-App-qa-01".
>
> I enabled "System-Assigned" Managed Identity on both apps and verified that both apss are running normally. With this change, all 10 QA Function Apps now have active Managed Identities, And giving 100% identity coverage for the "QA Function App" fleet.
---------------------------------------------------
> **For Task #30170, Hosting Tier and AlwaysOn:**
>
> Thankyou  Maurice to help on this 
> 
> I enabled "AlwaysOn = true" on the Dedicated S1 "UUDRI-Function-App-qa-01" with zero downtime.
>
> I also documented the "hosting-tier decisions": The 8 Dynamic Y1 apps remain by design, the EP1 app keeps "AlwaysOn = false" because it already has one always-ready instance, and S1 outbound VNet is deferred because the app currently has no deployed functions and no VNet in that resource group.
>
> All 8 of 8 QA remediation tasks under Parent Story #30165 are now complete.
>
> And I created the PROD Parent Story #30845, and provisioned the child tasks.
> Next, I’ll start the PROD implementation wave, focusing on "Application Insights", "diagnostic settings", and the "Durable Orchestrator failure alert".
>
> No blockers from my side. Thats all form my side Thankyou
