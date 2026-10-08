# Openings
>yesterday, I worked on observability task.
>First, I resolved the merge conflict with main in prod-plan-contract.json. I pulled in the latest changes, including PRs #614, #629, and #612, and re-baselined the monitoring contract by recalculating the SHA-256 hashes across all 19 required contract files.
>
>And  I also completed the requested Terraform formatting and cleanup changes and updated the PR documentation for the existing "DM0_GRAFANA_API_KEY secret".
>
>Second, I validated all five Terraform plans — dm0 monitoring, dm0 grafana-alerts, dev, qa, and prod. All five passed successfully with zero destructions, and the latest results are published in PR comment #6069328753.
>
>The current status is waiting for Soomin’s approval. Once the approval comes through, the deployment sequence is to merge PR #558, apply "dm0 monitoring", then "dm0 grafana-alerts", and finally send a synthetic test alert through "ag-helios-dm0-ops" to verify it reaches the DEMO Slack channel.
>
>I also have AB#33436 queued next for the DEMO Function App telemetry issue, which follows the same approach we used for the PROD telemetry fix.
>
>No blockers from my side
