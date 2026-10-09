# OPENINGS
>Friday, I worked on observability task.
>
> Ticket AB#33227, is now successfully deployed and verified. And PR #558 was merged into main,and I also resolved an initial Terraform apply failure by identifying a deleted Entra ID user in the Grafana admin role assignment.I worked with Soomin on PR #650 to remove that stale user while preserving the Helios-DevOps-Team group. PR #650 was approved and merged successfully.
>
>After that, both the dm0 monitoring and dm0 grafana-alerts Terraform applies completed successfully. And I also verified the three required reader role assignments on the Grafana managed identity, resolving the "Insufficient-Access-To-Resource" dashboard errors.
>
> Then I triggered a synthetic alert through "ag-helios-dm0-ops" and confirmed that it was successfully delivered to "#helios-demo-alerts" through the Slack relay Logic App.
>
>And I also accepted daily ownership of #helios-demo-alerts for alert triage and noise management.
>
>Next, I am going to start work on Ticket  AB#33436, which applies the same Application Insights telemetry fix we used in PROD to the DEMO Function Apps.
>
>No blockers from my side.

