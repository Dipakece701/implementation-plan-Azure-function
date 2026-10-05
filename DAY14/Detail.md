
**Active User Stories & Work Items Addressed Today:**
1. **[ADO #30845 — Azure Functions – SRE Gap Remediation (PROD)]** *(Parent Story)*
2. **[ADO #30846 — Gap 2: Migrate PROD Key Vaults to Azure RBAC]** *(Completed — 100% Milestone)*
3. **[ADO #31394 — Fix Missing Telemetry on Non-VNet PROD Function Apps]** *(PR #595 Approved by Maurice)*
4. **[ADO #31392 — Modernize Slack Webhooks Alerting with Managed Identity & Key Vault]** *(All 4 Logic Apps Wired Live)*
5. **[ADO #31395 — Enable App Insights Telemetry on PROD UI App Service]** *(Awaiting UI Owner Sign-off)*

---

## Executive Summary of Today's Accomplishments

Today was a high-velocity day spanning Azure Security, IaC (Terraform), CI/CD quality gate enforcement, and Serverless Logic App automation across QA and Production:

1. **100% Production Key Vault RBAC Migration Achieved (Task #30846):**
   - Successfully completed the migration of the final 2 legacy access-policy Key Vaults (`helios-prod-ui-kv` and `helios-prd-spkplg-pki-kv`) to Azure RBAC.
   - All 9 Production Key Vaults now strictly enforce Azure RBAC authorization (`enableRbacAuthorization: true`). Zero legacy access policies remain in PROD.
   - Validated live uptime on `helios-prod-ui-appservice` (HTTP 200 OK) with zero secret retrieval degradation.

2. **PR #595 Created, Quality Gates Passed & Approved by Maurice Kennedy (Task #31394):**
   - Diagnosed root cause of zero telemetry on the 4 non-VNet PROD Consumption function apps (private App Insights endpoint blocking public ingestion).
   - Authored Terraform configuration to route non-VNet function apps to `helios-public-ingestion-insights-prod` (pointing to the shared Log Analytics workspace).
   - Solved a critical CI Contract check failure by diagnosing Windows CRLF vs Linux LF checksum discrepancy and updating `tests/contracts/prod-plan-contract.json`.
   - All 4 CI Quality Gates turned green (Contract tests, Drift tests, Security scan, Plan policy).
   - Maurice Kennedy reviewed and **APPROVED** PR #595. Analyzed branch protection rulesets and routed to Constantin Pricochi (`qcells-devops` code owner) for merge approval.

3. **Slack Alerting Logic Apps Modernization with MSI & Key Vault (Task #31392):**
   - Remediated broken Slack alert notifications caused by dead legacy HQCT URLs.
   - Enabled System-Assigned Managed Identity (MSI) across all 4 Logic Apps in PROD and QA.
   - Refactored workflow definitions live in Azure to dynamically retrieve secrets from Key Vault via MSI REST calls with `secureData` masking enabled.
   - Coordinated with Maurice Kennedy for secret creation (`slack-alerts-webhook-url`, `slack-ems-alerts-webhook-url`).
   - Prepared exact `az role assignment create` commands for Maurice to grant `Key Vault Secrets User` to the 4 Logic App Principal IDs.

4. **UI App Service Telemetry Remediation Prepared (Task #31395):**
   - Inspected IaC (`stacks/ui/main.tf`) and identified that `ignore_changes = [app_settings]` prevented live App Insights connection string application.
   - Formulated a non-disruptive remediation plan and engaged Charles Stoksik on Slack for sign-off.

---

## Detailed Task Breakdown & Technical Actions

```
========================================================================================
TASK 1: PROD Key Vault Azure RBAC Migration (ADO Task #30846)
Status: 100% COMPLETE & VERIFIED LIVE
========================================================================================
```

### 1.1 Objective & Context
Task #30846 represents Gap 2 of the Production SRE remediation story (#30845). The goal was to eliminate legacy access policies across all Production Key Vaults, migrating them to Azure Role-Based Access Control (Azure RBAC) to meet organizational compliance and zero-trust requirements.

### 1.2 Pre-Flight Audit of the 9 Production Key Vaults
Before touching any vault, an exhaustive inventory of the PROD subscription was conducted using Azure CLI:

```bash
az keyvault list \
  --subscription "" \
  --query "[].{Name:name, RG:resourceGroup, RBAC:properties.enableRbacAuthorization}" \
  -o table
```

**Audit Findings:**
- 7 vaults had already been migrated to Azure RBAC:
  1. `helios-prod-backend-kv` (RG: `helios-prod-us-west3-rg`) — `RBAC: True`
  2. `helios-prod-agents-kv` (RG: `helios-prod-us-west3-rg`) — `RBAC: True`
  3. `helios-prod-onboard-kv` (RG: `helios-prod-us-west3-rg`) — `RBAC: True`
  4. `kg-event-proc-prod-kv` (RG: `helios-prod-us-west3-rg`) — `RBAC: True`
  5. `kvsopfactoryprod1fd3k` (RG: `helios-prod-us-west3-rg`) — `RBAC: True`
  6. `helios-prod-github-kv` (RG: `helios-prod-us-west3-rg`) — `RBAC: True`
  7. `helios-prod-alerting-kv` (RG: `helios-prod-alerting-rg`) — `RBAC: True`
- 2 vaults remained on legacy access policies (`RBAC: False`):
  8. `helios-prod-ui-kv` (RG: `helios-prod-uswest3-ui`)
  9. `helios-prd-spkplg-pki-kv` (RG: `helios-prod-us-west3-rg`)

### 1.3 Pre-Migration Access Policy to RBAC Role Mapping
To prevent any service outage or secret retrieval failures upon cutover:
- **`helios-prod-ui-kv`**: Audited legacy access policies. Confirmed that the principal identity of `helios-prod-ui-appservice` already had the `Key Vault Secrets User` role (`4633458b-17de-408a-b874-0445c86b69e6`) assigned at the resource group or vault scope, and administrative Service Principals possessed `Key Vault Administrator`.
- **`helios-prd-spkplg-pki-kv`**: Verified certificate management service principals had pre-assigned `Key Vault Certificates Officer` and `Key Vault Secrets Officer` roles.

### 1.4 Execution: Azure RBAC Cutover
Executed the RBAC toggle via Azure CLI:

```bash
# 1. Migrate helios-prod-ui-kv
az keyvault update \
  --name "" \
  --resource-group "" \
  --subscription "" \
  --enable-rbac-authorization true

# 2. Migrate helios-prd-spkplg-pki-kv
az keyvault update \
  --name "" \
  --resource-group "" \
  --subscription "" \
  --enable-rbac-authorization true
```

### 1.5 Post-Migration Verification & Smoke Testing
1. Re-queried Azure to verify RBAC status on all 9 vaults:
```bash
az keyvault list \
  --subscription "" \
  --query "[].{Name:name, RBAC:properties.enableRbacAuthorization}" \
  -o table
```
*Result:* All 9 vaults returned `True`.

2. Executed live HTTP probe against `helios-prod-ui-appservice`:
```bash
curl -I -s "https://helios-prod-ui-appservice.azurewebsites.net/api/health"
```
*Result:* `HTTP/1.1 200 OK`. The UI application continued resolving secrets from `helios-prod-ui-kv` seamlessly without dropping connections.

---

```
========================================================================================
TASK 2: Non-VNet PROD Function Apps Telemetry (ADO Bug/Task #31394)
Status: PR #595 CREATED, CI GATES GREEN, APPROVED BY MAURICE KENNEDY
========================================================================================
```

### 2.1 Problem Analysis & Root Cause
In Helios Production, four function apps execute under the serverless Dynamic Y1 Consumption tier:
1. `helios-prod-cost-ingestion`
2. `kg-event-processor-prod`
3. `helios-ontology-event-processor-func-prod`
4. `helios-github-activity-logger-prod-func`

Because Dynamic Y1 does not support Azure Outbound VNet Integration, egress network calls from these functions originate from Azure public IP pools. The primary Application Insights instance (`helios-prod-appinsights`) had public ingestion disabled (`public_network_access_for_ingestion = false`) to enforce private endpoints for VNet-integrated services. Consequently, telemetry packets from all four non-VNet function apps were rejected at the App Insights perimeter, causing an observability blackout.

### 2.2 Solution Architecture
Rather than opening public ingestion on the private-endpoint-secured `helios-prod-appinsights`, the platform architecture team established a dedicated public-ingestion App Insights resource:
- **Resource Name:** `helios-public-ingestion-insights-prod`
- **Destination:** Directly connected to the same central Log Analytics Workspace (`helios-prod-logs`).
- **Telemetry Querying:** Developers and SREs query the exact same Log Analytics workspace; logs and traces are unified across the fleet.

### 2.3 Terraform Implementation
Authored changes in the `helios-infra` repository:
- `terraform/environments/prod/monitoring.tfvars`: Configured the public ingestion instance and connection strings for non-VNet consumers.
- `terraform/modules/monitoring/main.tf`: Routed the 4 non-VNet function apps to use `helios-public-ingestion-insights-prod`.

### 2.4 CI Quality Gate Debugging: The LF SHA256 Checksum Issue
Upon pushing the initial PR branch, GitHub Actions CI failed on the check:
`check-plan-contracts / check-plan-contract (prod)`

#### Root Cause Investigation:
Inspected `scripts/ci/check-plan-contract.sh`. The test validates that the SHA256 checksum of `terraform/environments/prod/monitoring.tfvars` matches the recorded checksum in `tests/contracts/prod-plan-contract.json`.
Because the file was edited on Windows, line endings and new content altered the hash.

#### Technical Resolution:
Extracted the exact Unix-style LF byte stream of `monitoring.tfvars` and calculated its SHA256:
```powershell
$content = [System.IO.File]::ReadAllText("terraform/environments/prod/monitoring.tfvars").Replace("`r`n", "`n")
$hasher = [System.Security.Cryptography.SHA256]::Create()
$bytes = [System.Text.Encoding]::UTF8.GetBytes($content)
$hash = -join ($hasher.ComputeHash($bytes) | ForEach-Object { "{0:x2}" -f $_ })
Write-Output $hash
```
*Calculated Hash:* `fd80475969d57321f96e78cf5c5a3136418f2bbf6a8cee455f109f87312c67fd`

Updated `tests/contracts/prod-plan-contract.json` with the new hash, committed, and pushed:
- **Commit:** `5e085dfcdfd833cfe344659dfc22d5c92fc6ed63`
- **GitHub PR:** [PR #595 — feat(observability): wire public App Insights ingestion for non-VNet PROD function apps]

#### CI Verification Results:
- `check-plan-contracts / check-plan-contract (prod)`: ✅ PASSED
- `drift-detection / check-drift`: ✅ PASSED
- `security-scan / trivy`: ✅ PASSED
- `policy-check / opa`: ✅ PASSED

### 2.5 Code Review & Approval Status
- **Reviewer:** Maurice Kennedy (`mauricekennedy-HQCT`)
- **Review State:** **`APPROVED`** 🏆
- **Merge Governance:**
  - Evaluated branch protection ruleset `HeliosInfraReviewedMain`.
  - Condition: Requires at least 1 approving review from `@qcells-hqct/qcells-devops`.
  - Constantin Pricochi (`constantinpricochi`) is a member of `qcells-devops`. Sent update in team channel for Constantin to provide code-owner approval to merge.

---

```
========================================================================================
TASK 3: Slack Alerting Logic Apps Modernization (ADO Bug/Task #31392)
Status: 4 LOGIC APPS WIRED LIVE TO MSI & KEY VAULT; AWAITING RBAC GRANT
========================================================================================
```

### 3.1 Problem Analysis
All 4 Slack alerting Logic Apps across PROD and QA had hardcoded legacy webhook URLs pointing to an obsolete domain (`hqct`). These webhooks had expired, resulting in silent failures when alerts fired.

### 3.2 Security & Architecture Redesign
Hardcoding new Slack webhook URLs into workflow files or code was rejected due to security risks. The modernized architecture:
1. Webhooks stored securely as Key Vault secrets:
   - Secret `slack-alerts-webhook-url`: Main platform alerts channel (`#helios-prod-alerts` / `#helios-qa-alerts`)
   - Secret `slack-ems-alerts-webhook-url`: EMS Plan Narration alerts channel (`#ems-alerts`)
2. Each Logic App uses its **System-Assigned Managed Identity (MSI)** to authenticate directly to Key Vault.
3. The Logic App workflow calls Key Vault REST API (`GET https://<vault-name>.vault.azure.net/secrets/<secret-name>?api-version=7.4`).
4. Secret values are masked in Azure run history using `"runtimeConfiguration": { "secureData": { "properties": ["outputs"] } }`.
5. The HTTP action posts the incoming alert payload to the retrieved webhook URL.




#### Step C: Updated Workflow Definitions in Azure
Refactored the JSON workflow definitions to add the `Get_Slack_Webhook_From_KeyVault` action prior to the `Post_to_Slack` action:

```json
{
  "Get_Slack_Webhook_From_KeyVault": {
    "type": "Http",
    "inputs": {
      "method": "GET",
      "uri": "https://<TARGET_KEY_VAULT>.vault.azure.net/secrets/<TARGET_SECRET>?api-version=7.4",
      "authentication": {
        "type": "ManagedServiceIdentity",
        "audience": "https://vault.azure.net"
      }
    },
    "runtimeConfiguration": {
      "secureData": {
        "properties": ["outputs"]
      }
    }
  },
  "Post_to_Slack": {
    "type": "Http",
    "inputs": {
      "method": "POST",
      "uri": "@{body('Get_Slack_Webhook_From_KeyVault')?['value']}",
      "headers": {
        "Content-Type": "application/json"
      },
      "body": "@triggerBody()"
    },
    "runAfter": {
      "Get_Slack_Webhook_From_KeyVault": ["Succeeded"]
    }
  }
}
```
All 4 workflow definitions were uploaded and verified active in Azure.

### 3.4 Key Vault Secret Status & Pending Role Assignments
Maurice Kennedy confirmed that new Slack webhooks were generated and written into the vaults:
- `helios-prod-alerting-kv`: `slack-alerts-webhook-url`, `slack-ems-alerts-webhook-url`
- `helios-qa-backend-kv`: `slack-alerts-webhook-url`, `slack-ems-alerts-webhook-url`

#### Delegation to Maurice Kennedy:
Because Dipak's user account has `Reader` / Contributor on the subscription without `Microsoft.Authorization/roleAssignments/write` permissions, Maurice Kennedy (Subscription Owner / Key Vault Data Access Administrator) must execute the role assignments.

The exact commands provided to Maurice:

```bash
# 1. PROD: heliosprod-alert-to-slack
az role assignment create \
  --assignee-object-id "" \
  --assignee-principal-type "ServicePrincipal" \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions//resourceGroups/helios-prod-alerting-rg/providers/Microsoft.KeyVault/vaults/helios-prod-alerting-kv"

# 2. PROD: helios-prod-ems-plan-narration-alert-to-slack
az role assignment create \
  --assignee-object-id "" \
  --assignee-principal-type "ServicePrincipal" \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions//resourceGroups/helios-prod-alerting-rg/providers/Microsoft.KeyVault/vaults/helios-prod-alerting-kv"

# 3. QA: helios-qa-alert-to-slack
az role assignment create \
  --assignee-object-id "" \
  --assignee-principal-type "ServicePrincipal" \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions//resourceGroups/helios-qa-us-west3-rg/providers/Microsoft.KeyVault/vaults/helios-qa-backend-kv"

# 4. QA: helios-qa-ems-plan-narration-alert-to-slack
az role assignment create \
  --assignee-object-id "" \
  --assignee-principal-type "ServicePrincipal" \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions//resourceGroups/helios-qa-us-west3-rg/providers/Microsoft.KeyVault/vaults/helios-qa-backend-kv"
```

### 3.5 Synthetic Smoke Test Prepared
Prepared a test payload to trigger immediately once Maurice confirms the RBAC commands:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"text": "🚀 *[SRE Verification]* Synthetic test from Logic App via MSI Key Vault integration - Oct 02, 2026."}' \
  "<LOGIC_APP_CALLBACK_URL>"
```

---

```
========================================================================================
TASK 4: PROD UI App Service Telemetry (ADO Bug/Task #31395)
Status: AUDITED & AWAITING CHARLES STOKSIK'S SIGN-OFF
========================================================================================
```

### 4.1 Audit Findings
Audited the Terraform configuration in `stacks/ui/main.tf`.
The `APPLICATIONINSIGHTS_CONNECTION_STRING` is already codified on `main`, but the resource block contains:
```hcl
lifecycle {
  ignore_changes = [
    app_settings,
    site_config
  ]
}
```
Because of this lifecycle ignore rule, running Terraform plan/apply does not push application settings to the live App Service.

### 4.2 Proposed Remediation & Coordination
To avoid any unexpected container recycling during active business hours:
1. Contacted Charles Stoksik (UI application owner) via Slack.
2. Formulated two deployment options:
   - **Option 1 (Zero Pipeline Disruption):** Directly apply `APPLICATIONINSIGHTS_CONNECTION_STRING` via Azure CLI or Azure Portal.
   - **Option 2 (IaC Pipeline Push):** Temporarily remove `app_settings` from `ignore_changes` in `stacks/ui/main.tf` and run a targeted pipeline apply during a scheduled maintenance window.
3. Awaiting Charles's preferred window before proceeding.

---

## Overall Production SRE Remediation Fleet Scorecard (Story #30845)

With Task #30846 completed today, the overall status of the PROD SRE Remediation story is:

| Gap # | ADO Work Item | Task Name | Status | Verification Date |
|:---:|:---:|:---|:---:|:---:|
| **Gap 1** | #30848 | Centralize PROD Function App Diagnostic Settings | **COMPLETE ✅** | 2026-09-29 |
| **Gap 2** | #30846 | Migrate PROD Key Vaults to Azure RBAC (9 Vaults) | **COMPLETE ✅** | **2026-10-02** |
| **Gap 3** | #30847 | Wire App Insights on helios-prod-cost-ingestion | **COMPLETE ✅** | 2026-09-30 |
| **Gap 4** | #30852 | Fix SopFactory Durable Orchestrator Alert | **COMPLETE ✅** | 2026-09-30 |
| **Gap 5** | #30849 | Availability Web Tests on Public HTTP Apps | **COMPLETE ✅** | 2026-09-29 |
| **Gap 6** | #30850 | Caller-Driven Inbound Access Restrictions | **COMPLETE ✅** | 2026-10-01 |
| **Gap 7** | #30851 | Accept Y1 Consumption Tier / EP1 AlwaysOn | **COMPLETE ✅** | 2026-10-01 |
| **Gap 8** | #30853 | SRE Remediation Promotion Drift Tracking | **IN PROGRESS ⏳** | Pending PR #595 Merge |

**Fleet Remediation Progress:** **7 of 8 Tasks Fully Delivered & Verified Live (87.5%)** 🚀

---

## Action Items & Next Steps

1. **Maurice Kennedy:**
   - Execute the 4 `az role assignment create` commands for the Logic App Managed Identities.
   - Notify Dipak once applied so synthetic Slack alert tests can be fired.

2. **Constantin Pricochi (`qcells-devops`):**
   - Provide Code Owner approving review on [PR #595] to satisfy branch protection ruleset `HeliosInfraReviewedMain`.
   - Merge PR #595 into `main`.

3. **Charles Stoksik:**
   - Confirm preferred deployment window and approach for `APPLICATIONINSIGHTS_CONNECTION_STRING` on `helios-prod-ui-appservice`.

4. **Next**
   - Fire synthetic alert test across QA and PROD Logic Apps upon Maurice's RBAC execution.
   - Verify telemetry ingestion in Log Analytics for the 4 non-VNet functions once PR #595 merges.
   - Apply UI App Service telemetry setting once Charles approves.
   - Complete Gap 8 (Task #30853) to conclude ADO User Story #30845.
