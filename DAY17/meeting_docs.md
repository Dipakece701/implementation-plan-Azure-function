# OPENINGS
>yestday, I was working on Observability, Azure function  and Non aks Azure estate task.
>
>**For Azure function First, PROD Gap 1, Ticket AB#30853, is now  closed. **
>
>Constantin signed off on the narrower ADR and confirmed that the UUDRI workloads belong to Chenlu’s team and are outside our PROD drift scope. And EMS Plan Narration is running live in PROD with both "plan_narration_agent" and "realized_kpi_listener", and the SOP Factory promotion has been safely handed over to Sudhir under AB#28606.
>
>The Weather Eventstream Monitor is also decoupled and tracked separately under Ticket AB#33495.
---------------------------------------------------------------
Second, for the DEMO monitoring work, AB#33227 and I created a PR #558, 
>The branch is rebased cleanly against main, including Maurice’s runner fixes. Both Slack relay Logic Apps have their System-Assigned Managed Identities active, and all CI validation is green with zero destructions across the environments.
>
> I have published the validation results and re-requested review from Soomin.
>Once PR #558 gets approved, the plan is to merge it, apply the dm0 monitoring changes, then apply "dm0 grafana-alerts", and finally trigger synthetic alerts into the DEMO Slack channel to verify the complete flow.
---------------------------------------------------------------
Third, Non Aks Azure estate DEV  implementation plan
>I added Revision 5 based on the latest live Azure.
>
>Constantin reviewed it and approved to start "GAP-003" diagnostic settings and "GAP-010" Application Insights in Terraform, but we need to provide the expected log ingestion volume and cost estimate first. The other GAP are being held for team review.
>
>So my focus is getting a approval on PR #558, deploying the DEMO monitoring, and validating the synthetic alerts. After this task  I’ll work on the DEV ingestion cost estimate and Terraform preparation.
>
>No blockers from my side.
>
