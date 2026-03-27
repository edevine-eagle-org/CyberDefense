# ABS Cyber Security Portal

Live multi-department Azure security dashboard for the ABS Cyber Security organization. Queries Microsoft Sentinel, Azure Resource Graph, Log Analytics, and connected third-party data sources in real time using the signed-in user's delegated Azure AD credentials.

**Live URL:** https://edevine-eagle-org.github.io/CyberDefense/

---

## Architecture

Three-tier navigation with an executive home landing page:

```
Home (Executive Overview)
├── CyberDefense       - SOC / threat detection / response
├── CyberArchitecture  - Cloud security posture / infrastructure
└── CyberGRC           - Governance / risk / compliance
```

Each department has its own Executive Summary dashboard plus dedicated sub-dashboards scoped to that team's function. Panels shared across departments (e.g. Privileged Roles, Secure Score) are duplicated with context-appropriate focus per department.

---

## Authentication

Sign in with your ABS Azure AD account (`@eagle.org` or `-c0` accounts). The dashboard uses MSAL.js with PKCE - no passwords or secrets are stored anywhere.

- **App Registration:** `EA-SecurityDashboard-Prod-AppReg`
- **Client ID:** `467a1486-6459-4661-8397-c1dd41cd38e7`
- **Tenant:** American Bureau of Shipping (`d810b06c-d004-4d52-b0aa-4f3581ee7020`)
- **Workspace:** `law-security-prod` (`7123fe2d-f76e-4027-8784-f9b6eec61ba8`)

**Required roles on your account:**
- `Log Analytics Reader` on `law-security-prod`
- `Reader` at Management Group or subscription scope
- `Security Reader` at Management Group scope

---

## Department Dashboards

### Home - Executive Overview
Loads on sign-in. Shows three department health cards with live KPIs and a recent Sentinel incidents ticker. Click any card or department tab to drill in.

### CyberDefense
*Audience: SOC team, analysts, CyberDefense lead*

| Sub-dashboard | Data Sources |
|---|---|
| Executive Summary | Active incidents, MTTD, sign-in failures, UEBA anomalies, email/DNS blocks |
| Sentinel Health | `SentinelHealth`, `SecurityIncident`, `SecurityAlert`, `Usage` |
| MDE Coverage | `DeviceInfo`, `DeviceTvmSoftwareVulnerabilities` |
| Identity Threats | `BehaviorAnalytics`, `SigninLogs`, `AADServicePrincipalSignInLogs` |
| Email Threats | `ProofPointTAPMessagesBlocked_CL`, `ProofPointTAPMessagesDelivered_CL` |
| DNS Threats | `Cisco_Umbrella_dns_CL`, `ASimDnsActivityLogs` |
| Privileged Access | `AuthorizationResources`, `SigninLogs` |
| SharePoint | `OfficeActivity` - external sharing, bulk downloads, sensitive files, permission changes |
| Saviynt | `SaviyntAuditLogs_CL` |
| OCI Threats | `OCI_LogsV2_CL` (identity and sign-in events) |
| GitHub / DevOps | `GitHubAuditLogsV2_CL` |

### CyberArchitecture
*Audience: Cloud security engineers, architects, CyberArchitecture lead*

| Sub-dashboard | Data Sources |
|---|---|
| Executive Summary | Secure Score, critical controls, attack paths, public exposure, unprotected VMs |
| Security Posture | `securityresources` - Defender recommendations, attack paths, compliance |
| Networking | VNet, NSG, Firewall, DDoS, Private Endpoints, Bastion |
| Compute | VMs, AKS, App Services, Container Registry, VMSS |
| Data & Storage | Storage accounts, Key Vaults, SQL, Cosmos DB, PostgreSQL/MySQL, backup |
| AI & Apps | Cognitive Services, APIM, Function Apps, Logic Apps, risk scorecard |
| OCI Infrastructure | `OCI_LogsV2_CL` (K8s, compute, networking, database, key management) |

### CyberGRC
*Audience: GRC team, compliance officers, CISO*

| Sub-dashboard | Data Sources |
|---|---|
| Executive Summary | Policy violations, privileged roles, guest principals, compliance %, secure score |
| Governance | Subscriptions, Management Groups, RBAC summary, tag coverage, shadow subs |
| Policy & Compliance | `policyresources`, `securityresources` - policy states, Defender controls |
| Access Reviews | Privileged roles, guest principals, RBAC summary, CA policy coverage |
| Conditional Access | Sign-in failures, legacy auth, MFA success, CA policy failures, geo distribution |
| Audit Trail | `SaviyntAuditLogs_CL`, `OfficeActivity` admin events, `OCI_LogsV2_CL` identity |
| Regulatory | Security controls mapped to frameworks, compliance by subscription, quick wins |

---

## Portal Links

Every panel row has a clickable Open button that navigates directly to the relevant portal:

| Link Type | Destination |
|---|---|
| `PortalLink` | Direct Azure portal resource, SharePoint file (via `OfficeObjectId`), or relevant admin page |
| `IAMLink` | Azure subscription Access Control (IAM) |
| `DefenderLink` | Defender for Cloud recommendation blade |
| `MDELink` | Microsoft Defender for Endpoint device page |
| `IncidentUrl` | Direct Sentinel incident |
| `SentinelLink` | Sentinel blade in `law-security-prod` workspace |
| `EntraLink` | Microsoft Entra ID (sign-ins, CA policies, risky users) |

SharePoint panels use `OfficeObjectId` as the direct link to the specific file or item when available, falling back to `Site_Url` for site-level events.

Sentinel links open directly to the `law-security-prod` workspace at the correct blade (Incidents, Analytics Rules, Logs, Alerts, Data Connectors).

---

## Subscription Filter

After sign-in, a Subscriptions dropdown appears in the top bar. It dynamically loads all subscriptions the signed-in user has access to.

- Default: All subscriptions selected (97 of 103 sending data to `law-security-prod`)
- Search by subscription name or ID
- Select All / Clear All / Apply to scope all ARG queries to specific subscriptions
- Log Analytics queries are workspace-scoped and cover all subscriptions feeding into `law-security-prod`
- Changing the selection clears the cache and re-runs all active panels

---

## Data Sources

| Source | Table(s) | Coverage |
|---|---|---|
| Microsoft Sentinel | `SecurityIncident`, `SecurityAlert`, `SentinelHealth` | All subs via law-security-prod |
| Microsoft Entra ID | `SigninLogs`, `AADServicePrincipalSignInLogs`, `BehaviorAnalytics` | Full tenant |
| Microsoft Defender for Endpoint | `DeviceInfo`, `DeviceTvmSoftwareVulnerabilities` | Onboarded devices |
| Azure Resource Graph | `Resources`, `SecurityResources`, `PolicyResources`, `AuthorizationResources` | All 103 subs |
| Cisco Umbrella | `Cisco_Umbrella_dns_CL`, `ASimDnsActivityLogs` | DNS/proxy logs |
| ProofPoint TAP | `ProofPointTAPMessagesBlocked_CL`, `ProofPointTAPMessagesDelivered_CL` | Email threats |
| SharePoint / M365 | `OfficeActivity` | SharePoint workload |
| GitHub Enterprise | `GitHubAuditLogsV2_CL` | GitHub audit events |
| Saviynt | `SaviyntAuditLogs_CL` | IAM audit logs |
| Oracle Cloud (OCI) | `OCI_LogsV2_CL` | Identity, K8s, compute, network, DB |
| Log Analytics Usage | `Usage` | Ingestion volume and cost |

Subscription coverage: 97 of 103 subscriptions sending data to `law-security-prod`.

---

## How to Update the Dashboard

1. Edit `index.html` locally or download from the repo
2. Make your changes
3. Go to **github.com/edevine-eagle-org/CyberDefense**
4. Click **Add file -> Upload files**
5. Drop in the updated `index.html` and commit
6. GitHub Pages deploys automatically - allow ~60 seconds

---

## How to Add a New Query Panel

Find the relevant panel array in `index.html` inside `const PANELS={...}` and add:

```js
{
  label: 'My Panel Title',
  type: 'arg',         // 'arg' for Resource Graph, 'la' for Log Analytics
  viz: 'timechart',    // optional: 'timechart', 'barchart', 'tiles' - omit for table
  query: `Resources
| where type == 'microsoft.compute/virtualmachines'
| extend PortalLink = strcat('https://portal.azure.com/#resource', id)
| project VMName=name, Location=location, PortalLink`
}
```

**Rules:**
- Every panel must include a link column (`PortalLink`, `DefenderLink`, `MDELink`, `IAMLink`, `IncidentUrl`, `SentinelLink`, or `EntraLink`) - these render as clickable Open buttons automatically
- LA panels must use `{TimeRange}` instead of hardcoded `ago(...)` - it is replaced at query time with the selected time range
- If the query uses `summarize`, define the link column after the summarize (or include it in the summarize with `any(PortalLink)`)
- ARG `union` queries mixing different table types (e.g. `Resources` + `securityresources`) are not supported - use separate panels

---

## How to Add a KPI Tile

Add an entry to the relevant array in `const KDEFS={...}`:

```js
{
  l: 'My KPI Label',
  c: 'kc-cy',   // kc-cy=cyan  kc-rd=red  kc-gn=green  kc-yw=yellow  kc-bl=blue  kc-or=orange
  q: { t: 'arg', q: `Resources | summarize Count=count()` },
  k: 'Count'    // column name to display
}
```

---

## Config Reference

At the top of `index.html`:

```js
const CFG = {
  clientId:     '467a1486-6459-4661-8397-c1dd41cd38e7',
  tenantId:     'd810b06c-d004-4d52-b0aa-4f3581ee7020',
  workspaceId:  '7123fe2d-f76e-4027-8784-f9b6eec61ba8',
  wsResourceId: '/subscriptions/7ca56f5e-a44c-4311-a34a-0729be8f5a6e/resourceGroups/rg-security-prod/providers/Microsoft.OperationalInsights/workspaces/law-security-prod',
};
```

Subscriptions are loaded dynamically at runtime. If the App Registration is recreated or the workspace changes, update these values and re-upload.

---

## Files in This Repo

| File | Purpose |
|---|---|
| `index.html` | The entire dashboard - single self-contained file |
| `abs-logo-blue.jpg` | ABS logo JPG (reserved) |
| `abs-logo-blue.png` | ABS logo PNG with transparency (reserved) |
| `README.md` | This file |

---

## Maintainer

**Eddie Devine** - SIEM Engineer, ABS CyberDefense
`EDevine-C0@eagle.org`
