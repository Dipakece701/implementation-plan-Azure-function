# OPENINGS

> Hi everyone, I am working  on the Azure function QA Implementation.
>
> Yesterday, I completed and verified two more tasks, #30169 and #30171, under Parent Story #30165.
>
> **For Task #30169**- "**Application Insights wiring**"
>
> I configured "APPLICATIONINSIGHTS_CONNECTION_STRING" on the "helios-qa-cost-ingestion" "Function App" and connected it to our centralized "platform-backend-insights-qa" Application Insights instance.
>
> Previously, this application do not have Application Insights connected, so failures or execution issues from the daily timer job could go unrecorded.
>
> After the configuration change, the "Function App" is running normally and runtime telemetry such as "requests", "traces", "exceptions", "execution duration", and dependency information can now be captured.
---------------------------------------------------
> **For Task #30171** - **Key Vault RBAC migration**:
>
> Me and Maurice, migrated the remaining 3 legacy QA Key Vaults from Access Policies to Azure RBAC: "UUDRI-Key-Vault-qa-02", "helios-qa-ui-kv", and "helios-qa-spkplug-pki-kv".
>
> Before the cutover, we pre-assigned the required RBAC roles to the application identities and Terraform service principal so that switching RBAC on would not interrupt access. We also followed the principle of least privilege.
>
> After the migration, I performed a subscription-wide audit and confirmed 11 out of 11 QA Key Vaults are now using Azure RBAC, with no legacy access policies remaining. I also validated the QA UI and backend services after the cutover; they continued returning HTTP 200 with no authorization errors or downtime.
>
> Both #30169 and #30171 are completed, Next, I’ll continue with Task #30167 for inbound access restrictions,
>
> No blockers from my side.Thats all form my side Thankyou
