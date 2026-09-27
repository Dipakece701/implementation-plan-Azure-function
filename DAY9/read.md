
---

## 2. Before vs. After Implementation Scorecard

| SRE / Security Dimension | Baseline Before Work | Current State (After Implementation) | Verification Evidence |
|:---|:---:|:---:|:---|
| **Public Attack Surface** | All 10 QA apps wide open (`Allow all` default) | **50% Attack Surface Reduction** (5 background apps locked down) | Public HTTP probes return **HTTP 403 Forbidden** on all 5 locked apps. |
| **CI/CD Pipeline Safety** | Risk of deployment lockout if SCM not isolated | **100% Isolated & Protected** | Verified `scmIpSecurityRestrictionsUseMain = false` and `Allow all` on SCM endpoints. |
| **Public Web Services Health** | Potential false alerts if public endpoints blocked | **100% Operational** | Probes to `ontology` and `kg-event-processor` confirm **HTTP 200 OK**. |
| **Sam's Review Gaps** | `UUDRI-Function-App` unaddressed; SCM property unnamed | **Fully Resolved** | Idle site locked down with `DenyPublicHttp`; exact platform property named and verified. |

---

## 3. Caller Inventory & Verification Table (All 10 QA Apps)

| Category | # | Function App Name | Trigger / Workload | Inbound Rule | HTTP Probe | Status |
|:---|:---:|:---|:---|:---:|:---:|:---:|
| **Locked Down** | 1 | `helios-qa-cost-ingestion` | Timer (Daily 6 AM UTC) | `DenyPublicHttp` (Prio 100) | **HTTP 403** | Verified ✅ *(Pilot)* |
| **(5 Apps)** | 2 | `helios-device-telemetry-qa-func` | Event Hub (`telemetry-in`) | `DenyPublicHttp` (Prio 100) | **HTTP 403** | Verified ✅ |
| | 3 | `ems-plan-narration-function-qa` | Event Hub (`plan-narration-events`) | `DenyPublicHttp` (Prio 100) | **HTTP 403** | Verified ✅ |
| | 4 | `UUDRI-bill-processor-qa-01` | Blob Storage Trigger | `DenyPublicHttp` (Prio 100) | **HTTP 403** | Verified ✅ |
| | 5 | `UUDRI-Function-App-qa-01` | Idle S1 (0 functions) | `DenyPublicHttp` (Prio 100) | **HTTP 403** | Verified ✅ |
| **Preserved Public** | 6 | `helios-ontology-event-processor-func-qa` | HTTP (`/api/health`) + Event Hub | Default Allow | **HTTP 200** | Verified ✅ |
| **(5 Apps)** | 7 | `kg-event-processor-qa` | HTTP (`/api/health`) + Service Bus | Default Allow | **HTTP 200** | Verified ✅ |
| | 8 | `func-orchestrator-sopfactoryqao80ns` | HTTP (Onboarding UI) | Default Allow | **HTTP 200** | Verified ✅ |
| | 9 | `func-projector-sopfactoryqao80ns` | HTTP (SOP Publishing) | Default Allow | **HTTP 200** | Verified ✅ |
| | 10| `helios-github-activity-logger-qa-func` | Inbound GitHub Webhook | Default Allow | **HTTP 200** | Verified ✅ |

---

## 4. Q&A Cheat Sheet (Handling Team Questions)

### Q1: "What property guarantees CI/CD deployments won't break?"
* **Answer:**  
  `scmIpSecurityRestrictionsUseMain = false`. This tells Azure App Service to maintain a separate access restriction rule table for the SCM/Kudu deployment site (`*.scm.azurewebsites.net`). The SCM site retains its default `Allow all` rule (priority 2147483647), allowing GitHub Actions and Azure DevOps release pipelines to deploy packages without requiring private runners.

### Q2: "Why did we lock down `UUDRI-Function-App-qa-01` if it has no functions?"
* **Answer:**  
  Leaving an idle or empty site exposed to the public internet violates Zero Trust baselines and exposes the default Azure web host page to port scanners. Locking it down eliminates unnecessary attack surface. If the team deploys functions to it later, they can configure appropriate caller rules.

```
