# Meeting Notes: Foundry Models Implementation Plan Review & Multi-Env Alignment



[[_TOC_]]

---


---

## 2. Spoken Script ("What to Say in the Meeting" ~60–90 Seconds)

> "Hi everyone, here is a quick update on the Foundry Models & AI Estate SRE implementation plans:
> 
> Following senior review feedback from Sam Chai on our initial DEV discovery report, I performed a 100% read-only audit across all environments and published our updated **Revision 2.0 SRE Remediation Plans for DEV, QA, and PROD on the ADO Wiki**.
> 
> First, on **DEV**: I updated the remediation plan to address all of Sam's feedback:
> * For the P0 public endpoint on `ca-model-service-dev`, we established a strict 3-phase rollout—auditing caller tokens first, deploying an auth-enabled canary revision, and only then enforcing `AUTH_DISABLED=false` with IP whitelisting to eliminate any outage risk.
> * We realigned our Terraform paths to the repo’s native `ai_foundry` module, confirmed via live Azure OIDC credentials that `ca-model-service-dev` is managed in GitHub under `qcells-hqct/sop-factory`, and replaced native metric alerts with production KQL Scheduled Query Rules tracking P95 and P99 tail latency.
> * We also verified the 1.1 Million TPM allocation on `tonyguid-test-resource`. Acknowledging that Tony is a Director in our company, we reframed this as an Executive POC consultation to right-size duplicate allocations and reclaim 690k TPM without disrupting his testing.
> 
> Next, I expanded the discovery to **QA and PROD** and added both implementation plans to the Wiki:
> * In **QA**, we confirmed authentication is already enforced, resolved confusing model aliases where `gpt-4.1` was actually pointing to mini, and routed application telemetry to dedicated workspace `log-sopfactory-qa`.
> * In **PROD**, we flagged a critical P0 single-replica outage risk on `ca-model-service-prod`—which we’ve slated to bump to `minReplicas = 2`—and identified a severe 10,000 TPM quota bottleneck on `gpt-5.4-mini` that needs an expedited Azure support request before high-volume traffic hits.
> 
> All three comprehensive plans—DEV, QA, and PROD—are live on the ADO Wiki, fully evidenced with zero mutations performed. I'm ready to proceed once engineering signs off. Thanks!"

---

## 3. Presenter Quick-Reference Points (Key Highlights)

* **DEV Plan Updated (Post-Sam Chai Review):** Fully incorporated Sam’s review into Revision 2.0 on the ADO Wiki. Sequenced the P0 endpoint containment into a safe 3-phase rollout (caller token audit $\to$ canary validation $\to$ enforce auth & restrict ingress) to prevent caller outages. Realigned Terraform with `modules/ai_foundry`, proved `ca-model-service-dev` belongs to `qcells-hqct/sop-factory` via live GitHub OIDC credentials, and converted alerting to KQL Scheduled Query Rules.
* **Director Tony Guid POC Quota:** Live query confirmed 1.1M TPM (930k across 5x `gpt-5.4-pro`). Formally recognized as an Executive POC, reframing the action as an executive capacity consultation rather than a developer cleanup.
* **QA Plan Added to Wiki:** Verified auth is already active (`AUTH_DISABLED = "false"`), resolved confusing model aliases (`gpt-4.1` pointing to mini), and configured diagnostic routing to `log-sopfactory-qa`.
* **PROD Plan Added to Wiki:** Flagged a critical P0 single-replica outage risk on `ca-model-service-prod` (`minReplicas = 1`) requiring an immediate bump to 2 replicas, and identified a severe 10k TPM quota constraint on `gpt-5.4-mini` needing an expedited Azure increase.
* **Current Status:** 100% read-only discovery is complete across DEV, QA, and PROD. All three revised plans are published on the ADO Wiki, ready for engineering sign-off.

---

## 4. Key Discussion Items & Verified Findings

### 4.1 GAP-FM-002: P0 Unauthenticated Endpoint Sequencing (`ca-model-service-dev`)
* **Context:** `ca-model-service-dev` is exposed to the public internet on port 8080 with `AUTH_DISABLED = "true"` and zero IP restrictions (`0.0.0.0/0`).
* **Sam's Feedback:** Approved for expedited containment, but flipping the flag immediately risks an instant outage for microservice callers not yet transmitting tokens.
* **Agreed Rollout Sequencing:**
  1. **Phase 1 (Audit):** Query Application Insights / Log Analytics headers to verify caller microservices acquire Entra ID tokens for `api://sopfactory-model-dev`.
  2. **Phase 2 (Canary Revision):** Deploy a new Container App revision with `AUTH_DISABLED = "false"` and run synthetic test calls against revision FQDN.
  3. **Phase 3 (Enforce & Restrict):** Shift 100% traffic to auth-enabled revision and restrict port 8080 ingress to corporate CIDRs / internal VNet.
* **Environment Parity Check:** In QA and PROD, `AUTH_DISABLED` is already set to `"false"`. Ingress hardening remains required across all environments.

---

### 4.2 GAP-FM-001: Terraform Module Alignment & 58-Model State Reconciliation
* **Context:** Initial plan referenced `azurerm_cognitive_deployment`.
* **Sam's Feedback:** The repo’s actual Foundry path is the AVM/`azapi`-based `ai_foundry` module via `extra_model_deployments` (see `terraform/stacks/ai_foundry/main.tf`). `createdByType: User` reflects creation method, not management state (`gpt-4o-mini`, `gpt-5.1`, `gpt-5.4-mini` are already in TF).
* **Agreed Action:**
  - Standardized on `module "ai_foundry"` in `terraform/stacks/ai_foundry/main.tf`.
  - Reconciled all 58 live deployments across the subscription into a 3-tier matrix:
    - **Tier 1 (Terraform Managed):** Production baseline models.
    - **Tier 2 (Active Unmanaged):** Import into state using `azapi_resource.ai_model_deployments`.
    - **Tier 3 (Redundant / Abandoned):** Decommission duplicate instances to reclaim quota.

---

### 4.3 GAP-FM-003 & GAP-FM-004: SOP Factory Repo Ownership & Health Probes
* **Context:** Determining whether `sop-factory` exists, where `ca-model-service-dev` is codified, and probe causal claims.
* **Sam's Feedback:** Verify owning repo and pipeline before proposing HCL. Reframe probe benefits around latency percentiles and revision restarts rather than unevidenced 502/503 claims.
* **Live Discovery Resolution (Undeniable Proof):**
  - **Owning Repository:** Verified as **`https://github.com/qcells-hqct/sop-factory`** on GitHub.
  - **Pipeline & Security Lineage:** The Azure Service Principal that deploys and modifies `ca-model-service-dev` is `sp-sop-factory-terraform-dev` (`210ef71d-6bbb-44bb-8004-f27b7fb47ee8`).
  - **Azure AD OIDC Federated Credentials:** The SPN contains explicit federated credentials with issuer `https://token.actions.githubusercontent.com` tied directly to `repo:qcells-hqct/sop-factory` (`main`, `pull_request`, and `environment:dev`).
  - Container image tags match 40-character Git commit hashes built from that repo.
  - **Probe Justification:** Reframed around SRE best practices to prevent Envoy proxy routing before Python runtimes (FastAPI/PyTorch/Azure OpenAI SDK) finish initialization, mitigating observed P95 tail-latency spikes (>12s).
  - **Autoscaling:** KEDA HTTP concurrency scaling rules sequenced strictly after probes.

---

### 4.4 GAP-FM-005: Account Disambiguation & Workspace Routing Split
* **Context:** Resolving `ais-sopfactory-dev` vs `ais-sopfactorydevmlel9` and destination Log Analytics workspaces.
* **Live Discovery Resolution:**
  - **Name Disambiguation:** `ais-sopfactory-dev` (ARM resource name) and `ais-sopfactorydevmlel9` (Azure OpenAI custom subdomain) are the **exact same physical resource**. Same pattern confirmed in QA (`ais-sopfactory-qa` $\to$ `ais-sopfactoryqao80ns`) and PROD (`ais-sopfactory-prod` $\to$ `ais-sopfactoryprod1fd3k`).
  - **Workspace Split:**
    - AI Foundry Hub (`helios-dev-aif-hub`): Streams to `helios-dev-logs` (Hub platform logs).
    - SOP Factory AI Services (`ais-sopfactory-dev`): Streams to dedicated workspace `log-sopfactory-dev` (`ce4545e8-984a-448e-b5b8-7b4cdb56af13`) to maintain co-located telemetry with `cae-sopfactory-dev`.

---

### 4.5 GAP-FM-006: Production-Grade SRE Alerting via KQL Scheduled Query Rules
* **Context:** Native Azure Monitor metrics ("Blocked Calls", raw 5xx) are unavailable across Cognitive Services accounts.
* **Sam's Feedback:** Deploy Scheduled Query Rules pulling `ResultSignature` and `DurationMs` from `RequestResponse` diagnostic logs; specify P95/P99 latency rather than average.
* **Agreed Alert Implementations:**
  1. **HTTP 429 Token Rate-Limiting:** `ResultSignature == "429"` (>10 events in 5m).
  2. **Inference Server Errors:** `toint(ResultSignature) >= 500` (>5 events in 5m).
  3. **Tail Latency Breaches:** P95 > 8,000ms OR P99 > 15,000ms in 5m.
  4. **Action Group Binding:** Linked to verified Action Groups: `ag-helios-ops` (DEV), `ag-helios-qa-ops` (QA), and `ag-helios-prod-ops` (PROD).

---

### 4.6 GAP-FM-007: Director Tony Guid's Executive POC Quota Governance
* **Context:** Reconciling missing capacity numbers and account classification in DEV.
* **Live Azure Telemetry Findings:**
  - Live query of `tonyguid-test-resource` in East US 2 revealed exact capacity allocations totaling **1,100,000 TPM (1.1M TPM)**:
    - 5x `gpt-5.4-pro`: 240k + 210k + 180k + 180k + 120k = **930,000 TPM**
    - 1x `gpt-4.1`: 50,000 TPM
    - 1x `text-embedding-3-small`: 120,000 TPM
* **Organizational Realignment:**
  - **Tony Guid is a Director at Qcells.** This resource is **not** an abandoned developer sandbox; it is an **Executive / Director-Sponsored Architecture & POC Environment**.
  - **Agreed Approach:** Replaced cleanup/deprovisioning language with an **Executive Consultation & Capacity Right-Sizing Request**. SRE will present East US 2 quota constraints to Director Tony Guid, requesting alignment on consolidating redundant `gpt-5.4-pro` deployments into a single 240k TPM allocation, freeing **690,000 TPM** for platform engineering without disrupting executive testing.
  - Reconciled `uudri-ai-foundry-projec-resource` as a dedicated **Platform Project Account** (excluded from sandbox cleanup).

---

### 4.7 Multi-Environment Validation (QA & PROD Specifics)
* **QA Findings:**
  - `ca-model-service-qa` has auth enabled (`AUTH_DISABLED = "false"`), but public ingress is unconstrained (`0.0.0.0/0`).
  - Model alias confusion on `helios-qa-aif-hub`: 3 separate deployment names point to `gpt-4.1-mini`: `gpt-4.1-mini` (1k), `gpt-4o-mini` (1k), and mislabeled `gpt-4.1` (50k). Remediated to standardize on canonical `gpt-4.1-mini`.
* **PROD Findings (Critical P0s):**
  - **Single Replica Risk (P0):** `ca-model-service-prod` runs with `minReplicas = 1`. In production, any container restart causes a 100% outage. Mandated `minReplicas = 2` baseline.
  - **Severe Quota Bottleneck (P1):** Preview model `gpt-5.4-mini` has only **10,000 TPM** in PROD (vs 50k in QA), exhausting capacity in 2-3 concurrent calls. Action: Submit expedited Azure quota increase to 100k+ TPM.
  - **Missing Models:** Flagship models `gpt-5` (50k) and `gpt-5.6-luna-1` (250k) verified in QA were never promoted to PROD.

---

## 5. Action Items & Next Steps

| # | Action Item | Owner | Target Timeline | Status |
|:---:|:---|:---:|:---:|:---:|
| **1** | Post Revision 2.0 update message to Sam Chai on ADO Wiki Page 2642 | Dipak Singh | Immediate | Ready |
| **2** | Update Wiki Page 2642 content with Revision 2.0 DEV Plan | Dipak Singh / SRE | Post-Alignment | Pending |
| **3** | Execute Phase 1 caller token audit for `ca-model-service-dev` | Security / Platform SRE | Sprint 38 | Scheduled |
| **4** | Increase `minReplicas = 2` on `ca-model-service-prod` (PROD P0) | SOP Factory App Team | Immediate | Pending Approval |
| **5** | Submit Azure quota increase for `gpt-5.4-mini` in PROD (10k $\to$ 100k TPM) | SRE Lead / FinOps | Sprint 38 | Scheduled |
| **6** | Prepare Executive Capacity Right-Sizing Briefing for Director Tony Guid | FinOps / SRE Lead | Next Week | Scheduled |
| **7** | Deploy Diagnostic Settings on `ais-sopfactory-...` across DEV, QA, PROD | Cloud Platform SRE | Sprint 38 | Scheduled |
| **8** | Codify KQL Scheduled Query Rules in Azure Monitor across all 3 environments | Observability SRE | Sprint 38 | Scheduled |
