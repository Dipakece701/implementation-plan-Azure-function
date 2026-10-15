
## 1. Executive Summary & Major Milestones

```
+---------------------------------------------------------------------------------------------------------------+
|                                      OCTOBER 14 WORKSTREAM OVERVIEW                                           |
+---------------------------------------------------------------------------------------------------------------+
|  [AB#34088] Function Apps & UI Managed Identity Telemetry |  [AB#34089] AKS OTel Sidecar Workload Identity   |
|  - PR #663 Merged into main (Commit a21fec4e)             |  - Spike AB#34092 Completed & Architecture Solved |
|  - Phase A Applied to dm0 (All 5 roles active)            |  - OTel Collector 0.140.0 Upgrade Path Defined    |
|  - Phase B Live Activated (Authorization=AAD)             |  - Webhook Reinvocation Policy Confirmed          |
|  - Log Analytics Verified: 0 HTTP 403s, 100% 200 OK       |  - 22 Sidecar Services / 17 UAMIs Inventoried     |
|  - PR #668 APPROVED & SQUASH-MERGED (Commit 0b570d8e)     |  - Full Implementation Plan Artifact Created      |
+---------------------------------------------------------------------------------------------------------------+
```

1. **User Story AB#34088 — 100% Complete, Approved & Merged:**
   * **PR #663 Merged:** Merged into `main` ([Commit `a21fec4e`])).
   * **Phase A Infrastructure Apply:** Dispatched and applied all 5 sender stacks on `dm0`. Verified all 5 `Monitoring Metrics Publisher` role assignments active on `platform-backend-insights-dm0`.
   * **Phase B Live Activation:** Applied `APPLICATIONINSIGHTS_AUTHENTICATION_STRING = "Authorization=AAD"` via Azure CLI to all 5 senders.
   * **Zero-Drop Live Verification:** Audited Log Analytics workspace `helios-dm0-logs` (`cfaf3a1f-86e9-4cd3-8cc2-0f2e493f5060`). Continuous telemetry flowing with HTTP 200, zero ingestion drops, zero HTTP 403s.
   * **PR #668 Approved & Merged:** Soomin Park formally approved PR #668. PR was **squash-merged into `main`** ([Commit `0b570d8e`]). Local `main` branch synchronized.

2. **User Story AB#34089 — Read-Only Discovery & Spike Completed:**
   * Complete architectural analysis of `apps/otel-collector-sidecar/overlays/demo/opentelemetry-collector.yaml`.
   * Full answers documented for Spike Task **AB#34092**.
   * Comprehensive audit of all 22 AKS services in `argocd/applicationsets-demo/helios-apps-demo.yaml` mapped to 17 UAMIs for **AB#34093**.
   * Created formal implementation plan artifact: **[`demo_aks_otel_workload_identity_plan.md`]**.

---

## 2. Workstream A: AB#34088 Final Delivery & Verification

### Phase A Greenfield Apply Runs on `dm0`
All 5 sender stacks were applied through GitHub Actions CI with exactly **0 resources destroyed**:

| Sender Stack | GHA Apply Run ID | State Result | Role Assignment ID |
|---|---|---|---|
| **`kg-event-processor`** | [Run 38607795532] | `1 added, 1 changed, 0 destroyed` | `19ed845d-1992-8da3-6e63-4fdd92a17bce` |
| **`ui`** | [Run 38607894012] | `1 added, 1 changed, 0 destroyed` | `52fcdc60-bbb2-c682-018b-6beb54ef4567` |
| **`kg-ontology-event-processor`** | [Run 38607902868] | `1 added, 0 changed, 0 destroyed` | `57bf5b5b-cedd-ff5c-38f2-bbea29a0b7bc` |
| **`ems-plan-narration-function`** | [Run 38607911538] | `1 added, 0 changed, 0 destroyed` | `70b38570-afe3-d540-f632-2a28b9bdf865` |
| **`github-activity-logging`** | [Run 38607919669]| `1 added, 1 changed, 0 destroyed` | `ca689748-d67b-58d7-a3d2-5a93831585f8` |

### Phase B Live Activation & Telemetry Health
- Applied `APPLICATIONINSIGHTS_AUTHENTICATION_STRING="Authorization=AAD"` to all 5 applications via Azure CLI.
- Verified active RBAC role assignments on `platform-backend-insights-dm0`:
  - `helios-dm0-kg-event-processor-func` (`d024edb9-f24a-47b5-818e-59fa9d380dd6`) -> `Monitoring Metrics Publisher`
  - `helios-dm0-ui-appservice` (`0628a0e6-3f2f-4f95-a7dc-b8ac8b6c33f7`) -> `Monitoring Metrics Publisher`
  - `helios-ontology-event-processor-func-dm0` (`aa98aae6-a01d-474b-a6d4-77bfa8753542`) -> `Monitoring Metrics Publisher`
  - `helios-ems-plan-narration-func-dm0` (`b0980f4a-14e7-495a-9054-105f998b9513`) -> `Monitoring Metrics Publisher`
  - `helios-github-activity-logging-func-dm0` (`5aee0c67-e0b0-4945-956c-c79a980ed6e3`) -> `Monitoring Metrics Publisher`
- **Kusto Log Analytics Audit:**
  - `helios-dm0-ui-appservice`: > 245,000 requests/dependencies logged in 24h with `ResultCode: 200`.
  - `ems-plan-narration-function-dm0`: Healthy execution events logged with 0 errors.
  - `kg-event-processor-dm0`: Event processing telemetry verified.
  - **Zero HTTP 403 Forbidden errors observed across all senders.**

### Review Approval & Resolution of Soomin Park's Feedback (PR #668)
1. **Approval Verdict:** Soomin Park approved PR #668:
   > *"LGTM - approving. CI is green, and both the RBAC assignments and the live settings check out."*
2. **UI Application-Code Exporter Behavior Clarified:**
   * Soomin confirmed that Helios-UI `back/app/core/telemetry.py` with `azure-monitor-opentelemetry-exporter` (v1.0.0b56) natively falls back to `APPLICATIONINSIGHTS_AUTHENTICATION_STRING` and initializes `ManagedIdentityCredential()`. UI telemetry has been flowing continuously.
3. **PR Description Updated:** Documented the `ignore_changes = [app_settings]` semantics for `kg-ontology-event-processor`, `ems-plan-narration-function`, and `ui`, along with explicit CLI rollback commands.
4. **Squash Merged:** PR #668 merged cleanly into `main` ([Commit `0b570d8e`]).

---

## 3. Workstream B: AB#34089 Spike Findings & Architecture (AB#34092)

### Spike Question 1: Collector Version & Entra ID Exporter Support
* **Finding:** We analyzed the Golang source code of `exporter/azuremonitorexporter` across official tags in `open-telemetry/opentelemetry-collector-contrib`:
  - In **`v0.120.0`** (current sidecar image): `Config` does **NOT** embed `confighttp.ClientConfig` and has no `auth` parameter. Adding `auth.authenticator: azure_auth` fails with a fatal unmarshaling error (`decoding failed: unhandled field auth`).
  - In **`v0.140.0`**: `confighttp.ClientConfig` is squashed into `Config`, and `azuremonitor_exporter.go` initializes its HTTP client using:
    ```go
    telemetryConfiguration.Client, err = exporter.config.ClientConfig.ToClient(ctx, host, exporter.settings)
    ```
    This natively enables `auth.authenticator: azure_auth`, wrapping every outbound batch with the Entra ID bearer token (`Authorization: Bearer <token>`).
  - **Bonus Architectural Cleanup:** Moving to `0.140.0` brings OpenTelemetry semconv v1.34.0, which reads stable HTTP attribute names directly (`http.request.method`, `http.response.status_code`, `url.full`, `url.path`). This allows us to **completely drop the `transform/azmon-compat` processor** that was bridging Java 2.x agent telemetry.
* **Resolution:** Upgrade the central collector image in the CR from `0.120.0` to `0.140.0`.

### Spike Question 2: Webhook Ordering & Token Injection
* **Finding:** Azure Workload Identity's mutating webhook (`mutation.azure-workload-identity.io`) defines `reinvocationPolicy: IfNeeded`.
  - When the OpenTelemetry Operator's mutating webhook injects the `azmon-sidecar` container into a newly scheduled pod, Kubernetes admission control detects that the pod spec was altered and **re-invokes the Azure Workload Identity webhook**.
  - On re-invocation, the Workload Identity webhook injects `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_FEDERATED_TOKEN_FILE`, and mounts the projected service account token volume into the sidecar container.
* **Resolution:** Webhook ordering is guaranteed by Kubernetes admission control without needing per-pod or per-service manifest workarounds.

---

## 4. Full Inventory of DEMO (`dm0`) Sidecar Workloads (AB#34093)

An exhaustive audit of `argocd/applicationsets-demo/helios-apps-demo.yaml` confirmed **22 services** injecting `azmon-sidecar` in `dm0`. These map to **17 unique Azure User-Assigned Managed Identities**:

| # | Service Name | Namespace (`dm0`) | ServiceAccount | Azure Managed Identity Name | Client ID | Principal ID (Object ID) |
|---|--------------|-------------------|----------------|-----------------------------|-----------|--------------------------|
| 1 | `helios-platform-peak-demand-api` | `helios-platform-peak-demand-service-dm0` | `helios-peak-demand-api` | `demand-peak-service` | `bd0d7a91-b7e1-41f0-97d8-9a8037a37da1` | `7ab423d3-ae9e-44ae-905c-a01b091be96c` |
| 2 | `helios-platform-tariff-api` | `helios-platform-tariff-service-dm0` | `helios-tariff-api` | `tariff-service` | `6d568d05-c626-454a-bb0a-b1ab518b5c90` | `23ce37f1-6dcc-46a9-a992-68c610b09b6e` |
| 3 | `helios-platform-weather-connector` | `helios-platform-weather-service-dm0` | `helios-weather-connector` | `helios-weather-connector-dm0-id` | `38d15c46-ae60-4629-9d12-de597fcd4413` | `cb0f341a-d307-4bc0-b08b-068a59d5f085` |
| 4 | `object-store-service` | `object-store-service-dm0` | `object-store-service` | `aks-keyvault-identity` | `486428ea-d40e-459e-be27-5ab1699e3197` | `05fa962b-0081-4e1e-bb4d-8e661f13da38` |
| 5 | `weather-service` | `weather-service-dm0` | `weather-service` | `aks-keyvault-identity` | `486428ea-d40e-459e-be27-5ab1699e3197` | `05fa962b-0081-4e1e-bb4d-8e661f13da38` |
| 6 | `iam-service` | `iam-system-dm0` | `iam-service` | `aks-keyvault-identity` | `486428ea-d40e-459e-be27-5ab1699e3197` | `05fa962b-0081-4e1e-bb4d-8e661f13da38` |
| 7 | `digital-twin-api` | `digital-twin-dm0` | `digital-twin-api-sa` | `helios-dm0-fabric-aks-workload-id` | `79ad4b64-fbc5-4f46-8fe5-2d99388ef6bd` | `423634b5-c313-4ab2-bed2-708ef4c3327b` |
| 8 | `helios-platform-oms` | `helios-platform-oms-dm0` | `helios-platform-oms-sa` | `helios-dm0-fabric-aks-workload-id` | `79ad4b64-fbc5-4f46-8fe5-2d99388ef6bd` | `423634b5-c313-4ab2-bed2-708ef4c3327b` |
| 9 | `helios-platform-monetization-api` | `helios-platform-monetization-service-dm0` | `helios-monetization-api` | `helios-dm0-fabric-aks-workload-id` | `79ad4b64-fbc5-4f46-8fe5-2d99388ef6bd` | `423634b5-c313-4ab2-bed2-708ef4c3327b` |
| 10 | `helios-aem-api-service` | `helios-aem-api-service-dm0` | `helios-aem-api-service` | `helios-alarm-api-identity` | `fb12dfcb-e7db-404a-b244-30aa2e892b15` | `3c48d107-82d4-470d-a216-af79b25938ea` |
| 11 | `helios-aem-consumer-service` | `helios-aem-consumer-service-dm0` | `helios-aem-consumer-service` | `helios-alarm-otel-consumer-identity` | `791276ea-b164-4bc2-b319-7f68d81dd4fe` | `325163f1-8ad0-431a-964f-a13e0a999b78` |
| 12 | `helios-platform-hitl-worker` | `helios-platform-hitl-service-dm0` | `helios-hitl-worker` | `hitl-worker` | `4b6c86a9-5199-451d-86fd-2f2cf0f0fe4f` | `02927e53-861c-4858-b60b-4dfae146afed` |
| 13 | `helios-platform-hitl-service` | `helios-platform-hitl-service-dm0` | `helios-hitl-api` | `hitl-worker` | `4b6c86a9-5199-451d-86fd-2f2cf0f0fe4f` | `02927e53-861c-4858-b60b-4dfae146afed` |
| 14 | `helios-platform-alarm-processor` | `helios-platform-alarm-processor-dm0` | `helios-alarm-processor` | `helios-alarm-processor` | `29ca810a-61d0-426e-9d1c-1efe4095b21f` | `e4b8dbd0-f5d6-46c6-9e97-f6d012962ff2` |
| 15 | `helios-platform-weather-api` | `helios-platform-weather-service-dm0` | `helios-weather-api` | `weather-service` | `370785b3-742c-4388-a08b-5611ede09496` | `04928fbf-2418-44f1-b09d-31db9b99ff3d` |
| 16 | `helios-work-order-api` | `helios-work-order-api-dm0` | `helios-work-order-api` | `helios-work-order-api-dm0-id` | `251bac71-30ec-4fbf-b06f-d62b58f47179` | `ba8b29ad-0763-452f-9d39-723a95f5c63e` |
| 17 | `helios-analytics-soe-kpi-worker` | `helios-soe-kpi-worker-dm0` | `helios-soe-kpi-worker` | `helios-soe-kpi-worker` | `d61b69e8-00b1-447f-939c-74767dfec7f7` | `93c3db8c-d28a-4a57-bed7-ac4a4828babd` |
| 18 | `helios-analytics-soe-trigger-worker` | `site-optimization-worker-dm0` | `helios-soe-trigger-worker` | `helios-soe-trigger-worker` | `5bb70f75-a732-4f06-b6d7-e4a070c43a88` | `ed33b98b-66eb-4572-a87b-bc6e0d2e234c` |
| 19 | `helios-platform-market-api` | `helios-platform-market-service-dm0` | `helios-market-api` | `helios-market-api` | `562a1178-4627-4ed6-bfea-19f9bb2d6913` | `c3ca7760-638f-4d01-af48-50b402c98b5e` |
| 20 | `helios-platform-repo-manager-service` | `helios-platform-repo-manager-service-dm0` | `helios-repo-manager-api` | `repo-manager-service` | `ac7ea66e-12f4-401f-a21c-9ee33c39e692` | `20342601-5145-40f7-9afb-1b22960d8d6e` |
| 21 | `helios-platform-notification-api` | `helios-platform-notification-service-dm0` | `helios-notification-api` | `notification-service` | `3740133f-1fb3-4140-b226-727609176650` | `39ff913f-1be6-4c46-a6e8-b6db83cfcf05` |
| 22 | `helios-platform-telemetry-api` | `helios-platform-telemetry-service-dm0` | `helios-telemetry-api` | `telemetry-service` | `f5f84bbe-d6a0-4114-90bd-fb0562d9e996` | `8374cc3d-edba-469a-b371-de4e79f6300e` |

---

## 5. 🗣️ Key Topics & Questions for Team Discussion

### Topic 1: Terraform Role Assignment Placement (AB#34093)
Where should the `Monitoring Metrics Publisher` role assignments for the 17 workload identities be declared in `helios-infra`?

* **Option A (Recommended by SRE): In `terraform/stacks/aks/main.tf`**
  * **Pros:** All 17 UAMIs are created and managed directly inside `terraform/stacks/aks`. Placing the role assignments here keeps identity lifecycle and permissions co-located. It matches the pattern already used in `aks/main.tf` line 621 for `aks_monitoring_metrics_publisher`.
  * **Cons:** The `aks` stack is large; planning takes ~3–5 minutes.
* **Option B: In `terraform/stacks/app_insights/main.tf`**
  * **Pros:** Concentrates all role assignments on `platform-backend-insights` within the App Insights stack itself. Fast plan/apply cycle.
  * **Cons:** Requires either passing all 17 identity IDs into `app_insights` or querying them via `data "azurerm_user_assigned_identity"`, creating cross-stack coupling.

### Topic 2: CR Change Rollout Strategy (AB#34094)
Should the collector image bump (`0.120.0` -> `0.140.0`) and the `azure_auth` activation be deployed together or phased?
* **SRE Proposal:** Deploy both changes together in one single PR to `apps/otel-collector-sidecar/overlays/demo/opentelemetry-collector.yaml`. The 0.140.0 collector supports both connection-string routing and `azure_auth` token generation concurrently, preventing double pod restarts.

### Topic 3: Canary Service Selection
Which service should be the primary canary to verify the new CR before restarting the full fleet?
* **SRE Proposal:** **`helios-platform-tariff-api`** (`helios-platform-tariff-service-dm0`).
  * Has 2 replicas, steady active traffic (HTTP requests/dependencies), standalone architecture, and no upstream blockers.
  * We will restart `tariff-api` first, verify its sidecar logs for clean Entra token acquisition, and confirm Log Analytics continues reporting HTTP 200 without 403s.

### Topic 4: Secret Deprecation & Local Auth Gate (AB#34090)
* Once the sidecar authenticates with Workload Identity, the `otel-azmon-connection-string` secret is no longer used for authentication.
* Gate rule: local auth will **not** be disabled on `platform-backend-insights-dm0` until all 22 AKS services and all 5 non-AKS senders show sustained green telemetry under Entra auth for a minimum of 24 hours.

---

## 6. Immediate Execution Queue

1. **PR #668 Merged & Synced:** PR #668 is squash-merged into `main` ([Commit `0b570d8e`]); local workspace updated. Phase B code parity for AB#34088 is complete.
2. **Team Decision on Placement:** Confirm Option A (`aks` stack) or Option B (`app_insights` stack) during today's standup.
3. **Dispatch AB#34093 PR (`helios-infra`):** Add `azurerm_role_assignment` for the 17 workload identities and execute Terraform apply on `dm0`.
4. **Dispatch AB#34094 PR (`helios-gitops`):** Update `azmon-sidecar` CR to collector `0.140.0` with `azure_auth` extension and drop `transform/azmon-compat`.
5. **Canary & Fleet Restart:** Roll restart on `tariff-api`, audit telemetry, then roll restart remaining 21 workloads.
