# Meeting Notes: Foundry Models Implementation Plan Review & Multi-Env Alignment



**Master Implementation Files Produced:**
* DEV: [`reports/implementation-plan/DEV-Env-Implementation-plan-Foundry-Models.md`]
* QA: [`reports/implementation-plan/QA-Env-Implementation-plan-Foundry-Models.md`]
* PROD: [`reports/implementation-plan/PROD-Env-Implementation-plan-Foundry-Models.md`]

---

[[_TOC_]]

---

## 1. Executive Summary

Following senior architectural review by Principal SRE Reviewer Sam Chai, all three environment implementation plans (DEV, QA, and PROD) have achieved **Conditional Approval** across all major operational, security, and reliability dimensions.

Every technical critique and newly discovered architecture nuance has been investigated, verified against live Azure state, and reconciled:
1. **Terraform Schemas Standardized:** Corrected flat deployment fields to native nested `model = { format, name, version }` and `scale = { type, capacity }` objects matching `modules/ai_foundry/main.tf`, and replaced ARM/KEDA JSON probe syntax with native `azurerm_container_app` provider blocks (`readiness_probe {}`, `liveness_probe {}`, `http_scale_rule {}`).
2. **Caller Dependency Uncovered in QA & PROD:** Identified that the primary caller of the model-serving Container App in both QA and PROD is an Azure Function App orchestrator running on a Consumption plan without VNet integration. Restricting Container App ingress to currently observed outbound IPs would trigger silent outages during Azure IP rotation. We established VNet integration for the orchestrator as a mandatory prerequisite to ingress internalization.
3. **PROD AI Hub Consumer Decoupled:** Resolved a critical architectural misattribution: `ca-model-service-prod` does **not** consume `helios-prod-aif-hub` (it has its own dedicated account `ais-sopfactory-prod` with `gpt-5` at 100k TPM). The real consumer of the Hub is `ems-plan-narration-function-prod` via `pms-core-project`, around which quota planning is now properly scoped.
4. **Director Tony Guid POC Reconciled:** Verified 1.1M TPM on `tonyguid-test-resource` in East US 2; established an executive consultation workflow to consolidate duplicate allocations and reclaim 690,000 TPM without disrupting executive testing.
---

## 3. Presenter Quick-Reference Points (Key Highlights)

* **All 3 Plans Reviewed by Sam Chai:** DEV (Page 2642), QA (Page 2656), and PROD (Page 2658) have achieved conditional approval.
* **HCL Provider Schemas Fixed Across All Envs:** Replaced flat fields with nested `model = { format, name, version }` and `scale = { type, capacity }` objects matching `modules/ai_foundry/main.tf`. Replaced ARM JSON with native `azurerm_container_app` blocks (`readiness_probe {}`, `liveness_probe {}`, `http_scale_rule {}`).
* **Critical Caller Dependency (QA & PROD):** Consumer orchestrator function apps (`func-orchestrator-sopfactory*`) run on Consumption plans without VNet integration across 25 dynamic outbound IPs. Restricting ingress to currently active IPs causes silent outages during Azure IP reassignment. Mandated VNet integration as a prerequisite to ingress internalization.
* **PROD Hub Consumer Decoupled:** `ca-model-service-prod` uses `ais-sopfactory-prod` (which already has `gpt-5` at 100k TPM). The actual consumer of `helios-prod-aif-hub` is `ems-plan-narration-function-prod` via `pms-core-project`.
* **Director Tony Guid 1.1M TPM Allocation:** Confirmed 930k TPM across 5x `gpt-5.4-pro` in East US 2. Reframed as an Executive POC consultation to consolidate duplicate instances and reclaim 690k TPM.
* **PROD Reliability (P0):** Bumping `ca-model-service-prod` baseline from 1 to 2 replicas (`minReplicas = 2`) to eliminate single-replica failure risk.

---

## 4. Environment-by-Environment Detailed Breakdown

### 4.1 DEV Environment (Wiki Page 2642)
* **Status:** Conditional Approval (Comments #10416224, #10425018).
* **Approved Items:**
  - GAP-FM-002: 3-phase rollout (`Caller Token Audit` $\to$ `Canary Revision` $\to$ `Auth Enforcement & Ingress Restriction`).
  - GAP-FM-005: Disambiguation of `ais-sopfactory-dev` vs `ais-sopfactorydevmlel9` and workspace split (`helios-dev-logs` for Hub, `log-sopfactory-dev` for SOP Factory).
  - GAP-FM-006: KQL Scheduled Query Rules for 429 throttling, 5xx server errors, and P95/P99 latency breaches.
  - GAP-FM-007: 1.1M TPM capacity math on `tonyguid-test-resource` and executive right-sizing workflow.
* **Blockers Resolved in Revision 2.2:**
  - Replaced flat HCL fields with nested `model` and `scale` objects matching `modules/ai_foundry/main.tf`.
  - Replaced ARM/KEDA probe schema with native `azurerm_container_app` provider blocks.
  - Attached raw Azure CLI outputs confirming GitHub Actions OIDC federation (`qcells-hqct/sop-factory`), Service Principal `sp-sop-factory-terraform-dev` (`210ef71d-6bbb-44bb-8004-f27b7fb47ee8`), and workspace `log-sopfactory-dev`.

---

### 4.2 QA Environment (Wiki Page 2656)
* **Status:** Conditional Approval (Comment #10427372).
* **Approved Items:**
  - GAP-FM-005: `ais-sopfactory-qa` / `ais-sopfactoryqao80ns` identity and workspace routing (`log-sopfactory-qa` vs `helios-qa-logs`).
  - GAP-FM-006: Production-grade KQL Scheduled Query Rules linked to `ag-helios-qa-ops`.
  - GAP-FM-001: Model deployment inventory (9 models) and alias consolidation (resolving mislabeled `gpt-4.1` pointing to `gpt-4.1-mini`). Raw SKU capacities confirmed.
* **Blockers & Architectural Discoveries Resolved:**
  - **Caller Dependency on GAP-FM-002:** The consumer of `ca-model-service-qa` is `func-orchestrator-sopfactoryqao80ns` on a Consumption plan with no VNet integration. Currently using 7 of 25 possible outbound IPs. Allowlisted all 25 IPs in the interim, and sequenced VNet integration as a prerequisite to ingress internalization.
  - Standardized HCL on nested `model`/`scale` objects and native `azurerm_container_app` probe/scaling syntax.

---

### 4.3 PROD Environment (Wiki Page 2658)
* **Status:** Conditional Approval (Comment #10427903).
* **Approved Items:**
  - GAP-FM-004: P0 single-replica outage risk on `ca-model-service-prod` confirmed (`minReplicas = 1`, `scale.rules = null`). Approved for `minReplicas = 2` + KEDA HTTP concurrency scaling.
  - GAP-FM-003: P0 missing probes confirmed (`probes: []`). Approved for native HTTP readiness/liveness probes.
  - GAP-FM-002: Public ingress on port 8080 approved for phased restriction under AB#27339.
  - GAP-FM-005: 75% diagnostic blind spot confirmed; workspace `log-sopfactory-prod` verified.
  - GAP-FM-006: Scheduled Query Rules against `AzureDiagnostics` linked to `ag-helios-prod-ops` approved.
* **Blockers & Architectural Discoveries Resolved:**
  - **GAP-FM-008 Consumer Decoupling:** Live RBAC and endpoint inspection revealed `ca-model-service-prod` connects to `ais-sopfactory-prod` (which already has `gpt-5` at 100,000 TPM). The actual consumer of `helios-prod-aif-hub` is `ems-plan-narration-function-prod` via `pms-core-project`. Re-scoped Hub quota planning around the narration function.
  - **Caller Dependency on GAP-FM-002:** Consumer `func-orchestrator-sopfactoryprod1fd3k` runs on a Consumption plan without VNet integration across 25 dynamic outbound IPs. Sequenced VNet integration as a mandatory prerequisite to Phase 6 ingress lockdown.
  - Standardized HCL on nested `model`/`scale` objects and native `azurerm_container_app` probe/scaling syntax.

---

## 5. Prioritized Multi-Environment Action Items

| Priority | Action Item | Target Resource | Owning Team | Prerequisite / Sequencing |
|:---:|:---|:---|:---:|:---|
| **P0** | Bump baseline replicas to 2 (`minReplicas = 2`) | `ca-model-service-prod` | SOP Factory App Team | Immediate reliability fix. |
| **P0** | Implement `/api/health` probes in code & IaC | `ca-model-service-*` (DEV, QA, PROD) | SOP Factory App Team | Must deploy alongside or before autoscaling rules. |
| **P0** | Execute 3-phase auth rollout | `ca-model-service-dev` | Security / Platform SRE | Audit caller tokens $\to$ canary validation $\to$ enforce auth & whitelist. |
| **P1** | Deploy Log Analytics Diagnostic Settings | AI accounts across DEV, QA, PROD | Platform SRE | Stream SOP Factory to `log-sopfactory-*` and Hub to `helios-*-logs`. |
| **P1** | Deploy KQL Scheduled Query Rules (429, 5xx, P95) | Azure Monitor across DEV, QA, PROD | Observability / SRE | Bind to `ag-helios-ops`, `ag-helios-qa-ops`, `ag-helios-prod-ops`. |
| **P1** | Sequence Orchestrator VNet Integration | `func-orchestrator-sopfactory*` (QA, PROD) | Core Infra / App Team | Mandatory prerequisite before internalizing Container App ingress. |
| **P1** | Reconcile Narration Function Model Bindings | `ems-plan-narration-function-prod` | Narration / Infra Team | Audit model usage on `pms-core-project` before requesting Hub quota. |
| **P2** | Executive Consultation on 690k TPM Consolidation | `tonyguid-test-resource` (East US 2) | SRE Lead / Director Tony Guid | Consolidate 5x `gpt-5.4-pro` to reclaim regional quota. |
| **P2** | Reconcile Model IaC in `helios-infra` | `ai_model_deployments` (DEV, QA, PROD) | Platform SRE | Codify approved models using nested `model`/`scale` schema; import state. |
