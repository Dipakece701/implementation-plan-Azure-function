# OPENINGS
>Hi everyone, I am working on the Azure Function QA SRE remediation.
>
>I completed and verified Task #30167, Gap 6 — Inbound Access Restrictions.
>
>clarifying the handling of UUDRI-Function-App-qa-01 and documenting the exact SCM property required to keep CI/CD deployments working.
>
>I followed a staged rollout. First I piloted the "DenyPublicHttp" rule on "helios-qa-cost-ingestion" and verified that public requests return "**HTTP"** 403 while the SCM deployment endpoint remains accessible.
>
>Then I rolled the same restriction out to the remaining four background or idle apps, including the "UUDRI" applications.
>
>After that, I ran positive probes across all 10 QA Function Apps. The 5 background or idle apps now return HTTP 403, while the 5 legitimate "public-facing" apps continue returning HTTP 200.
>
>For CI/CD, I verified "scm-Ip-Security-Restrictions-Use-Main = false", so the SCM/Kudu endpoint maintains its separate access restriction rules and deployments are not blocked.
>
>So Task #30167 is fully implemented and verified live and closed in ADO.
>
>No blockers from my side.
