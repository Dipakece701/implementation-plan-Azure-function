# Azure Functions SRE Sync: QA Remediation Kickoff & Work Item Alignment


---

## 1. Executive Scorecard & ADO Hierarchy

```
[USER STORY #30165] Azure Functions — SRE Gap Remediation (QA)
│   ├── Linked to Discovery Story: #26565
│   └── Tags: qa-function-apps; sre-gap-remediation
│
├── [#30174] Arm availability web tests in QA (Gap 5) ───────────── [CLOSED ✅ - Maurice]
├── [#30166] Diagnostic settings on remaining 5 apps (Gap 4) ────── [NEW 🟡 - Ready]
├── [#30167] Inbound access restrictions: caller model (Gap 6) ──── [NEW 🟡 - Ready]
├── [#30168] Durable orchestrator failure alert (Gap 8) ─────────── [NEW 🟡 - Ready]
├── [#30169] Wire App Insights on cost-ingestion (Gap 3) ────────── [NEW 🟡 - Ready]
├── [#30170] Accept Y1 Consumption tier decision (Gap 7) ────────── [NEW 🟡 - Ready]
├── [#30171] Migrate 3 Key Vaults to Azure RBAC (Gap 1) ─────────── [NEW 🟡 - Ready]
└── [#30172] Enable Managed Identity on UUDRI apps (Gap 2) ──────── [NEW 🟡 - Ready]
```

---

## 2. Detailed Work Item & Gap Mapping Table

| Task ID | Work Item Title | Wiki Gap | Initial State | Scope & Acceptance Criteria Summary |
|:---|:---|:---:|:---:|:---|
| **[#30174]** | **Arm availability web tests in QA (Gap 5)** | Gap 5 | **`Closed`** ✅ | **Completed by Maurice.** Terraform apply `35393837219` deployed; `/api/health` returning HTTP 200 on `ontology` and `kg-event-processor`. |
| **[#30166]** | Configure diagnostic settings on remaining 5 apps | Gap 4 | `New` 🟡 | Deploy `diag-helios-qa` with `FunctionAppLogs` and `AllMetrics` to `helios-qa-logs` on the 5 unconfigured apps. |
| **[#30167]** | Restrict inbound access using caller-driven model | Gap 6 | `New` 🟡 | Apply `DenyPublicHttp` rule (priority 100) to 4 background apps; keep SCM open; keep HTTP routes accessible. |
| **[#30168]** | Add durable orchestrator failure alert | Gap 8 | `New` 🟡 | Scheduled query alert on `appi-sopfactory-qa` routing to `ag-helios-qa-ops` for failed orchestrations. |
| **[#30169]** | Wire Application Insights on cost-ingestion | Gap 3 | `New` 🟡 | Configure `APPLICATIONINSIGHTS_CONNECTION_STRING` linking app to `platform-backend-insights-qa`. |
| **[#30170]** | Accept Y1 Consumption tier in QA | Gap 7 | `New` 🟡 | Document AlwaysOn and Outbound VNet as not applicable on Y1 Consumption (aligns with Wiki page 2574). |
| **[#30171]** | Migrate 3 QA Key Vaults to Azure RBAC | Gap 1 | `New` 🟡 | Pre-assign `Key Vault Secrets User` roles to Managed Identities, then toggle `enable-rbac-authorization true`. |
| **[#30172]** | Enable Managed Identity on UUDRI apps | Gap 2 | `New` 🟡 | Enable System-Assigned Managed Identity via CLI on `UUDRI-bill-processor-qa-01` and `UUDRI-Function-App-qa-01`. |

---

## 3. Platform Readiness & Blocker Summary

* **Subscription RBAC:** Dipak's active `Contributor` role on `Helios - QA` (`663afada-2155-4c4d-b908-ac771ef2d133`) provides full permissions for diagnostic settings, network access restrictions, alert rules, and app settings.
* **Target Workspaces & Action Groups:** Verified live in `helios-qa-us-west3-rg`:
  * Log Analytics: `helios-qa-logs`
  * Action Group: `ag-helios-qa-ops` (short name: `helios-qa`)
  * Application Insights: `appi-sopfactory-qa` and `platform-backend-insights-qa`
* **Blockers:** **None.** All prerequisites are satisfied to begin immediate rollout.

---

## 4. Next Steps & Meeting Action Items

1. **Assign Child Tasks:** Review ownership distribution with Sam and Maurice for the 7 open child tasks.
2. **Execute First Wave Implementation:**
   * Pilot Task #30166 (Diagnostic Settings) on remaining 5 apps.
   * Pilot Task #30167 (Inbound Restrictions) on the 4 background event apps.
   * Deploy Task #30168 (Durable Orchestrator Alert) to `ag-helios-qa-ops`.
3. **Coordinate RBAC Grant:** Confirm subscription owner role to grant `Key Vault Secrets User` role assignment for Task #30171 prior to vault toggle.
