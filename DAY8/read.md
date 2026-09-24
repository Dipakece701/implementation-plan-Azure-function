# FOR READ
---

## 2. Before vs. After Implementation Scorecard

| SRE / Security Dimension | Baseline Before Work | Current State (After Implementation) | Verification Evidence |
|:---|:---:|:---:|:---|
| **Cost Ingestion Telemetry (Task #30169)** | Disconnected (`[]` — 0 telemetry, silent failure risk) | **Active Telemetry Streaming** (`requests`, `traces`, `exceptions`) | `az functionapp config appsettings` confirms connection string bound to `platform-backend-insights-qa`. App is `Running`. |
| **QA Key Vault RBAC Posture (Task #30171)** | 8 RBAC / 3 Legacy Access Policies (72% Coverage) | **11 of 11 Vaults on Azure RBAC (100% Coverage)** | Subscription sweep confirms `enableRbacAuthorization: true` across all 11 vaults in QA. |
| **QA UI App Service Security** | Legacy Access Policy (`Get`, `List`) | **Modern Azure RBAC (`Key Vault Secrets User`)** | Pre-assigned prior to cutover; UI frontend probed live returning **HTTP 200 OK** with zero downtime. |
| **Terraform Deployment Pipeline** | Legacy Access Policies on UI & PKI vaults | **Azure RBAC (`Key Vault Administrator` & `Secrets User`)** | Least privilege enforced: Full admin on UI vault, Read-only secret access on Sparkplug PKI vault. |
| **UUDRI AI Foundry Security** | Access Policy on `UUDRI-Key-Vault-qa-02` | **Azure RBAC (`Key Vault Administrator`)** | Native Managed Identity roles active; API requests succeeding without authorization drops. |

---

## 3. Technical Implementation Details

### A. Task #30169: Application Insights Wiring on `helios-qa-cost-ingestion`
* **Target Resource:** `helios-qa-cost-ingestion` (Linux Y1 Consumption in `helios-qa-us-west3-rg`)
* **Function Workload:** `cost_ingestion` (Daily Timer Trigger @ 06:00 UTC)
* **Target Telemetry Sink:** `platform-backend-insights-qa` (AppId: `62f240b5-eefc-42cc-b423-867d3dd5abea`)
* **App Setting Applied:**
  ```text
  APPLICATIONINSIGHTS_CONNECTION_STRING = InstrumentationKey=78d74f48-45fd-4481-8077-ce73f9d4686b;IngestionEndpoint=https://westus3-1.in.applicationinsights.azure.com/;LiveEndpoint=https://westus3.livediagnostics.monitor.azure.com/;ApplicationId=62f240b5-eefc-42cc-b423-867d3dd5abea
  ```
* **Runtime Verification:** `State == "Running"`, `UsageState == "Normal"`.

---

### B. Task #30171: 100% Azure RBAC Estate Coverage (All 11 QA Vaults)

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                      QA KEY VAULT ESTATE: 100% AZURE RBAC MIGRATION                    │
├────┬─────────────────────────────┬────────────────────────┬──────────────┬─────────────┤
│ #  │ Key Vault Name              │ Resource Group         │ RBAC Status  │ Cutover By  │
├────┼─────────────────────────────┼────────────────────────┼──────────────┼─────────────┤
│ 1  │ UUDRI-Key-Vault-qa-02       │ uudri-qa-rg            │ True ✅      │ Dipak Singh │
│ 2  │ helios-qa-ui-kv             │ helios-qa-uswest3-ui   │ True ✅      │ Maurice K.  │
│ 3  │ helios-qa-spkplug-pki-kv    │ helios-qa-us-west3-rg  │ True ✅      │ Maurice K.  │
│ 4  │ helios-qa-backend-kv        │ helios-qa-us-west3-rg  │ True ✅      │ Baseline    │
│ 5  │ helios-qa-agents-kv         │ helios-qa-us-west3-rg  │ True ✅      │ Baseline    │
│ 6  │ helios-qa-onboarding-kv     │ helios-qa-us-west3-rg  │ True ✅      │ Baseline    │
│ 7  │ kg-event-processor-qa-kv    │ helios-qa-us-west3-rg  │ True ✅      │ Baseline    │
│ 8  │ kvsopfactoryqao80ns         │ helios-qa-us-west3-rg  │ True ✅      │ Baseline    │
│ 9  │ kvsardemoqaasfus            │ helios-qa-us-west3-rg  │ True ✅      │ Baseline    │
│ 10 │ helios-qa-coverage-kv       │ helios-qa-us-west3-rg  │ True ✅      │ Baseline    │
│ 11 │ helios-qa-sbom-sign-kv      │ helios-qa-us-west3-rg  │ True ✅      │ Baseline    │
└────┴─────────────────────────────┴────────────────────────┴──────────────┴─────────────┘
```

#### Pre-Cutover Role Pre-Assignments Verified:
1. `helios-qa-ui-appservice` ➔ **`Key Vault Secrets User`** on `helios-qa-ui-kv` (Least Privilege).
2. `helios-qa-terraform-sp` ➔ **`Key Vault Administrator`** on `helios-qa-ui-kv` (Matches CRUD access).
3. `helios-qa-terraform-sp` ➔ **`Key Vault Secrets User`** on `helios-qa-spkplug-pki-kv` (Least Privilege).
4. `UUDRI-AI-Foundry-qa-01` ➔ **`Key Vault Administrator`** on `UUDRI-Key-Vault-qa-02`.

---

## 4. Q&A Cheat Sheet (Handling Team Questions)

### Q1: "Did wiring App Insights on cost-ingestion require modifying application code?"
* **Answer:**  
  *No code changes were required.* Azure Functions natively instruments runtime invocations, execution duration, and Python exceptions once `APPLICATIONINSIGHTS_CONNECTION_STRING` is set in the platform configuration.

### Q2: "How did we guarantee that switching to RBAC wouldn't take down the QA UI web application?"
* **Answer:**  
  In Azure Key Vault, toggling `enableRbacAuthorization = true` instantly disables legacy Access Policies. To guarantee zero downtime, Maurice pre-assigned the `Key Vault Secrets User` role to `helios-qa-ui-appservice` *before* the cutover. When the switch was flipped, the UI app was already authorized under RBAC, resulting in uninterrupted secret retrieval.

### Q3: "Why did we assign `Key Vault Secrets User` instead of `Key Vault Administrator` to Terraform on the PKI vault?"
* **Answer:**  
  Maurice rightly noticed that on `helios-qa-spkplug-pki-kv`, Terraform only had `secrets: [get, list]` under its legacy policy. Giving Terraform `Key Vault Administrator` would have granted excessive create/purge permissions. In accordance with the Principle of Least Privilege, we assigned `Key Vault Secrets User` to match its exact operational scope.

---

## 5. QA SRE Remediation Progress Tracker (Parent #30165)

| Task ID | SRE Gap Description | Status | Current Milestone Delivery |
|:---:|:---|:---:|:---|
| **#30174** | Gap 5: Availability Web Tests (`ontology` & `kg`) | **Closed** ✅ | Completed by Maurice (Terraform PR #518). |
| **#30166** | Gap 4: Centralized Diagnostic Settings (10/10 Apps) | **Closed** ✅ | Completed by Dipak (10/10 streaming to `helios-qa-logs`). |
| **#30168** | Gap 8: SOP Factory Durable Orchestrator Alert | **Closed** ✅ | Completed by Dipak (`alert-sopfactory-orchestrator-failure`). |
| **#30169** | Gap 3: Wire App Insights on `cost-ingestion` | **Closed** ✅ | **Completed & Verified by Dipak (Live in Azure).** |
| **#30171** | Gap 1: Key Vault RBAC Migration (11/11 Vaults) | **Closed** ✅ | **Completed & Verified by Dipak & Maurice (100% RBAC).** |
| **#30167** | Gap 6: Inbound Access Restrictions (5 Background Apps) | **In Progress** 🟡 | Ticket updated (Rev 3). Plan approved, ready for pilot. |
| **#30172** | Gap 2: Enable System Managed Identity on UUDRI | **New** 🟡 | Ready for execution. |
| **#30170** | Gap 7: Accept Y1 Consumption Tier as By-Design | **New** 🟡 | Architectural acceptance item. |

**Progress:** **5 out of 8 tasks (62.5%)** are now completed and verified live in QA.
