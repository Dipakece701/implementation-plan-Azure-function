# Openings
>I am working on Azure Functions PROD remediation. I  completed two Ticket #30850 and #30851
>
>**For Task #30850: Restrict inbound access on PROD function apps using caller-driven model:**
>
>I followed the caller-driven security model For all 8 PROD "Function Apps".
>
>The two background-only applications, "helios-prod-cost-ingestion" and "ems-plan-narration-function-prod", using the "Deny-Public-Http" rule. And Both now return **HTTP** 403 to public callers, And the other 6 applications that require public access remain operational.
>
>I also verified that "scm-Ip-Security-Restrictions-Use-Main = false", so the SCM/Kudu deployment endpoints remain independently accessible for Azure DevOps and GitHub Actions without affecting the CI/CD pipelines.
--------------------------------------------
>**For Task #30851: Accept Y1 Consumption tier in PROD and configure AlwaysOn on EP1 plan narration**:
>
>We reviewed the PROD hosting architecture with Maurice.The 7 Dynamic Y1 apps are accepted as by-design, while the EP1 "ems-plan-narration-function-prod" keeps AlwaysOn = false, because the EP1 plan already maintains one pre-warmed instance. Its outbound VNet integration is also active through "appservice-subnet".
>
>Both tasks is completed and closed.
>
>Today, I am going to work on #30846 Migrate PROD Key Vaults from Access Policies to Azure RBAC.
>
>No blockers from my side.
