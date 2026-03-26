# ABS CyberDefense — Security Dashboard

Live Azure security dashboard for the ABS IMS Security team. Queries Microsoft Sentinel, Azure Resource Graph, Log Analytics, and connected data sources in real time using the signed-in user's delegated Azure AD credentials.

**Live URL:** https://edevine-eagle-org.github.io/CyberDefense/

---

## Authentication

Sign in with your ABS Azure AD account (`@eagle.org`). The dashboard uses MSAL.js with PKCE — no passwords or secrets are stored anywhere.

**App Registration:** `EA-SecurityDashboard-Prod-AppReg`  
**Tenant:** American Bureau of Shipping (`d810b06c-d004-4d52-b0aa-4f3581ee7020`)  
**Workspace:** `law-security-prod` (`7123fe2d-f76e-4027-8784-f9b6eec61ba8`)

**Required roles on your account:**
- `Log Analytics Reader` on `law-security-prod`
- `Global Reader` or `Reader` at Management Group / subscription scope
- `Security Reader` at Management Group / subscription scope

---

## Dashboard Tabs

| Tab | Type | Coverage |
|---|---|---|
| 🛡️ Security Posture | ARG + LA | Defender recommendations, attack paths, KV expiry, UEBA, CA policy, backup, compliance, quick wins |
| 🔍 Sentinel Health | ARG + LA | Workspace inventory, rule health, ingestion, anomalies, connectors, incidents, MTTD/MTTR, Proofpoint, Umbrella |
| 🖥️ MDE Coverage | LA | Device onboarding, vulnerability exposure, incidents, alert volume, ingestion |
| 🏢 Tenant & Governance | ARG + LA | Subscriptions, management groups, RBAC, privileged roles, policy compliance, tag coverage, sign-ins, shadow subs |
| 🌐 Networking | ARG | VNets, subnets, NSGs, permissive rules, public IPs, peerings, gateways, firewall, DDoS, private endpoints |
| 💻 Compute | ARG | VMs, disk encryption, agent coverage, AKS, app services, container registries, VMSS |
| 🗄️ Data & Storage | ARG | Storage accounts, key vaults, secret/cert expiry, SQL, Cosmos DB, PostgreSQL/MySQL, backup coverage |
| 🤖 AI & Apps | ARG | AI services, shadow AI, AI posture, APIM, app inventory, risk scorecard, logic apps, function apps |
| 🐙 GitHub / DevOps | LA | Audit events, security alerts, workflow failures, secrets passed to runners, access & policy changes |

---

## Data Sources

| Source | Table(s) | Notes |
|---|---|---|
| Microsoft Sentinel | `SecurityIncident`, `SecurityAlert`, `SentinelHealth` | Incident and alert data |
| Microsoft Entra ID | `SigninLogs`, `AADServicePrincipalSignInLogs`, `AuditLogs` | Sign-in and identity data |
| Microsoft Defender for Endpoint | `DeviceInfo`, `DeviceNetworkEvents`, `DeviceTvmSoftwareVulnerabilities` | Endpoint coverage |
| UEBA | `BehaviorAnalytics` | User and entity behavior |
| Cisco Umbrella | `Cisco_Umbrella_dns_CL`, `Cisco_Umbrella_proxy_CL` | DNS and proxy logs |
| ProofPoint TAP | `ProofPointTAPMessagesBlocked_CL`, `ProofPointTAPMessagesDelivered_CL` | Email threat data |
| Azure Resource Graph | `Resources`, `SecurityResources`, `PolicyResources` | Cross-subscription inventory |
| GitHub | `GitHubAuditLogsV2_CL` | GitHub Enterprise audit logs |
| Log Analytics Usage | `Usage` | Ingestion volume and cost |

Coverage: **97 of 103 subscriptions** sending data to `law-security-prod`.

---

## How to Update the Dashboard

1. Edit `index.html` locally or download from this repo
2. Make your changes
3. Go to **github.com/edevine-eagle-org/CyberDefense**
4. Click **Add file → Upload files**
5. Drop in the updated `index.html` and commit
6. GitHub Pages deploys automatically — allow ~60 seconds

---

## How to Add a New Query Panel

In `index.html`, find the relevant dashboard array (e.g. `d5` for Security Posture) inside the `PANELS` object and add a new entry:

```js
{
  label: 'My Panel Title',
  type: 'arg',        // 'arg' for Resource Graph, 'la' for Log Analytics
  viz: 'timechart',   // optional: 'timechart', 'barchart', 'tiles' — omit for table
  query: `Resources
| where type == 'microsoft.compute/virtualmachines'
| summarize Count=count() by location`
}
```

To add a KPI tile to the strip, add an entry to `KDEFS[dashId]`:

```js
{
  l: 'My KPI Label',
  c: 'kc-cy',   // kc-cy=cyan, kc-rd=red, kc-gn=green, kc-yw=yellow, kc-bl=blue, kc-or=orange
  q: { t: 'arg', q: `Resources | summarize Count=count()` },
  k: 'Count'    // column name to display
}
```

---

## Config Reference

At the top of `index.html`:

```js
const CFG = {
  clientId:     '467a1486-6459-4661-8397-c1dd41cd38e7',   // App Registration Client ID
  tenantId:     'd810b06c-d004-4d52-b0aa-4f3581ee7020',   // ABS Tenant ID
  workspaceId:  '7123fe2d-f76e-4027-8784-f9b6eec61ba8',   // law-security-prod GUID
  wsResourceId: '/subscriptions/7ca56f5e-.../workspaces/law-security-prod',
  subscriptions: ['7ca56f5e-a44c-4311-a34a-0729be8f5a6e'],
};
```

If the App Registration is recreated or the workspace changes, update these values and re-upload.

---

## Maintainer

**Eddie Devine** — SIEM Engineer, ABS Cyber Defense  
`EDevine-C0@eagle.org`
