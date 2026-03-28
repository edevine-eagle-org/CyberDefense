# ABS CyberDefense · Security Dashboard

A browser-based executive security dashboard for **American Bureau of Shipping**, built on top of Microsoft Sentinel, Microsoft Defender for Cloud, Microsoft Entra ID, Microsoft Defender for Endpoint, and several third-party data sources. All data is fetched live at runtime - nothing is stored or cached.

> **Designed and Built by:** Eddie Devine, SIEM Engineer - ABS CyberDefense

---

## Live Dashboard

**[https://edevine-eagle-org.github.io/CyberDefense/](https://edevine-eagle-org.github.io/CyberDefense/)**

Sign in with your ABS Azure AD account to load live data. No data is pre-loaded or mocked.

---

## Overview

The dashboard is a single HTML file (`index.html`) with no build step, no backend, and no server. It authenticates directly against Microsoft Azure using MSAL (Microsoft Authentication Library) via redirect-based login and queries Azure APIs in the browser. Every panel, KPI, and chart is driven by live KQL (Kusto Query Language) or Azure Resource Graph queries executed against your own tenant. Queries run in full with no result limits - all data is available for investigations.

---

## Architecture

```
Browser → MSAL (Azure AD Auth) → Azure REST APIs
                                  ├── Log Analytics (KQL) - Sentinel, MDE, Entra logs
                                  ├── Azure Resource Graph - Posture, RBAC, Policy
                                  └── Microsoft 365 Defender API - MDE device data
```

**No data leaves the browser.** Queries run client-side against your tenant's APIs using your own delegated credentials.

---

## Dashboard Sections

### 🏠 Home
Executive summary view with KPI cards for each department and a live tactical operations panel showing active incidents, alert volume, sign-in failures, UEBA anomalies, emails blocked, and DNS blocks.

### 🛡️ CyberDefense
| Tab | Data Sources | What It Shows |
|---|---|---|
| Executive Summary | Sentinel, Proofpoint, Umbrella | Incident overview, identity risk, email/DNS threats, alert trend |
| Sentinel Health | Log Analytics | Workspace inventory, rule health, ingestion by table, data connectors, analytic rules, incident MTTD/MTTR |
| MDE Coverage | Defender for Endpoint | Device onboarding coverage, high-exposure devices, vulnerabilities, incident detail |
| Identity Threats | Entra ID, Sentinel UEBA | Sign-in anomalies, UEBA anomalies, CA policy coverage, SP sign-in activity |
| Email Threats | Proofpoint TAP | Blocked/delivered threats by sender, classification, score |
| DNS Threats | Cisco Umbrella | Blocked domains by category, volume, identity |
| Privileged Access | Entra ID, ARG | Identity & PIM, managed identities, KV secrets/certs, CA policy |
| SharePoint | Office 365 | External sharing, bulk downloads, sensitive file activity, anonymous links |
| Saviynt | Saviynt ECM | Audit logs, user details, access requests, endpoints, user groups |
| OCI Threats | Oracle Cloud | Identity & sign-ins, user/group changes, Kubernetes, networking, compute |
| GitHub / DevOps | GitHub Enterprise | Dependabot alerts, secret scanning, audit log, actions |

### 🏗️ CyberArchitecture
| Tab | Data Sources | What It Shows |
|---|---|---|
| Executive Summary | ARG, Defender | Posture overview, infrastructure risk, compliance trend, secure score by subscription |
| Posture & Compliance | Defender for Cloud | Vulnerability management, attack paths, AKS security, backup coverage, orphaned resources, compliance, secure score |
| Networking | ARG | VNet overview, subnets, NSG rules, public IPs, VNet peering, VPN gateways, Azure Firewall, DDoS, private endpoints, Bastion |
| Compute & Infra | ARG | VM inventory, disk encryption, monitoring agents, AKS clusters, container registries, VMSS |
| App & Integration | ARG | App Services, API Management, Web App security, Function Apps |
| Storage & Data | ARG | Storage accounts, Key Vaults, secrets/certs, SQL/Postgres/MySQL/Cosmos/MongoDB |
| OCI Architecture | Oracle Cloud | Kubernetes, integration activity, networking, compute, database, key management |
| Workspace Health | Log Analytics | Workspace inventory, sentinel health, ingestion metrics |

### 📋 CyberGRC
| Tab | Data Sources | What It Shows |
|---|---|---|
| Executive Summary | ARG, Policy | Governance overview, access risk, policy compliance by subscription, regulatory controls |
| Governance | ARG | Subscription overview, management groups, RBAC summary, tag coverage, shadow subscriptions |
| Policy | Policy Insights, Defender | Policy compliance, regulatory compliance, secure score |
| Access Control | Entra ID, ARG | Identity & PIM, CA policy coverage, privileged roles, guest principals, RBAC summary |
| Conditional Access | Entra ID, Sentinel | Sign-in failures, legacy auth, geo distribution, CA policy coverage |
| Audit & Activity | Azure Activity, SharePoint, OCI | Audit overview, all activity, admin activity, permission changes |
| Regulatory | Defender for Cloud | Compliance frameworks, secure score, quick wins, remediation guidance |

---

## Portal Deep Links

Every table row includes an **Open** button that links directly to the exact item in the relevant portal - not a generic landing page. Confirmed portal destinations include:

| Panel | Links To |
|---|---|
| Sentinel Incidents | Microsoft Sentinel Incidents blade |
| Sentinel Alerts / Rule Health | Microsoft Sentinel Analytics blade |
| Ingestion / Logs | Microsoft Sentinel Logs blade |
| Data Connectors | Microsoft Sentinel Data Connectors blade |
| Sign-in Failures / Anomalies | Entra ID Sign-in Events |
| UEBA Anomalies | Risky Users (Entra ID Protection) |
| CA Policy Coverage | Entra Conditional Access Policies |
| SP Sign-in Activity | Entra Enterprise Applications |
| Secure Score | Microsoft Security exposure score |
| Quick Wins / Recommendations | Microsoft Security exposure recommendations |
| MDE Devices | Microsoft Defender device inventory |
| SharePoint Audit | Microsoft Purview Audit Search |
| Proofpoint | Proofpoint TAP dashboard |
| Umbrella DNS | Cisco Umbrella dashboard |
| Regulatory Compliance | Defender for Cloud Regulatory Compliance |
| Azure Resources | Direct Azure Portal resource deep link |
| Privileged Roles / RBAC | Subscription IAM blade |
| Guest Principals | Entra External Identities |

---

## Prerequisites

- An **Azure AD App Registration** in your tenant with the following delegated API permissions:
  - `https://management.azure.com/user_impersonation`
  - `https://api.loganalytics.io/Data.Read`
  - `https://graph.microsoft.com/SecurityEvents.Read.All`
  - `https://graph.microsoft.com/User.Read`
- A **Log Analytics Workspace** connected to Microsoft Sentinel
- The app registration redirect URI must be set to your GitHub Pages URL (e.g. `https://<org>.github.io/<repo>/`)

---

## Configuration

Open `index.html` and update the `CFG` object near the top of the `<script>` block:

```js
const CFG = {
  clientId:    'YOUR_APP_REGISTRATION_CLIENT_ID',
  tenantId:    'YOUR_AZURE_TENANT_ID',
  workspaceId: 'YOUR_LOG_ANALYTICS_WORKSPACE_ID',
  wsResourceId: '/subscriptions/YOUR_SUB_ID/resourceGroups/YOUR_RG/providers/Microsoft.OperationalInsights/workspaces/YOUR_WORKSPACE_NAME',
  subscriptions: ['YOUR_SUBSCRIPTION_ID'],
};
```

| Field | Where to find it |
|---|---|
| `clientId` | Azure Portal - App Registrations - your app - Application (client) ID |
| `tenantId` | Azure Portal - Azure Active Directory - Overview - Tenant ID |
| `workspaceId` | Log Analytics Workspace - Overview - Workspace ID |
| `wsResourceId` | Log Analytics Workspace - Properties - Resource ID |
| `subscriptions` | Azure Portal - Subscriptions - Subscription ID |

---

## Authentication

Sign-in uses **MSAL redirect flow** (`loginRedirect`) rather than popups, which avoids browser popup blockers. Clicking Sign In navigates to `login.microsoftonline.com`, authenticates, then redirects back to the dashboard automatically. A **Sign Out** button is available in the top-right corner next to your account email once signed in.

---

## Deployment

No build step required. Just commit `index.html` to the `main` branch of a GitHub Pages-enabled repository.

```bash
git add index.html
git commit -m "update dashboard"
git push origin main
```

GitHub Pages will serve it at `https://<org>.github.io/<repo>/`. Pages typically redeploys within 1-2 minutes of a push.

---

## Data Sources

| Source | Query Type | Used For |
|---|---|---|
| Microsoft Sentinel | KQL (Log Analytics) | Incidents, alerts, sign-in logs, UEBA, Proofpoint, Umbrella, MDE |
| Azure Resource Graph | ARG | Resources, RBAC, policy, posture, networking, storage |
| Microsoft Defender for Cloud | ARG | Secure score, recommendations, attack paths, compliance |
| Microsoft Entra ID | Entra / Sentinel | Sign-ins, risky users, CA policies, service principals |
| Microsoft Defender for Endpoint | MDE API | Device inventory, onboarding status, vulnerabilities |
| Proofpoint TAP | Custom Log (KQL) | Email threats blocked and delivered |
| Cisco Umbrella | Custom Log (KQL) | DNS blocks and malicious domains |
| Oracle Cloud Infrastructure | Custom Log (KQL) | OCI identity, compute, networking, database events |
| Saviynt ECM | Custom Log (KQL) | IAM audit, access requests, user management |
| GitHub Enterprise | Custom Log (KQL) | Dependabot, secret scanning, audit log, actions |
| Azure Activity | KQL | Governance audit, permission changes |
| SharePoint / Office 365 | KQL | File activity, external sharing, admin changes |

---

## Technology Stack

- **Auth:** [`@azure/msal-browser`](https://github.com/AzureAD/microsoft-authentication-library-for-js) v2.38.3 - redirect-based login/logout
- **Charts:** [`Chart.js`](https://www.chartjs.org/) v4.4.2
- **Fonts:** IBM Plex Mono, IBM Plex Sans, Rajdhani (Google Fonts)
- **No frameworks.** Pure HTML, CSS, and vanilla JavaScript.

---

## Security & Privacy

- Authentication is handled entirely by Microsoft MSAL - no credentials are stored anywhere in the app.
- All API calls are made directly from the browser to Microsoft APIs using your delegated token.
- No data is sent to any third-party server.
- No analytics, tracking, or logging of any kind.
- Queries are read-only - the dashboard makes no write operations to any Azure resource.

---

## Author

**Eddie Devine** - SIEM Engineer, ABS CyberDefense
American Bureau of Shipping


## Architecture

```
Browser → MSAL (Azure AD Auth) → Azure REST APIs
                                  ├── Log Analytics (KQL) - Sentinel, MDE, Entra logs
                                  ├── Azure Resource Graph - Posture, RBAC, Policy
                                  └── Microsoft 365 Defender API - MDE device data
```

**No data leaves the browser.** Queries run client-side against your tenant's APIs using your own delegated credentials.

---

## Dashboard Sections

### 🏠 Home
Executive summary view with KPI cards for each department and a live tactical operations panel showing active incidents, alert volume, sign-in failures, UEBA anomalies, emails blocked, and DNS blocks.

### 🛡️ CyberDefense
| Tab | Data Sources | What It Shows |
|---|---|---|
| Executive Summary | Sentinel, Proofpoint, Umbrella | Incident overview, identity risk, email/DNS threats, alert trend |
| Sentinel Health | Log Analytics | Workspace inventory, rule health, ingestion by table, data connectors, analytic rules, incident MTTD/MTTR |
| MDE Coverage | Defender for Endpoint | Device onboarding coverage, high-exposure devices, vulnerabilities, incident detail |
| Identity Threats | Entra ID, Sentinel UEBA | Sign-in anomalies, UEBA anomalies, CA policy coverage, SP sign-in activity |
| Email Threats | Proofpoint TAP | Blocked/delivered threats by sender, classification, score |
| DNS Threats | Cisco Umbrella | Blocked domains by category, volume, identity |
| Privileged Access | Entra ID, ARG | Identity & PIM, managed identities, KV secrets/certs, CA policy |
| SharePoint | Office 365 | External sharing, bulk downloads, sensitive file activity, anonymous links |
| Saviynt | Saviynt ECM | Audit logs, user details, access requests, endpoints, user groups |
| OCI Threats | Oracle Cloud | Identity & sign-ins, user/group changes, Kubernetes, networking, compute |
| GitHub / DevOps | GitHub Enterprise | Dependabot alerts, secret scanning, audit log, actions |

### 🏗️ CyberArchitecture
| Tab | Data Sources | What It Shows |
|---|---|---|
| Executive Summary | ARG, Defender | Posture overview, infrastructure risk, compliance trend, secure score by subscription |
| Posture & Compliance | Defender for Cloud | Vulnerability management, attack paths, AKS security, backup coverage, orphaned resources, compliance, secure score |
| Networking | ARG | VNet overview, subnets, NSG rules, public IPs, VNet peering, VPN gateways, Azure Firewall, DDoS, private endpoints, Bastion |
| Compute & Infra | ARG | VM inventory, disk encryption, monitoring agents, AKS clusters, container registries, VMSS |
| App & Integration | ARG | App Services, API Management, Web App security, Function Apps |
| Storage & Data | ARG | Storage accounts, Key Vaults, secrets/certs, SQL/Postgres/MySQL/Cosmos/MongoDB |
| OCI Architecture | Oracle Cloud | Kubernetes, integration activity, networking, compute, database, key management |
| Workspace Health | Log Analytics | Workspace inventory, sentinel health, ingestion metrics |

### 📋 CyberGRC
| Tab | Data Sources | What It Shows |
|---|---|---|
| Executive Summary | ARG, Policy | Governance overview, access risk, policy compliance by subscription, regulatory controls |
| Governance | ARG | Subscription overview, management groups, RBAC summary, tag coverage, shadow subscriptions |
| Policy | Policy Insights, Defender | Policy compliance, regulatory compliance, secure score |
| Access Control | Entra ID, ARG | Identity & PIM, CA policy coverage, privileged roles, guest principals, RBAC summary |
| Conditional Access | Entra ID, Sentinel | Sign-in failures, legacy auth, geo distribution, CA policy coverage |
| Audit & Activity | Azure Activity, SharePoint, OCI | Audit overview, all activity, admin activity, permission changes |
| Regulatory | Defender for Cloud | Compliance frameworks, secure score, quick wins, remediation guidance |

---

## Prerequisites

- An **Azure AD App Registration** in your tenant with the following delegated API permissions:
  - `https://management.azure.com/user_impersonation`
  - `https://api.loganalytics.io/Data.Read`
  - `https://graph.microsoft.com/SecurityEvents.Read.All`
  - `https://graph.microsoft.com/User.Read`
- A **Log Analytics Workspace** connected to Microsoft Sentinel
- The app registration must have its redirect URI set to your GitHub Pages URL

---

## Configuration

Open `index.html` and update the `CFG` object near the top of the `<script>` block:

```js
const CFG = {
  clientId:    'YOUR_APP_REGISTRATION_CLIENT_ID',
  tenantId:    'YOUR_AZURE_TENANT_ID',
  workspaceId: 'YOUR_LOG_ANALYTICS_WORKSPACE_ID',
  wsResourceId: '/subscriptions/YOUR_SUB_ID/resourceGroups/YOUR_RG/providers/Microsoft.OperationalInsights/workspaces/YOUR_WORKSPACE_NAME',
  subscriptions: ['YOUR_SUBSCRIPTION_ID'],
};
```

| Field | Where to find it |
|---|---|
| `clientId` | Azure Portal → App Registrations → your app → Application (client) ID |
| `tenantId` | Azure Portal → Azure Active Directory → Overview → Tenant ID |
| `workspaceId` | Log Analytics Workspace → Overview → Workspace ID |
| `wsResourceId` | Log Analytics Workspace → Properties → Resource ID |
| `subscriptions` | Azure Portal → Subscriptions → Subscription ID |

---

## Deployment

No build step required. Just commit `index.html` to the `main` branch of a GitHub Pages-enabled repository.

```bash
git add index.html
git commit -m "update dashboard"
git push origin main
```

GitHub Pages will serve it at `https://<org>.github.io/<repo>/`.

---

## Data Sources

| Source | Query Type | Used For |
|---|---|---|
| Microsoft Sentinel | KQL (Log Analytics) | Incidents, alerts, sign-in logs, UEBA, Proofpoint, Umbrella, MDE |
| Azure Resource Graph | ARG | Resources, RBAC, policy, posture, networking, storage |
| Microsoft Defender for Cloud | ARG | Secure score, recommendations, attack paths, compliance |
| Microsoft Entra ID | Entra / Sentinel | Sign-ins, risky users, CA policies, service principals |
| Microsoft Defender for Endpoint | MDE API | Device inventory, onboarding status, vulnerabilities |
| Proofpoint TAP | Custom Log (KQL) | Email threats blocked and delivered |
| Cisco Umbrella | Custom Log (KQL) | DNS blocks and malicious domains |
| Oracle Cloud Infrastructure | Custom Log (KQL) | OCI identity, compute, networking, database events |
| Saviynt ECM | Custom Log (KQL) | IAM audit, access requests, user management |
| GitHub Enterprise | Custom Log (KQL) | Dependabot, secret scanning, audit log, actions |
| Azure Activity | KQL | Governance audit, permission changes |
| SharePoint / Office 365 | KQL | File activity, external sharing, admin changes |

---

## Technology Stack

- **Auth:** [`@azure/msal-browser`](https://github.com/AzureAD/microsoft-authentication-library-for-js) v2.38.3
- **Charts:** [`Chart.js`](https://www.chartjs.org/) v4.4.2
- **Fonts:** IBM Plex Mono, IBM Plex Sans, Rajdhani (Google Fonts)
- **No frameworks.** Pure HTML, CSS, and vanilla JavaScript.

---

## Security & Privacy

- Authentication is handled entirely by Microsoft MSAL - no credentials are stored anywhere in the app.
- All API calls are made directly from the browser to Microsoft APIs using your delegated token.
- No data is sent to any third-party server.
- No analytics, tracking, or logging of any kind.
- Queries are read-only - the dashboard makes no write operations to any Azure resource.

---

## Author

**Eddie Devine** - SIEM Engineer, ABS CyberDefense  
American Bureau of Shipping
