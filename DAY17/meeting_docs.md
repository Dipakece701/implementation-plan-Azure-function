# OPENINGS
>yestday, I was working on Observability, and Azure function task.
>
>**First worked on Azure function  PROD Gap 1, Ticket AB#30853 Promotion drift, is now  closed.**
>
>Constantin signed off on the narrower ADR and confirmed that the UUDRI workloads belong to Chenlu’s team and are outside our PROD drift scope. And EMS Plan Narration is running live in PROD with both "plan_narration_agent" and "realized_kpi_listener", and the SOP Factory promotion has been safely handed over to Sudhir under AB#28606.
>
>The Weather Eventstream Monitor is also decoupled and tracked separately under Ticket AB#33495.
---------------------------------------------------------------
Second, for the DEMO Observability work, AB#33227
>The branch is rebased cleanly against main, including Maurice’s runner fixes. Both Slack relay Logic Apps have their System-Assigned Managed Identities active, and all CI validation is green with zero destructions across the environments.
>
> I have added the validation results and re-requested review to Soomin.
>Once PR #558 gets approved, the plan is to merge it, apply the dm0 monitoring changes, then apply "dm0 grafana-alerts", and finally trigger synthetic alerts into the DEMO Slack channel to verify the complete flow.

> Sommin added few commentd on PR 558. I'll work on that.
> 
>So my focus is getting a approval on PR #558, deploying the DEMO Observability, and validating the synthetic alerts. 
>No blockers from my side.
>
