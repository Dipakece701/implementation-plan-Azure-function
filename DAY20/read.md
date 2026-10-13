## 1. Executive Summary & Major Milestones

1. **Strategic Pivot: Application Insights Consolidation ([AB#34086]):**
   * **Replaced Parallel-Instance Model:** Closed [PR #653] / AB#33436 (which attempted to duplicate the PROD #595 secondary public App Insights pattern).
   * **Consolidated Architecture:** Standardized on `platform-backend-insights-dm0` as the single unified telemetry sink for all AKS, App Service, and non-VNet Function App workloads.
2. **PR #655 Delivered, Approved & Merged ([AB#34087]):**
   * **Branch & Commit:** `feat/ab-34087-open-dm0-platform-backend-insights` merged into `main` as [`9f14fa44`].
   * **Scope:** Added `dm0/app_insights.tfvars` with `internet_ingestion_enabled = true`. Parameterized module and stack with `default = false`.
   * **Security Boundary Preserved:** Reverted AMPLS `ingestion_access_mode` to fixed `PrivateOnly` per Soomin's guidance to maintain exfiltration boundaries for VNet-integrated workloads.
   * **Approval:** Approved by **Soomin Park** (`state: APPROVED`, `LGTM`).
3. **Terraform Apply Executed Successfully in `dm0` (100% Green):**
   * **Workflow Run:** [Run 38317217693], Job [`115036643843`]).
   * **Result:** `Apply complete! Resources: 0 added, 1 changed, 0 destroyed.`
   * In-place modification completed in 3 seconds: `internet_ingestion_enabled: true` live in Azure subscription `bb47b7ba-e899-4c87-be6e-2f36dc1165c7`.
4. **DEV Non-AKS Estate Observability Alignment (Wiki 2386):**
   * Reviewed DEV estate remediation plan with Constantin Pricochi.
   * Scoped immediate scope down to **GAP-003** (diagnostic settings) and **GAP-010** (App Insights connection) in pure Terraform.
   * Committed to producing ingestion volume (GB/day) and cost estimates prior to any rollout.

---

## 2. Detailed Workstream Status & Operational Highlights

```
+---------------------------------------------------------------------------------------------------------------+
|                                      OCTOBER 12 WORKSTREAM OVERVIEW                                           |
+---------------------------------------------------------------------------------------------------------------+
|  [AB#34086 / AB#34087] App Insights Consolidation     |  [Wiki 2386] DEV Non-AKS Estate Implementation        |
|  - PR #653 closed as superseded                       |  - Revision 5 published to ADO Wiki (37 resources)    |
|  - PR #655 created, approved, and merged to main      |  - Constantin alignment: Waves 2–5 paused for team     |
|  - AMPLS access mode locked at PrivateOnly            |  - Approved to start GAP-003 & GAP-010 in Terraform   |
|  - GitHub Actions apply: Run 38317217693 (SUCCESS)   |  - Cost & ingestion model required before rollout      |
|  - platform-backend-insights-dm0 accepts public logs  |                                                       |
|-------------------------------------------------------+-------------------------------------------------------|
|  Upcoming Execution Queue (Feature AB#34086 Roadmap)                                                          |
|  1. AB#34087 Verification: Query helios-dm0-logs for rows from 4 Function Apps                                |
|  2. AB#34088: Wire 4 Function Apps & UI App Service with Entra ID Auth (Monitoring Metrics Publisher)        |
|  3. Cleanup PR: Drop unused ampls_dns_ips & ampls_blob_ip variables across all envs                           |
|  4. AB#34089: Migrate AKS azmon-sidecar OTel collector to Workload Identity                                   |
+---------------------------------------------------------------------------------------------------------------+
```

### Workstream A: DEMO App Insights Consolidation — [AB#34086] / [AB#34087]

#### Context & Strategic Alignment
* Initial plan was to replicate PROD PR #595 by creating `platform-backend-public-insights-dm0` via PR #653 (AB#33436).
* Soomin Park requested consolidating all workloads into a single instance (`platform-backend-insights-dm0`) under Feature AB#34086 to eliminate split telemetry sinks.
* Formally closed PR #653 with full traceability.

#### Implementation & Review Iteration (PR #655)
* **Code Changes:**
  * `terraform/modules/app_insights/main.tf` & `variables.tf`: Added `internet_ingestion_enabled` (default `false`).
  * `terraform/stacks/app_insights/main.tf` & `variables.tf`: Threaded variable to module.
  * `terraform/environments/dm0/app_insights.tfvars`: Landed with `internet_ingestion_enabled = true`.
* **Security & Architectural Nuance (AMPLS vs. Component):**
  * *Soomin’s Feedback:* Revert AMPLS `ingestion_access_mode` from dynamic back to fixed `PrivateOnly`.
  * *Rationale:* AMPLS access mode governs clients *inside* connected VNets (preventing exfiltration to outside sinks). The 4 non-VNet Function Apps (`virtualNetworkSubnetId: null`) do not route through AMPLS; component-level `internet_ingestion_enabled = true` unblocks them directly without compromising the internal VNet perimeter.
  * Fixed in commit `1a72c1c9`.
* **Plan & Apply Verification:**
  * **Plan on Branch:** [Run 38309045297] — Exactly `Plan: 0 to add, 1 to change, 0 to destroy`.
  * **PR Merge:** Approved by Soomin (`LGTM`), merged into `main` as [`9f14fa44`].
  * **Live Apply:** [Run 38317217693] — Completed successfully in 3 seconds. Zero drift across all existing state resources.

#### Target Function Apps Unblocked
1. `kg-event-processor-dm0`
2. `helios-ontology-event-processor-func-dm0`
3. `helios-github-activity-logger-dm0-func`
4. `ems-plan-narration-function-dm0`

---

### Workstream B: DEV Estate Non-AKS SRE Alignment (Wiki 2386)

* **Scope Audit:** Revision 5 documents 37 compute/managed resources across 10 resource groups in DEV subscription `a6498579-cfb7-41e9-a957-14375196a386`.
* **Guidance from Constantin Pricochi:**
  * Priority remains DEMO and PROD first; DEV comes after.
  * Waves 2–5 (Container probes, autoscaling, RBAC migration, VNet boundaries) on hold pending team discussion.
  * Approved starting **GAP-003** (Diagnostic settings on compute resources) and **GAP-010** (App Insights connection) strictly in Terraform.
  * **Requirement:** Produce a daily ingestion volume (GB/day) and cost estimate prior to rollout to ensure alignment with our current log trimming initiative (~40 GB/day cut).

---

## 3. Key Technical Decisions & Architecture Reference

| Area | Decision / Architecture | Rationale |
| :--- | :--- | :--- |
| **Telemetry Component** | Consolidated on `platform-backend-insights-dm0` | Avoids fragmenting telemetry across multiple App Insights components. |
| **Ingestion Security** | `internet_ingestion_enabled = true`, AMPLS `PrivateOnly` | Allows non-VNet Function Apps to push logs publicly while keeping VNet internal traffic strictly bounded. |
| **Blast Radius** | Module default `internet_ingestion_enabled = false` | Ensures DEV, QA, and PROD experience zero drift / zero plan changes. |
| **Auth Transition** | Step 1: Public ingestion enabled; Step 2: Entra ID auth | Allows unblocking Function Apps immediately before completing Managed Identity role assignments. |
| **Variable Cleanup** | Defer `ampls_dns_ips` / `ampls_blob_ip` removal to follow-up PR | Prevents invalidating approvals on active deployment PRs; cleans unused variables globally. |

---

## 4. Risks & Mitigations

* **Local Auth Surface on Public Component:**
  * *Observation:* Local auth (connection strings / instrumentation keys) remains active on `platform-backend-insights-dm0` while public ingestion is open.
  * *Risk:* Exposure of ingestion endpoint if keys are leaked.
  * *Mitigation:* Daily cap verified on `helios-dm0-logs`. Moving immediately to **AB#34088** (Entra ID Managed Identity ingestion) and **AB#34090** (disable local authentication).
* **DEV Log Ingestion Influx:**
  * *Risk:* Adding diagnostic settings across 27 compute resources could inflate daily ingestion costs.
  * *Mitigation:* Modeling log generation rates (GB/day) and presenting cost projection to Constantin prior to opening PRs.

---

## 5. Next Immediate Action Items

1. [x] Pivot architecture from PR #653 to consolidation Feature AB#34086.
2. [x] Create and refine PR #655 (AB#34087) with `internet_ingestion_enabled = true` and AMPLS `PrivateOnly`.
3. [x] Secure approval from Soomin Park on PR #655.
4. [x] Merge PR #655 into `main` ([Commit 9f14fa44]).
5. [x] Apply `dm0 app_insights` via GitHub Actions ) — 100% green.
6. [ ] **Verify Ingestion Rows ([AB#34087]):** Query `helios-dm0-logs` for `AppRequests` and `AppTraces` across all 4 Function Apps.
7. [ ] **Follow-up Cleanup PR:** Remove unused `ampls_dns_ips` and `ampls_blob_ip` variables from `app_insights` module, stack, and tfvars.
8. [ ] **Kick off [AB#34088]:** Implement Entra ID authentication for the 4 Function Apps and UI App Service (`Monitoring Metrics Publisher` role + `APPLICATIONINSIGHTS_AUTHENTICATION_STRING = "Authorization=AAD"`).
9. [ ] **DEV GAP-003 / GAP-010 Cost Modeling:** Calculate expected daily GB/day ingestion and cost estimate for Constantin's sign-off.
