---

## 1. What was Ticket AB#30853?

### Background & Context
During the platform SRE gap analysis comparing the QA environment (`helios-qa-us-west3-rg`) against PROD (`helios-prod-us-west3-rg`), several cloud services, function triggers, and logic apps operating in QA were found to be completely missing or drifting in PROD.

### The Problem
Leaving these differences unaddressed created operational blind spots and environment drift:
1. **Scope Ambiguity:** UUDRI apps (`UUDRI-bill-processor-qa-01`, `UUDRI-Function-App-qa-01`) and Device Telemetry (`helios-device-telemetry-qa-func`) existed in QA, but were never promoted to PROD. It was unclear if they were abandoned features, in-flight work, or intentional QA-only prototypes.
2. **EMS Plan Narration Drift:** The QA Plan Narration Function App had two active functions (`plan_narration_agent` and `realized_kpi_listener`), whereas PROD only had `plan_narration_agent`. The Event Hub listener was inactive in PROD due to an unseeded connection string secret.
3. **SOP Factory Orchestrator Drift:** The SOP Factory orchestrator in QA had a `pipelineSmeQueue` trigger that was absent in PROD.
4. **Weather Eventstream Monitor Drift:** A Logic App (`helios-data-weather-eh-eventstream-monitor-qa`) was active in QA, but completely missing in PROD.

### The Objective
Audit each drifted workload, remediate what belongs in PROD safely without breaking active read paths, defer idle infrastructure, and record architectural decisions so PROD and QA remain aligned.

---

## 2. Our Remediation Plan (5 Phases)

To execute safely without causing service disruptions, we structured the remediation into 5 distinct phases:

* **Phase 1: Workload Scope Sign-Off (UUDRI & Telemetry)**  
  Engage engineering leadership to formally classify `UUDRI` and `helios-device-telemetry-qa-func` as QA-only prototypes so unnecessary compute is not deployed to PROD.
* **Phase 2: EMS Plan Narration Parity (`realized_kpi_listener`)**  
  Obtain the required Key Vault secret, deploy the latest backend code to PROD, configure the Event Hub app settings, and verify both functions load and listen live.
* **Phase 3: SOP Factory Orchestrator Parity (`pipelineSmeQueue`)**  
  Safely promote the missing queue trigger without triggering an uncontrolled multi-service deployment that could break downstream production read paths.
* **Phase 4: Weather Eventstream Monitor Logic App**  
  Inspect live PROD weather Event Hub traffic. If active, deploy via Terraform; if idle, create a dedicated tracking item and defer deployment to avoid idle cloud resources.
* **Phase 5: Ticket Documentation & Governance**  
  Document the phased plan and live progress directly in Azure Boards work items using audit-grade native HTML.

---

## 3. What We Achieved & Live Updates

### ✅ Phase 2: EMS Plan Narration Parity — 100% COMPLETE & LIVE
* Maurice Kennedy seeded secret `EMS-REALIZED-KPI-EVENT-HUB-CONNECTION-STRING` in `helios-prod-backend-kv` (scoped to `helios-eventhub-soe-optimizer-kpi-results`).
* Dispatched GitHub Actions workflow in `qcells-hqct/helios-plan-narration-backend` (Run 37515048014) &rarr; Succeeded.
* Injected 3 App Settings:
  * `REALIZED_KPI_EVENT_HUB_CONNECTION_STRING` = `@Microsoft.KeyVault(...)`
  * `REALIZED_KPI_EVENT_HUB_NAME` = `helios-eventhub-soe-optimizer-kpi-results`
  * `REALIZED_KPI_EVENTHUB_CONSUMER_GROUP` = `realized-kpi-plan-narration`
* Executed ARM trigger sync. Verified live via Azure CLI: **2/2 functions loaded and actively running** in PROD (`plan_narration_agent`, `realized_kpi_listener`).

### ✅ Phase 4: Weather Eventstream Monitor — 100% RESOLVED & DECOUPLED
* Polled live Azure metrics on `helios-prod-eventhub-ns`: weather Event Hubs currently have **0 to 1 events** (idle; upstream ingestion not streaming yet).
* SRE decision with Maurice: Defer deploying an idle Logic App to PROD.
* Created standalone release tracking task **AB#33495** (*Deploy helios-data-weather-eh-eventstream-monitor-prod once upstream weather eventstream is active*) in `Helios\Devops` linked as Related to AB#30853. Phase 4 is complete.

### 🛡️ Phase 3: SOP Factory Orchestrator — GUARDRAIL ENFORCED
* Discovered hazard: Running `deploy-prod.yml` in `helios-sop-factory` deploys 8 services in parallel and breaks PROD read APIs (`catalog-api` / `projector` return `total: 0`) due to pre-#287 build dependencies.
* Established hard guardrail: Fold `pipelineSmeQueue` promotion into Sudhir's parent cutover ticket **AB#28606**.

### 🔄 Phase 1: Workload Scope Sign-Off — PIVOTED TO CONSTANTIN
* Samuel Chai is out on medical leave.
* Maurice recommended routing architectural sign-off to Constantin Pricochi. Maurice was asked to loop in Constantin to sign off on designating UUDRI and telemetry apps as QA-only prototypes.

### 📝 Phase 5: Documentation & Boards Audit — 100% COMPLETE
* Updated discussion on AB#30853 with audit-grade native HTML via REST API:
  * Comment `10519901`: 5-Phase Remediation Plan.
  * Comment `10519904`: Live Progress & Verification Status.

---

## 4. Daily Standup Story ("What I Say in Standup")

> "Yesterday and today, I worked on **AB#30853** to remediate PROD service promotion drift and missing components.
> 
> * **EMS Plan Narration (Phase 2):** Achieved 100% parity in PROD. Maurice seeded the Event Hub connection string secret in Key Vault, we deployed the backend via GitHub Actions, configured the app settings, and verified live that both the HTTP agent and the `realized_kpi_listener` are active and listening.
> * **Weather Monitoring (Phase 4):** Audited PROD Event Hub metrics and confirmed weather streams are currently idle. In alignment with Maurice, rather than deploying an idle Logic App, we decoupled it into tracking ticket **AB#33495**, which will be deployed once live upstream weather data starts streaming.
> * **SOP Factory (Phase 3):** Identified a risk where an isolated deploy breaks production read paths, so we established a guardrail with Maurice and are coordinating this promotion with Sudhir under **AB#28606**.
> * **Prototype Scope (Phase 1):** Sam Chai is on medical leave, so Maurice is looping in Constantin for architectural sign-off to designate UUDRI and telemetry apps as QA-only prototypes.
> 
> **Blockers / Next Steps:**  
> Waiting on the intro to Constantin for the Phase 1 sign-off and syncing with Sudhir on Phase 3 so we can close out AB#30853."

---

## 5. Quick Reference & Links

| Item | Reference Link | Status |
| :--- | :--- | :---: |
| **Main Ticket** | AB#30853 (PROD Service Drift Gap 1) | Active |
| **Weather Tracking Ticket** | AB#33495 (Weather Monitor Release) | New / Backlog |
| **SOP Cutover Ticket** | AB#28606 (Sudhir's SOP Cutover) | Dependency |
| **Deployment Run** | GitHub Actions Run 37515048014 | Succeeded ✅ |
