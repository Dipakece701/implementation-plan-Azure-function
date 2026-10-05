# OPENINGS
>Friday I am working on the Azure Functions PROD remediation and observability work.
>
>**For Task #30846: for Key Vault RBAC migration.**
>
> I and Maurice migrated the final two legacy Key Vaults, "helios-prod-ui-kv" and "helios-prd-spkplg-pki-kv", to Azure RBAC.
>
>We pre-assigned the required "least-privilege" roles before the cutover and verified the UI application after the change with HTTP 200 and no downtime. And the PROD Key Vault estate to 9 out of 9 using Azure RBAC
---------------------------------
## And On the Observability part 
> I Worked on 3 Task.
>
> **For AB#31394:"Function apps send no telemetry: private-only App Insights without VNet" and PR #595 **:
>
> The three Consumption Function Apps to use the public "Application Insights" "ingestion" endpoint while still sending telemetry to the same central "helios-prod-log" workspace.
>
> Now constatin add some comment so  I'll work on that.
>
> **For AB#31392:Alert Slack relay returns NotFound (DEMO since 30 Sep, QA last 2 runs)  **:
>
> I enabled System-Assigned Managed Identity on the four Logic Apps across PROD and QA and updated the workflows to retrieve Slack webhook secrets from Key Vault using MSI, with secure output masking enabled. This removes the hardcoded webhook URLs.
>
> **For AB#31395: PROD UI App Service sends no telemetry (empty instrumentation key) **:
>
> the PROD UI Application Insights configuration has been prepared, and I’m currently waiting for Charles's confirmation before applying it in PROD.
>
> So the main next steps i am working on comment.
>
>After this work i'll work on finish the remaining PROD Task promotion-drift item for Azure function.
>
> No blockers from my side.
