
---

## 3. Before vs. After Implementation Scorecards

### A. Task #30848: Diagnostic Settings Scorecard (All 8 PROD Apps)
| # | Function App Name | Resource Group | Baseline (Before) | Current State (After Work) | Target Workspace |
|:---:|:---|:---|:---:|:---:|:---:|
| 1 | `helios-prod-cost-ingestion` | `helios-prod-us-west3-rg` | `None (0)` 🔴 | **`diag-helios-prod`** ✅ | `helios-prod-logs` |
| 2 | `kg-event-processor-prod` | `helios-prod-us-west3-rg` | `None (0)` 🔴 | **`diag-helios-prod`** ✅ | `helios-prod-logs` |
| 3 | `ems-plan-narration-function-prod` | `helios-prod-us-west3-rg` | `None (0)` 🔴 | **`diag-helios-prod`** ✅ | `helios-prod-logs` |
| 4 | `helios-github-activity-logger-prod-func` | `helios-prod-us-west3-rg` | `None (0)` 🔴 | **`diag-helios-prod`** ✅ | `helios-prod-logs` |
| 5 | `func-orchestrator-sopfactoryprod1fd3k` | `helios-prod-us-west3-rg` | `None (0)` 🔴 | **`diag-helios-prod`** ✅ | `helios-prod-logs` |
| 6 | `func-projector-sopfactoryprod1fd3k` | `helios-prod-us-west3-rg` | `None (0)` 🔴 | **`diag-helios-prod`** ✅ | `helios-prod-logs` |
| 7 | `helios-ontology-event-processor-func-prod` | `helios-prod-us-west3-rg` | `None (0)` 🔴 | **`diag-helios-prod`** ✅ | `helios-prod-logs` |
| 8 | `helios-alerting-prod-func` | `helios-prod-alerting-rg` | `None (0)` 🔴 | **`diag-helios-prod`** ✅ | `helios-prod-logs` |

**Fleet Coverage:** **8 of 8 Apps (100% Coverage)** 🏆

---

### B. Task #30849: Availability Web Tests Scorecard
| Web Test Name | Target App & Endpoint | Provisioned By | Verified By | Enabled Status | Probe Response | Compliance Status |
|:---|:---|:---:|:---:|:---:|:---:|:---:|
| `kg-event-processor-prod-availability` | `kg-event-processor-prod`<br>`/api/health` | Maurice (Terraform) | Dipak (Live Audit) | **True** ✅ | **HTTP 200 OK** | **COMPLIANT ✅** |
| `helios-ontology-event-processor-func-prod-availability` | `helios-ontology-event-processor-func-prod`<br>`/api/health` | Maurice (Terraform) | Dipak (Live Audit) | **True** ✅ | **HTTP 200 OK** | **COMPLIANT ✅** |

---

## 4. Master PROD SRE Remediation Progress (Parent Story #30845)

| Task ID | Gap Description | Owner | Status | Live Evidence / Outcome |
|:---:|:---|:---:|:---:|:---|
| **#30848** | **Gap 4:** Centralized Diagnostic Settings (8/8 Apps) | Dipak | **Completed** ✅ | 100% of apps streaming `FunctionAppLogs` & `AllMetrics` to `helios-prod-logs`. |
| **#30849** | **Gap 5:** Verify Availability Web Tests (`kg` & `ontology`) | Maurice / Dipak | **Verified** ✅ | Deployed via Terraform; probes returning HTTP 200 OK. Ready to close. |
| **#30847** | **Gap 3:** Wire App Insights on `cost-ingestion` | Dipak | `New` 🟡 | Next up in Wave 1. |
| **#30852** | **Gap 8:** SOP Factory Durable Orchestrator Alert | Dipak | `New` 🟡 | In Wave 1 backlog. |
| **#30853** | **Gap 1:** Service Promotion Drift & Missing Components | Dipak | `New` 🟡 | Fully specified & approved by Sam; tracks UUDRI, 2 funcs, Logic App. |
| **#30846** | **Gap 2:** Migrate 2 PROD Key Vaults to Azure RBAC | Dipak | `New` 🟡 | In Wave 2 backlog. |
| **#30850** | **Gap 6:** Inbound Access Restrictions (Caller Model) | Dipak | `New` 🟡 | In Wave 2 backlog. |
| **#30851** | **Gap 7:** Accept Y1 Tier & Configure AlwaysOn on EP1 | Dipak | `New` 🟡 | In Wave 2 backlog. |

**Current PROD Progress:** **2 of 8 Tasks Complete (25%)** 🚀
