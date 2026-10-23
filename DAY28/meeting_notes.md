# OPENINGS
> Yesterday, I worked on Ticket AB#34970, I finalized the SRE tabletop exercise materials and incorporated Constantin’s feedback.
>
> The session has been rescheduled to Monday, October 26, at 10:30 AM Pacific time.
>
> I updated the incident procedure, scenario brief, live-notes template, and ticket templates.
>
> For the reliability scenario, I switched the failing service to oms-api because it sits behind the internal APIM gateway. For the security scenario, I updated the Key Vault secret reference and documented the detection gap.
>
>The scenario starts with reviewing the audit logs for blocked HTTP 403 attempts and follows our incident severity and response procedure.
>
>For AB#34093 and PR #703, Soomin reviewed the proposed AKS Workload Identity role assignments.
>
> The 17 DEMO assignments are correct, but the current implementation needs changes to remain compatible with DEV, QA, and PROD.
>
>  I’ll update the Terraform implementation to use azapi_resource_list, dynamically identify AKS-federated managed identities, and enable the feature flag only for DEMO.
>
> My next step is to run Terraform plans across all four environments. The expected result is 46 role assignments in DEMO, with zero changes and no errors in DEV, QA, and PROD. I’ll post the plan results for Soomin’s review.
>
> No blocker from my side.
