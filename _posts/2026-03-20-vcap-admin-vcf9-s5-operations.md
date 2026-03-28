---
layout: post
title: "VCAP-ADMIN VCF 9.0 — VCF Operations & Fleet: Objectives 4.35–4.47"
date: 2026-03-20
categories: [certification, vcf, vcap]
tags: [vcap, vcf9, vcf-operations, fleet-management, 3v0-11-26, study-guide]
series: "VCAP-ADMIN VCF 9.0 Study Guide (3V0-11.26)"
series_part: 5
---

# Section: VCF Operations & Fleet Management — Objectives 4.35–4.47

Part of the [VCAP-ADMIN VCF 9.0 Study Guide series]({% post_url 2026-03-20-vcap-admin-vcf9-study-guide-index %}).

---

<details markdown="1">
<summary><strong>4.35 — Perform VCF Operations Day 2 Tasks</strong></summary>

### Key Concepts

VCF Operations consists of three components:
1. **VCF Operations** (formerly vROps) — analytics, alerting, capacity, cost, policy
2. **VCF Operations for Logs** (formerly vRLI) — centralized log collection and analysis
3. **VCF Operations for Networks** (formerly vRNI) — network visibility and troubleshooting

**Centralized Log Collection:**
In VCF 9, log forwarding uses the **Cloud Proxy** (not the deprecated Log Forwarding agent). The Cloud Proxy aggregates logs from vSphere, NSX, and other VCF components and forwards them to VCF Operations for Logs.

For NSX syslog to VCF Operations for Logs: configure NSX **Node Profiles** → syslog to the Cloud Proxy IP using the **LI protocol** (port 9000 for LI plain, 9543 for LI-TLS). The VCF Operations analytics appliance does not directly receive syslog — the Cloud Proxy/collector handles that.

**Linking vCenter to VCF Operations:**
- VCF Operations can link up to **15 vCenter instances**
- All linked vCenters must share a **common Identity Provider** (VCF SSO or same LDAP)
- Deactivate Enhanced Linked Mode (ELM) before connecting vCenter to VCF Operations SSO
- Path: `VCF Operations > Administration > Cloud Accounts > Add vCenter`

**VCF Operations for Logs Day 2:**
- Log sources configured via: `VCF Operations for Logs > Administration > Log Sources`
- Create log sources for ESXi hosts, NSX Manager, vCenter, SDDC Manager
- Log filtering and archiving policies configurable per environment

**Day 2 maintenance tasks:**
- Certificate renewal: VCF Operations certificates managed via Fleet Management
- Scale out: add VCF Operations nodes (Analytics or Data nodes) for larger environments
- Backup: VCF Operations backup via `Administration > Backup and Restore`

### Exam Decision Points
- Cloud Proxy = **replaces** Log Forwarding agent for NSX log collection
- NSX syslog to VCF Ops for Logs = **LI protocol** via Cloud Proxy (not direct to analytics appliance)
- Max vCenters linked to VCF Operations = **15**
- Linking vCenters requires **common IDP** and **ELM deactivated first**

</details>

---

<details markdown="1">
<summary><strong>4.36 — Reclaim Resources using VCF Operations</strong></summary>

### Key Concepts

VCF Operations identifies wasted resources across 6 categories, ranked by reclamation value:

1. **Powered-off VMs** — highest potential savings; VMs consuming storage but no compute
2. **Idle VMs** — VMs powered on but with negligible CPU/memory/network activity
3. **Snapshots** — old snapshots consuming significant extra storage
4. **Orphaned disks** — VMDKs on datastores with no associated VM
5. **Oversized VMs** — VMs with far more CPU/memory than they actually use
6. **Reclaimable hosts** — hosts that could be decommissioned based on capacity analysis

**Navigation:**
`VCF Operations > Optimize > Reclaim`

**Two savings dashboards:**
- **Potential Savings** — what could be reclaimed if action is taken
- **Realized Savings** — what has actually been reclaimed (post-action tracking)

**Powered-off VM reclamation workflow:**
VCF Operations identifies VMs that have been powered off for configurable periods. Review list → take action (delete or archive).

**Idle VM detection:**
Defined by CPU, memory, disk I/O, and network activity thresholds being below a threshold for a sustained period. Configurable idle thresholds in VCF Operations Optimization settings.

**Snapshot reclamation:**
Lists snapshots older than a configurable age threshold. VCF Operations cannot delete snapshots directly — you take action in vSphere Client. VCF Operations tracks the delta.

**Orphaned disk detection:**
VMDKs in datastores that are not associated with any registered VM. Must verify before deleting — some intentionally orphaned disks (e.g., detached RDMs) may be in use by non-vCenter systems.

### Exam Decision Points
- Reclamation priority order: **Powered-off > Idle > Snapshots > Orphaned Disks > Oversized > Reclaimable Hosts**
- VCF Operations identifies; **action is taken in vSphere Client** (VCF Ops doesn't delete directly in most cases)
- Realized Savings = **post-action** tracking (not just identified savings)
- Orphaned disks: **verify before deleting** — may be intentionally detached

</details>

---

<details markdown="1">
<summary><strong>4.37 — Rightsize Workloads using VCF Operations</strong></summary>

### Key Concepts

Rightsizing adjusts VM resource allocations (CPU and memory) to better match actual workload consumption, eliminating waste and improving density.

**Navigation:**
`VCF Operations > Optimize > Rightsize`

**Rightsizing recommendations:**
VCF Operations analyzes historical CPU and memory utilization for each VM and recommends:
- **Rightsize up** — VM is consistently hitting its limits; increase allocation
- **Rightsize down** — VM is consistently far below its allocation; reduce (reclaim capacity)

**Rightsizing calculation basis:**
- Uses configurable time window (e.g., last 30 days)
- Applies a buffer/headroom percentage to avoid under-provisioning
- Considers peak utilization, not just average

**Applying rightsizing recommendations:**
Recommendations are advisory — the administrator reviews and applies changes in vSphere Client. VCF Operations does not directly modify VM configurations (in standard mode).

**Rightsizing in context of Operational Intent (link to 4.41):**
Operational Intent's **Consolidate** mode aggressively rightsizes and consolidates VMs to reduce host count. **Balance** mode rightsizes to spread load more evenly.

**Key consideration:**
Rightsizing memory down on a VM with memory limits already set may cause unexpected ballooning. Always review current memory limits before reducing allocation.

### Exam Decision Points
- Rightsizing recommendations = **advisory**; admin applies in vSphere Client
- Recommendations factor in **peak utilization** + headroom buffer (not just average)
- Rightsize down = reduce vCPU/vRAM (reclaim capacity)
- Rightsize up = increase vCPU/vRAM (prevent contention)
- Check for memory limits before rightsizing memory down

</details>

---

<details markdown="1">
<summary><strong>4.38 — Perform Capacity Forecasting What-If Scenarios</strong></summary>

### Key Concepts

VCF Operations What-If scenarios allow capacity planning simulations without modifying the live environment.

**Navigation:**
`VCF Operations > Plan > What-If Analysis`

**7 What-If scenario types:**

| Scenario Type | What it simulates |
|---|---|
| Add VMs | Impact of adding a specified number of VMs with defined profiles |
| Remove VMs | Effect of decommissioning VMs |
| Add Hosts | Available capacity when new hosts are added |
| Remove Hosts | Impact of removing hosts (planned decommission or failure) |
| Add Datastores | Storage capacity effect |
| Remove Datastores | Storage impact of datastore removal |
| Migrate Workloads | Moving VMs between clusters/datacenters |

**Combining scenarios:**
Multiple scenarios can be combined — but only if they target the **same object** (e.g., same cluster). You cannot combine scenarios targeting different clusters in a single analysis.

**Committed Scenarios:**
When you commit a What-If scenario, VCF Operations **reserves the projected capacity** in its planning calculations. This prevents the capacity model from showing free capacity that is already earmarked for a planned project. Committed scenarios appear in the capacity dashboard as planned utilization.

**Missing allocation values (overcommit ratios):**
If What-If results show empty or missing CPU/memory allocation values, this indicates that **overcommit ratios are not enabled** in the VCF Operations capacity settings. Enable them via `Administration > Global Settings > Capacity`.

**Time-based forecasting:**
VCF Operations uses historical trend data to project when capacity will be exhausted (runway analysis). The default projection period is configurable (typically 30/60/90 days).

### Exam Decision Points
- Combining scenarios requires **same object** (same cluster, etc.)
- Committed Scenarios = **reserve capacity** in planning model
- Missing allocation values = **overcommit ratios not enabled** in capacity settings
- What-If scenarios are **non-destructive** — no changes to live environment
- 7 scenario types to know: Add/Remove VMs, Add/Remove Hosts, Add/Remove Datastores, Migrate Workloads

</details>

---

<details markdown="1">
<summary><strong>4.39 — Configure Cost Management in VCF Operations</strong></summary>

### Key Concepts

VCF Operations Cost Management assigns financial values to virtual infrastructure resources, enabling cost showback and chargeback.

**8 cost drivers (cost categories):**
1. CPU
2. Memory
3. Storage
4. Network (optional)
5. License costs
6. Power and cooling
7. Facilities (rack space, floor space)
8. Maintenance/support

**Two pricing model types:**

| Model | Description |
|---|---|
| Rate-based | Fixed cost per unit ($/vCPU/month, $/GB RAM/month, $/GB storage/month) |
| Cost-based | Total infrastructure cost divided proportionally across VMs by consumption |

**Pricing Cards:**
Pricing cards define the cost per resource unit per environment. Multiple pricing cards can exist for different cost models (e.g., Production pricing vs Development pricing).

`VCF Operations > Plan > Cost > Pricing Cards > Add`

**Deployment modes (mutually exclusive):**
- **All Datacenters mode** — one pricing card applies to all datacenters/clusters
- **Specific Datacenter mode** — different pricing cards for different datacenters

> ⚠️ All Datacenters and Specific Datacenter modes are **mutually exclusive** — you cannot mix them. Choose one approach and apply consistently.

**Cost recalculation:**
VCF Operations recalculates costs every **24 hours**. Cost data is not real-time — there is up to a 24-hour lag after configuration changes before costs reflect accurately.

**Business Groups:**
Cost data can be segmented by Business Groups (mapped to VCF Operations groups/tags) to provide per-team or per-project cost visibility.

### Exam Decision Points
- 8 cost drivers — CPU, Memory, Storage, Network, License, Power/Cooling, Facilities, Maintenance
- All Datacenters vs Specific Datacenter = **mutually exclusive** — cannot mix
- Cost recalculates every **24 hours** (not real-time)
- Rate-based = fixed $/unit; Cost-based = total cost divided proportionally
- Pricing cards are applied per **deployment mode** choice

</details>

---

<details markdown="1">
<summary><strong>4.40 — Create, Update, and Apply a VCF Operations Policy</strong></summary>

### Key Concepts

VCF Operations Policies define which alert definitions, symptom definitions, and analysis settings apply to a group of objects. They are the mechanism for customizing monitoring and capacity behavior per object group.

**6 configurable policy elements:**
1. Alert definitions (which alerts are active)
2. Symptom definitions (which symptoms are monitored)
3. Capacity and time remaining settings
4. Workload automation settings (DRS integration)
5. Custom metrics/super metrics
6. Compliance benchmarks

**Navigation:**
`VCF Operations > Configure > Policies > Policy Library`

**Default Policy:**
There is always a **Default Policy** in VCF Operations. It always has the lowest rank (rank D). Every object not explicitly assigned to another policy is governed by the Default Policy. The Default Policy **cannot be deleted**.

**Policy ranking:**
Policies are ranked. When multiple policies apply to an object (through group membership), the highest-ranked policy wins. Reordering policies requires at least **2 active policies** — you can't reorder with only the Default Policy.

**Creating a policy:**
1. `Policy Library > Add`
2. Name the policy and set rank
3. Configure elements (inherit from base policy or customize)
4. Apply to object groups

**Applying a policy:**
Policies are applied to **object groups** (custom groups, vSphere tags groups, applications, etc.). An object not in any policy's scope falls back to the Default Policy.

**Updating a policy:**
Changes to a policy apply immediately to all objects governed by that policy. No restart or reapplication needed — VCF Operations picks up policy changes in the next analysis cycle.

### Exam Decision Points
- Default Policy = **always rank D** (lowest); cannot be deleted
- Objects not in any policy's scope = **governed by Default Policy**
- Reorder Policies requires **2+ active policies** (Default alone cannot be reordered)
- Policy changes apply **immediately** to governed objects
- 6 configurable policy elements (know the list)

</details>

---

<details markdown="1">
<summary><strong>4.41 — Configure Business Intent and Operational Intent</strong></summary>

### Key Concepts

**Operational Intent:**
Defines how DRS should manage workload distribution within a cluster. VCF Operations pushes Operational Intent settings to vCenter as DRS configuration.

**Three Operational Intent modes:**

| Mode | DRS Behavior |
|---|---|
| Balance | Distribute workloads evenly across hosts — standard DRS balancing |
| Consolidate | Aggressively pack VMs onto fewer hosts (for maximum host power savings) |
| Moderate | Balanced approach between Balance and Consolidate |

**Cluster Headroom Buffer:**
Operational Intent includes a configurable headroom buffer — the % of cluster capacity reserved for failover and burst. This feeds into capacity calculations.

`VCF Operations > Configure > Operational Intent > [Cluster] > Edit`

**Business Intent:**
Business Intent applies business context to VMs using **vCenter tags** — but the workflow is indirect:

1. Define tags in **VCF Operations** (not vCenter)
2. VCF Operations pushes tag definitions **to vCenter**
3. Apply the tags to VMs in **vSphere Client**
4. Reference the tags back in **Business Intent UI** in VCF Operations

> ⚠️ Tags must be created in VCF Operations first, then applied in vSphere Client. The workflow does NOT start in vCenter.

**WLP Settings permission:**
To configure Operational Intent, the VCF Operations user must have **WLP Settings Write** permission (Workload Placement settings). Without this, the Operational Intent configuration is read-only.

### Exam Decision Points
- Operational Intent modes: **Balance / Consolidate / Moderate**
- Business Intent tag workflow: **VCF Operations → vCenter → vSphere Client → VCF Operations** (not from vCenter first)
- WLP Settings Write permission required to configure Operational Intent
- Consolidate mode = **pack VMs onto fewer hosts** (density focus)
- Balance mode = **spread across hosts** (performance focus)

</details>

---

<details markdown="1">
<summary><strong>4.42 — Create Business Applications in VCF Operations</strong></summary>

### Key Concepts

Business Applications (or just "Applications" in VCF Operations) group VMs by business function, enabling application-level monitoring, alerting, and cost visibility.

**Navigation:**
`VCF Operations > Environment > Applications > Add`

**Application types by creation method:**

| Method | Type | Behavior |
|---|---|---|
| Manually created | Static | Members defined manually; do not change unless edited |
| Auto-discovered (Service Discovery) | Dynamic | Members discovered and updated automatically based on traffic flows |

**Application structure:**
Applications can contain **Tiers** (optional). Tiers group VMs by function within the application (e.g., Web Tier, App Tier, DB Tier). Tiers are optional — a single-tier application is valid.

**Application metadata:**

| Field | Options |
|---|---|
| Business Criticality | Low / Medium / Critical |
| Environment | DR / Dev / Prod / Staging / Test |

These metadata fields feed into alert prioritization and cost segmentation.

**Service Discovery (auto-discovery):**
Requires VCF Operations for Networks integration. Traffic flows between VMs are analyzed to infer application topology and group VMs automatically into discovered applications.

**Why Applications matter:**
- Application-level health views consolidate VM alerts into a single application health score
- Cost management can be segmented per application
- SLA-based alerting can target specific business criticality levels

### Exam Decision Points
- Manually created applications = **static** (no auto-update)
- Service Discovery applications = **dynamic** (auto-update based on traffic flows)
- Tiers are **optional** in application definitions
- Business Criticality values: **Low / Medium / Critical** (not High)
- Environment values: **DR / Dev / Prod / Staging / Test**

</details>

---

<details markdown="1">
<summary><strong>4.43 — Configure Alerts in VCF Operations</strong></summary>

### Key Concepts

Alerts in VCF Operations are composed of **symptoms** and **recommendations**.

**Alert Definition structure:**
- **Symptoms** — conditions that must be true to trigger the alert (metric threshold, property condition, message event)
- **Recommendations** — actions suggested to the operator when the alert fires
- **Alert criticality** — Info / Warning / Immediate / Critical
- **Subtype** — Application / Compliance / Efficiency / Availability / Performance / Capacity

**Compliance subtype:**
Use the **Compliance** subtype for alerts that contribute to the compliance scoring dashboard. This is the only subtype that feeds into compliance scoring — choosing a different subtype (e.g., Performance) for compliance-related alerts will not appear in compliance scores.

**Symptom-Based Criticality:**
When a single alert definition has multiple symptom conditions at different severity thresholds (e.g., CPU > 80% = Warning, CPU > 95% = Critical), use **Symptom Based criticality** in the alert definition. This allows the alert to dynamically reflect the criticality of the triggered symptom rather than a fixed alert level.

**Wait/Cancel Cycle:**
Controls how long VCF Operations waits before firing or canceling an alert:
- **Wait Cycle** — number of analysis cycles (typically 5-minute intervals) the symptom must be true before the alert fires
- **Cancel Cycle** — number of cycles the symptom must be false before the alert is cancelled

Default recommendation: set Wait and Cancel to **1** cycle for most alerts to avoid both false positives (too low) and delayed detection (too high).

**Navigation:**
`VCF Operations > Configure > Alerts > Alert Definitions > Add`

**Notification configuration:**
Connect VCF Operations to notification systems via:
`Administration > Management > Outbound Settings` — configure SMTP, SNMP trap receivers, ServiceNow, Slack webhook, etc.

`Administration > Alerts > Alert Plugins` — configure which alerts route to which notification endpoints.

### Exam Decision Points
- Compliance scoring = requires **Compliance subtype** in alert definition
- Wait/Cancel Cycle = **1** is the typical recommended starting value
- Symptom-Based criticality = when single alert has **multiple threshold tiers**
- Alert definition = symptoms + recommendations (both required)
- Outbound Settings = where notification plugins (email, SNMP, webhooks) are configured

</details>

---

<details markdown="1">
<summary><strong>4.44 — Scale Fleet Management Lifecycle Components</strong></summary>

### Key Concepts

Fleet Management in VCF 9 handles lifecycle operations (upgrades, patching, configuration) for all VCF components at scale.

**Fleet Management components that can be scaled:**
- VCF Operations nodes (add Analytics or Data nodes for larger environments)
- VCF Operations for Logs nodes (add worker nodes for higher log throughput)
- SDDC Manager (HA configuration for SDDC Manager is configurable)

**Scaling VCF Operations:**
`VCF Operations > Administration > Cluster Management > Add Node`

Node types:
- **Data Node** — increases analytics processing capacity
- **Remote Collector Node** — extends data collection to remote sites without routing all traffic through primary cluster

Scaling VCF Operations for Logs:
- Add worker nodes to increase log ingestion capacity
- Workers registered to the master VCF Ops for Logs instance

**When to scale:**
- VCF Operations showing high CPU/memory utilization on the primary analytics node
- Log ingestion rates exceeding VCF Ops for Logs capacity (dropped messages)
- Remote sites with high latency to primary VCF Operations cluster

### Exam Decision Points
- VCF Operations scaling = add **Data Node** (analytics) or **Remote Collector Node** (remote sites)
- VCF Ops for Logs scaling = add **worker nodes**
- Remote Collector = **reduces WAN traffic** between remote site and VCF Operations primary

</details>

---

<details markdown="1">
<summary><strong>4.45 — Upgrade VCF Fleet Components using Fleet Management</strong></summary>

### Key Concepts

Fleet Management provides a centralized upgrade orchestration capability for all VCF components.

**VCF 9 upgrade order (must be followed):**
1. Fleet Management appliance (SDDC Manager)
2. VCF Operations instance
3. Remaining management components (VCF Automation, VCF Ops for Logs, VCF Ops for Networks)
4. SDDC Manager (if separate from Fleet Management)
5. NSX Manager
6. vCenter Server
7. ESXi hosts
8. vSAN (post-ESXi; disk format upgrades)

**Critical: NSX host components upgrade with ESXi:**
NSX VIBs on ESXi hosts are bundled with the ESXi upgrade bundle in the SDDC Manager LCM. They are upgraded as part of the host upgrade — not as a separate NSX-side operation.

**VCF Operations for Logs — cannot be upgraded in-place:**
Unlike VCF Operations and other components, VCF Operations for Logs **must be redeployed** for major version upgrades. There is no in-place upgrade path.

**Fleet Management upgrade path:**
`Fleet Management > Lifecycle > Bundle Management > Import Bundle`
Then: `Upgrade > Select components > Schedule`

**Bundle types:**
- **Update Bundle** — patch/minor update
- **Upgrade Bundle** — major version upgrade (e.g., VCF 8.x → 9.x)

**SDDC Manager backup before upgrade:**
Always take an SDDC Manager backup before any upgrade operation:
`SDDC Manager > Administration > Backup and Restore > Backup Now`

**Health pre-checks:**
Fleet Management runs pre-checks before starting an upgrade. ERROR-level pre-checks block the upgrade. WARNING-level pre-checks are advisory — upgrade can proceed but investigate the warning.

### Exam Decision Points
- Upgrade order: Fleet Mgmt → VCF Operations → other mgmt → SDDC Manager → **NSX → vCenter → ESXi → vSAN**
- VCF Ops for Logs = **redeploy only** (no in-place upgrade)
- NSX host VIBs upgrade with **ESXi host** (not separate NSX operation)
- ERROR pre-check = **blocks upgrade**; WARNING = non-blocking
- Always backup SDDC Manager **before** upgrading

</details>

---

<details markdown="1">
<summary><strong>4.46 — Configure VCF SSO using Identity and Access Management</strong></summary>

### Key Concepts

VCF SSO provides unified authentication across VCF components via the **Identity Broker**, replacing Enhanced Linked Mode (ELM).

**Identity Broker configuration path:**
`VCF Operations > Fleet Management > Identity and Access > Identity Broker`

**Supported Identity Providers (via Identity Broker):**
- Okta
- Ping Identity
- Microsoft Entra ID (Azure AD)
- Microsoft ADFS
- Generic SAML 2.0
- Active Directory via AD/LDAP

**Supported protocols:** SAML, OIDC, LDAP

**User provisioning methods:**
- **SCIM 2.0** (System for Cross-domain Identity Management) — automated provisioning/deprovisioning
- **JIT (Just-in-Time)** — user account created dynamically on first login

**Identity Broker supports one IdP at a time** — you cannot configure multiple simultaneous IdPs.

**VCF SSO scope:**
SSO applies to: vSphere (vCenter), NSX, VCF Operations, VCF Automation.
VCF SSO does **NOT** apply to: SDDC Manager, ESXi hosts.

**Post-SSO role assignment:**
After configuring Identity Broker SSO, users must have service roles assigned in the Provider Management Portal before they can access VCF component consoles:
`Provider Portal > Access Control > Import Users > Assign Role`

**Group-based access:**
Groups can be dynamically created based on SAML assertion attributes on first login (JIT group provisioning). Groups can also be pre-provisioned via SCIM.

### Exam Decision Points
- Identity Broker supports **one IdP at a time**
- SDDC Manager and ESXi = **outside VCF SSO scope**
- SCIM = automated provisioning; JIT = dynamic on-login creation
- Post-SSO: users must have **service roles assigned** in Provider Portal before accessing components
- JIT users: must **log in once** before roles can be assigned (account doesn't exist until first login)
- ELM must be **deactivated before** connecting vCenter to Identity Broker SSO

</details>

---

<details markdown="1">
<summary><strong>4.47 — Monitor Configuration Drift using VCF Operations</strong></summary>

### Key Concepts

VCF Operations provides two drift monitoring mechanisms:

**1. Configuration Templates (vCenter-level drift):**
Templates define the desired configuration state for vCenter objects (clusters, hosts, datastores, networks). VCF Operations compares live configuration against the template and flags deviations as drift.

`VCF Operations > Configure > Configuration > Templates`

Requirements:
- Template must have an **active policy** applied to it — templates without an active policy don't generate drift alerts

**2. vSphere Configuration Profiles (cluster-level drift):**
vSphere Configuration Profiles (managed in vSphere Client) enforce consistent cluster-level configuration (host profiles at cluster scope). VCF Operations monitors for profile drift.

> ⚠️ There is an **8-hour delay** before cluster-level drift from vSphere Configuration Profiles is reflected in VCF Operations. This is not a real-time check.

**Git Integration:**
VCF Operations supports exporting configuration snapshots to Git for version control and drift tracking over time.

Supported Git providers:
- **GitLab**
- **GitHub**

Not supported: Azure DevOps Repos, Bitbucket (as of VCF 9.0 — verify in current docs).

**Configuration path for Git integration:**
`VCF Operations > Administration > Repository Settings > Git Configuration`

**Required permissions for drift monitoring:**
- **Manage** privilege — full read/write to templates and drift remediation
- **View** privilege — read-only visibility into drift status

### Exam Decision Points
- Configuration Templates require an **active policy** to generate drift alerts
- vSphere Configuration Profile drift has **8-hour delay** in VCF Operations
- Git integration supports **GitLab and GitHub** (not Azure DevOps by default)
- Manage vs View = **Manage** required for remediation; **View** for drift visibility only
- Configuration Templates = **vCenter objects**; vSphere Configuration Profiles = **cluster/host level**

</details>

---

*Previous: [Supervisor & VKS — Objectives 4.30–4.34]({% post_url 2026-03-20-vcap-admin-vcf9-s4-supervisor %}) · Next: [NSX — Objectives 4.48–4.60]({% post_url 2026-03-20-vcap-admin-vcf9-s6-nsx %})*
