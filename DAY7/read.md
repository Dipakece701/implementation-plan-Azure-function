
---

## 1. Before vs. After Implementation Scorecard

| Capability / SRE Dimension | Baseline Before Work | Current State (After Implementation) | Verification Evidence |
|:---|:---:|:---:|:---|
| **QA Diagnostic Settings (Task #30166)** | Only 5 of 10 apps configured (50% coverage; 5 apps had 0 logs) | **10 of 10 Apps Active (100% Coverage)** streaming `FunctionAppLogs` & `AllMetrics` | Verified via subscription audit; all 10 apps report `Count: 1` routing to `helios-qa-logs`. |
| **Durable Orchestration Alerting (Task #30168)** | 0 trigger alerts; silent failure risk on async document onboarding | **Active Scheduled Query Alert** (`alert-sopfactory-orchestrator-failure`) | Verified via `az monitor scheduled-query show`; Severity 1, routes to `ag-helios-qa-ops`. |

---

## 2. Detailed Verification Breakdown

### Task #30166: All 10 QA Function Apps Streaming to `helios-qa-logs`
1. `UUDRI-bill-processor-qa-01` (Windows Y1) ➔ **Active** (Workspace: `helios-qa-logs`)
2. `UUDRI-Function-App-qa-01` (Windows S1) ➔ **Active** (Workspace: `helios-qa-logs`)
3. `func-projector-sopfactoryqao80ns` (Linux Y1) ➔ **Active** (Workspace: `helios-qa-logs`)
4. `helios-device-telemetry-qa-func` (Linux Y1) ➔ **Active** *(Pilot App)*
5. `func-orchestrator-sopfactoryqao80ns` (Linux Y1) ➔ **Active** (Workspace: `helios-qa-logs`)
6. `helios-qa-cost-ingestion` (Linux Y1) ➔ **Active** (Pre-existing baseline)
7. `ems-plan-narration-function-qa` (Linux EP1) ➔ **Active** (Pre-existing baseline)
8. `helios-github-activity-logger-qa-func` (Linux Y1) ➔ **Active** (Pre-existing baseline)
9. `helios-ontology-event-processor-func-qa` (Linux Y1) ➔ **Active** (Pre-existing baseline)
10. `kg-event-processor-qa` (Windows Y1) ➔ **Active** (Pre-existing baseline)

### Task #30168: Alert Rule Live Parameters
* **Target Scope:** `appi-sopfactory-qa` (Application Insights)
* **KQL Logic:** `requests | where (name has 'sopFactoryOrchestrator' or operation_Name has 'sopFactoryOrchestrator') and success == false`
* **Notification Routing:** `ag-helios-qa-ops`
* **Schedule:** Evaluated every 5 minutes / 5-minute lookback window
* **Auto-Mitigate:** Enabled (auto-resolves once subsequent runs succeed)

---

## 3. QA Remediation Progress Tracker

```
[PARENT STORY #30165] QA SRE Remediation Progress: 3 of 8 Gaps Complete (37.5%)
│
├── [#30174] Availability Web Tests (Gap 5) ─────────────── [CLOSED ✅ - Maurice]
├── [#30166] Diagnostic Settings across 10 apps (Gap 4) ─── [COMPLETE ✅ - Dipak]
├── [#30168] Durable Orchestrator Failure Alert (Gap 8) ─── [COMPLETE ✅ - Dipak]
│
├── [#30167] Inbound Access Restrictions (Gap 6) ────────── [NEXT 🚀 - 5 apps lockdown]
├── [#30169] Wire App Insights on cost-ingestion (Gap 3) ── [UPCOMING 🟡]
├── [#30170] Accept Y1 Consumption Tier Decision (Gap 7) ── [UPCOMING 🟡]
├── [#30171] Migrate 3 Key Vaults to Azure RBAC (Gap 1) ──── [UPCOMING 🟡]
└── [#30172] Enable Managed Identity on UUDRI (Gap 2) ───── [UPCOMING 🟡]
```

---

## 5. Next Steps

1. **Mark Closed in ADO:** Update work items [#30166] and [#30168] to `Closed`.
2. **Kick Off Task #30167 (Inbound Restrictions):** Apply `DenyPublicHttp` rule on the 5 background/idle apps (`cost-ingestion`, `device-telemetry`, `ems-plan-narration`, `uudri-bill-processor`, and `UUDRI-Function-App-qa-01`), keeping SCM endpoints open (`scmIpSecurityRestrictionsUseMain = false`).
