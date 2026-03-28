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

<details>
<summary><strong>4.35 — Perform VCF Operations Day 2 Tasks</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>VCF Operations consists of three components:<br />
1. <strong>VCF Operations</strong> (formerly vROps) — analytics, alerting, capacity, cost, policy<br />
2. <strong>VCF Operations for Logs</strong> (formerly vRLI) — centralized log collection and analysis<br />
3. <strong>VCF Operations for Networks</strong> (formerly vRNI) — network visibility and troubleshooting</p>
<p><strong>Centralized Log Collection:</strong><br />
In VCF 9, log forwarding uses the <strong>Cloud Proxy</strong> (not the deprecated Log Forwarding agent). The Cloud Proxy aggregates logs from vSphere, NSX, and other VCF components and forwards them to VCF Operations for Logs.</p>
<p>For NSX syslog to VCF Operations for Logs: configure NSX <strong>Node Profiles</strong> → syslog to the Cloud Proxy IP using the <strong>LI protocol</strong> (port 9000 for LI plain, 9543 for LI-TLS). The VCF Operations analytics appliance does not directly receive syslog — the Cloud Proxy/collector handles that.</p>
<p><strong>Linking vCenter to VCF Operations:</strong></p>
<ul>
<li>VCF Operations can link up to <strong>15 vCenter instances</strong></li>
<li>All linked vCenters must share a <strong>common Identity Provider</strong> (VCF SSO or same LDAP)</li>
<li>Deactivate Enhanced Linked Mode (ELM) before connecting vCenter to VCF Operations SSO</li>
<li>Path: <code>VCF Operations &gt; Administration &gt; Cloud Accounts &gt; Add vCenter</code></li>
</ul>
<p><strong>VCF Operations for Logs Day 2:</strong></p>
<ul>
<li>Log sources configured via: <code>VCF Operations for Logs &gt; Administration &gt; Log Sources</code></li>
<li>Create log sources for ESXi hosts, NSX Manager, vCenter, SDDC Manager</li>
<li>Log filtering and archiving policies configurable per environment</li>
</ul>
<p><strong>Day 2 maintenance tasks:</strong></p>
<ul>
<li>Certificate renewal: VCF Operations certificates managed via Fleet Management</li>
<li>Scale out: add VCF Operations nodes (Analytics or Data nodes) for larger environments</li>
<li>Backup: VCF Operations backup via <code>Administration &gt; Backup and Restore</code></li>
</ul>
<h3>Exam Decision Points</h3>
<ul>
<li>Cloud Proxy = <strong>replaces</strong> Log Forwarding agent for NSX log collection</li>
<li>NSX syslog to VCF Ops for Logs = <strong>LI protocol</strong> via Cloud Proxy (not direct to analytics appliance)</li>
<li>Max vCenters linked to VCF Operations = <strong>15</strong></li>
<li>Linking vCenters requires <strong>common IDP</strong> and <strong>ELM deactivated first</strong></li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.36 — Reclaim Resources using VCF Operations</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>VCF Operations identifies wasted resources across 6 categories, ranked by reclamation value:</p>
<ol>
<li><strong>Powered-off VMs</strong> — highest potential savings; VMs consuming storage but no compute</li>
<li><strong>Idle VMs</strong> — VMs powered on but with negligible CPU/memory/network activity</li>
<li><strong>Snapshots</strong> — old snapshots consuming significant extra storage</li>
<li><strong>Orphaned disks</strong> — VMDKs on datastores with no associated VM</li>
<li><strong>Oversized VMs</strong> — VMs with far more CPU/memory than they actually use</li>
<li><strong>Reclaimable hosts</strong> — hosts that could be decommissioned based on capacity analysis</li>
</ol>
<p><strong>Navigation:</strong><br />
<code>VCF Operations &gt; Optimize &gt; Reclaim</code></p>
<p><strong>Two savings dashboards:</strong></p>
<ul>
<li><strong>Potential Savings</strong> — what could be reclaimed if action is taken</li>
<li><strong>Realized Savings</strong> — what has actually been reclaimed (post-action tracking)</li>
</ul>
<p><strong>Powered-off VM reclamation workflow:</strong><br />
VCF Operations identifies VMs that have been powered off for configurable periods. Review list → take action (delete or archive).</p>
<p><strong>Idle VM detection:</strong><br />
Defined by CPU, memory, disk I/O, and network activity thresholds being below a threshold for a sustained period. Configurable idle thresholds in VCF Operations Optimization settings.</p>
<p><strong>Snapshot reclamation:</strong><br />
Lists snapshots older than a configurable age threshold. VCF Operations cannot delete snapshots directly — you take action in vSphere Client. VCF Operations tracks the delta.</p>
<p><strong>Orphaned disk detection:</strong><br />
VMDKs in datastores that are not associated with any registered VM. Must verify before deleting — some intentionally orphaned disks (e.g., detached RDMs) may be in use by non-vCenter systems.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Reclamation priority order: <strong>Powered-off &gt; Idle &gt; Snapshots &gt; Orphaned Disks &gt; Oversized &gt; Reclaimable Hosts</strong></li>
<li>VCF Operations identifies; <strong>action is taken in vSphere Client</strong> (VCF Ops doesn't delete directly in most cases)</li>
<li>Realized Savings = <strong>post-action</strong> tracking (not just identified savings)</li>
<li>Orphaned disks: <strong>verify before deleting</strong> — may be intentionally detached</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.37 — Rightsize Workloads using VCF Operations</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Rightsizing adjusts VM resource allocations (CPU and memory) to better match actual workload consumption, eliminating waste and improving density.</p>
<p><strong>Navigation:</strong><br />
<code>VCF Operations &gt; Optimize &gt; Rightsize</code></p>
<p><strong>Rightsizing recommendations:</strong><br />
VCF Operations analyzes historical CPU and memory utilization for each VM and recommends:<br />
- <strong>Rightsize up</strong> — VM is consistently hitting its limits; increase allocation<br />
- <strong>Rightsize down</strong> — VM is consistently far below its allocation; reduce (reclaim capacity)</p>
<p><strong>Rightsizing calculation basis:</strong></p>
<ul>
<li>Uses configurable time window (e.g., last 30 days)</li>
<li>Applies a buffer/headroom percentage to avoid under-provisioning</li>
<li>Considers peak utilization, not just average</li>
</ul>
<p><strong>Applying rightsizing recommendations:</strong><br />
Recommendations are advisory — the administrator reviews and applies changes in vSphere Client. VCF Operations does not directly modify VM configurations (in standard mode).</p>
<p><strong>Rightsizing in context of Operational Intent (link to 4.41):</strong><br />
Operational Intent's <strong>Consolidate</strong> mode aggressively rightsizes and consolidates VMs to reduce host count. <strong>Balance</strong> mode rightsizes to spread load more evenly.</p>
<p><strong>Key consideration:</strong><br />
Rightsizing memory down on a VM with memory limits already set may cause unexpected ballooning. Always review current memory limits before reducing allocation.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Rightsizing recommendations = <strong>advisory</strong>; admin applies in vSphere Client</li>
<li>Recommendations factor in <strong>peak utilization</strong> + headroom buffer (not just average)</li>
<li>Rightsize down = reduce vCPU/vRAM (reclaim capacity)</li>
<li>Rightsize up = increase vCPU/vRAM (prevent contention)</li>
<li>Check for memory limits before rightsizing memory down</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.38 — Perform Capacity Forecasting What-If Scenarios</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>VCF Operations What-If scenarios allow capacity planning simulations without modifying the live environment.</p>
<p><strong>Navigation:</strong><br />
<code>VCF Operations &gt; Plan &gt; What-If Analysis</code></p>
<p><strong>7 What-If scenario types:</strong></p>
<table>
<thead>
<tr>
<th>Scenario Type</th>
<th>What it simulates</th>
</tr>
</thead>
<tbody>
<tr>
<td>Add VMs</td>
<td>Impact of adding a specified number of VMs with defined profiles</td>
</tr>
<tr>
<td>Remove VMs</td>
<td>Effect of decommissioning VMs</td>
</tr>
<tr>
<td>Add Hosts</td>
<td>Available capacity when new hosts are added</td>
</tr>
<tr>
<td>Remove Hosts</td>
<td>Impact of removing hosts (planned decommission or failure)</td>
</tr>
<tr>
<td>Add Datastores</td>
<td>Storage capacity effect</td>
</tr>
<tr>
<td>Remove Datastores</td>
<td>Storage impact of datastore removal</td>
</tr>
<tr>
<td>Migrate Workloads</td>
<td>Moving VMs between clusters/datacenters</td>
</tr>
</tbody>
</table>
<p><strong>Combining scenarios:</strong><br />
Multiple scenarios can be combined — but only if they target the <strong>same object</strong> (e.g., same cluster). You cannot combine scenarios targeting different clusters in a single analysis.</p>
<p><strong>Committed Scenarios:</strong><br />
When you commit a What-If scenario, VCF Operations <strong>reserves the projected capacity</strong> in its planning calculations. This prevents the capacity model from showing free capacity that is already earmarked for a planned project. Committed scenarios appear in the capacity dashboard as planned utilization.</p>
<p><strong>Missing allocation values (overcommit ratios):</strong><br />
If What-If results show empty or missing CPU/memory allocation values, this indicates that <strong>overcommit ratios are not enabled</strong> in the VCF Operations capacity settings. Enable them via <code>Administration &gt; Global Settings &gt; Capacity</code>.</p>
<p><strong>Time-based forecasting:</strong><br />
VCF Operations uses historical trend data to project when capacity will be exhausted (runway analysis). The default projection period is configurable (typically 30/60/90 days).</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Combining scenarios requires <strong>same object</strong> (same cluster, etc.)</li>
<li>Committed Scenarios = <strong>reserve capacity</strong> in planning model</li>
<li>Missing allocation values = <strong>overcommit ratios not enabled</strong> in capacity settings</li>
<li>What-If scenarios are <strong>non-destructive</strong> — no changes to live environment</li>
<li>7 scenario types to know: Add/Remove VMs, Add/Remove Hosts, Add/Remove Datastores, Migrate Workloads</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.39 — Configure Cost Management in VCF Operations</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>VCF Operations Cost Management assigns financial values to virtual infrastructure resources, enabling cost showback and chargeback.</p>
<p><strong>8 cost drivers (cost categories):</strong><br />
1. CPU<br />
2. Memory<br />
3. Storage<br />
4. Network (optional)<br />
5. License costs<br />
6. Power and cooling<br />
7. Facilities (rack space, floor space)<br />
8. Maintenance/support</p>
<p><strong>Two pricing model types:</strong></p>
<table>
<thead>
<tr>
<th>Model</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>Rate-based</td>
<td>Fixed cost per unit ($/vCPU/month, $/GB RAM/month, $/GB storage/month)</td>
</tr>
<tr>
<td>Cost-based</td>
<td>Total infrastructure cost divided proportionally across VMs by consumption</td>
</tr>
</tbody>
</table>
<p><strong>Pricing Cards:</strong><br />
Pricing cards define the cost per resource unit per environment. Multiple pricing cards can exist for different cost models (e.g., Production pricing vs Development pricing).</p>
<p><code>VCF Operations &gt; Plan &gt; Cost &gt; Pricing Cards &gt; Add</code></p>
<p><strong>Deployment modes (mutually exclusive):</strong></p>
<ul>
<li><strong>All Datacenters mode</strong> — one pricing card applies to all datacenters/clusters</li>
<li><strong>Specific Datacenter mode</strong> — different pricing cards for different datacenters</li>
</ul>
<blockquote>
<p>⚠️ All Datacenters and Specific Datacenter modes are <strong>mutually exclusive</strong> — you cannot mix them. Choose one approach and apply consistently.</p>
</blockquote>
<p><strong>Cost recalculation:</strong><br />
VCF Operations recalculates costs every <strong>24 hours</strong>. Cost data is not real-time — there is up to a 24-hour lag after configuration changes before costs reflect accurately.</p>
<p><strong>Business Groups:</strong><br />
Cost data can be segmented by Business Groups (mapped to VCF Operations groups/tags) to provide per-team or per-project cost visibility.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>8 cost drivers — CPU, Memory, Storage, Network, License, Power/Cooling, Facilities, Maintenance</li>
<li>All Datacenters vs Specific Datacenter = <strong>mutually exclusive</strong> — cannot mix</li>
<li>Cost recalculates every <strong>24 hours</strong> (not real-time)</li>
<li>Rate-based = fixed $/unit; Cost-based = total cost divided proportionally</li>
<li>Pricing cards are applied per <strong>deployment mode</strong> choice</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.40 — Create, Update, and Apply a VCF Operations Policy</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>VCF Operations Policies define which alert definitions, symptom definitions, and analysis settings apply to a group of objects. They are the mechanism for customizing monitoring and capacity behavior per object group.</p>
<p><strong>6 configurable policy elements:</strong><br />
1. Alert definitions (which alerts are active)<br />
2. Symptom definitions (which symptoms are monitored)<br />
3. Capacity and time remaining settings<br />
4. Workload automation settings (DRS integration)<br />
5. Custom metrics/super metrics<br />
6. Compliance benchmarks</p>
<p><strong>Navigation:</strong><br />
<code>VCF Operations &gt; Configure &gt; Policies &gt; Policy Library</code></p>
<p><strong>Default Policy:</strong><br />
There is always a <strong>Default Policy</strong> in VCF Operations. It always has the lowest rank (rank D). Every object not explicitly assigned to another policy is governed by the Default Policy. The Default Policy <strong>cannot be deleted</strong>.</p>
<p><strong>Policy ranking:</strong><br />
Policies are ranked. When multiple policies apply to an object (through group membership), the highest-ranked policy wins. Reordering policies requires at least <strong>2 active policies</strong> — you can't reorder with only the Default Policy.</p>
<p><strong>Creating a policy:</strong><br />
1. <code>Policy Library &gt; Add</code><br />
2. Name the policy and set rank<br />
3. Configure elements (inherit from base policy or customize)<br />
4. Apply to object groups</p>
<p><strong>Applying a policy:</strong><br />
Policies are applied to <strong>object groups</strong> (custom groups, vSphere tags groups, applications, etc.). An object not in any policy's scope falls back to the Default Policy.</p>
<p><strong>Updating a policy:</strong><br />
Changes to a policy apply immediately to all objects governed by that policy. No restart or reapplication needed — VCF Operations picks up policy changes in the next analysis cycle.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Default Policy = <strong>always rank D</strong> (lowest); cannot be deleted</li>
<li>Objects not in any policy's scope = <strong>governed by Default Policy</strong></li>
<li>Reorder Policies requires <strong>2+ active policies</strong> (Default alone cannot be reordered)</li>
<li>Policy changes apply <strong>immediately</strong> to governed objects</li>
<li>6 configurable policy elements (know the list)</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.41 — Configure Business Intent and Operational Intent</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p><strong>Operational Intent:</strong><br />
Defines how DRS should manage workload distribution within a cluster. VCF Operations pushes Operational Intent settings to vCenter as DRS configuration.</p>
<p><strong>Three Operational Intent modes:</strong></p>
<table>
<thead>
<tr>
<th>Mode</th>
<th>DRS Behavior</th>
</tr>
</thead>
<tbody>
<tr>
<td>Balance</td>
<td>Distribute workloads evenly across hosts — standard DRS balancing</td>
</tr>
<tr>
<td>Consolidate</td>
<td>Aggressively pack VMs onto fewer hosts (for maximum host power savings)</td>
</tr>
<tr>
<td>Moderate</td>
<td>Balanced approach between Balance and Consolidate</td>
</tr>
</tbody>
</table>
<p><strong>Cluster Headroom Buffer:</strong><br />
Operational Intent includes a configurable headroom buffer — the % of cluster capacity reserved for failover and burst. This feeds into capacity calculations.</p>
<p><code>VCF Operations &gt; Configure &gt; Operational Intent &gt; [Cluster] &gt; Edit</code></p>
<p><strong>Business Intent:</strong><br />
Business Intent applies business context to VMs using <strong>vCenter tags</strong> — but the workflow is indirect:</p>
<ol>
<li>Define tags in <strong>VCF Operations</strong> (not vCenter)</li>
<li>VCF Operations pushes tag definitions <strong>to vCenter</strong></li>
<li>Apply the tags to VMs in <strong>vSphere Client</strong></li>
<li>Reference the tags back in <strong>Business Intent UI</strong> in VCF Operations</li>
</ol>
<blockquote>
<p>⚠️ Tags must be created in VCF Operations first, then applied in vSphere Client. The workflow does NOT start in vCenter.</p>
</blockquote>
<p><strong>WLP Settings permission:</strong><br />
To configure Operational Intent, the VCF Operations user must have <strong>WLP Settings Write</strong> permission (Workload Placement settings). Without this, the Operational Intent configuration is read-only.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Operational Intent modes: <strong>Balance / Consolidate / Moderate</strong></li>
<li>Business Intent tag workflow: <strong>VCF Operations → vCenter → vSphere Client → VCF Operations</strong> (not from vCenter first)</li>
<li>WLP Settings Write permission required to configure Operational Intent</li>
<li>Consolidate mode = <strong>pack VMs onto fewer hosts</strong> (density focus)</li>
<li>Balance mode = <strong>spread across hosts</strong> (performance focus)</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.42 — Create Business Applications in VCF Operations</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Business Applications (or just "Applications" in VCF Operations) group VMs by business function, enabling application-level monitoring, alerting, and cost visibility.</p>
<p><strong>Navigation:</strong><br />
<code>VCF Operations &gt; Environment &gt; Applications &gt; Add</code></p>
<p><strong>Application types by creation method:</strong></p>
<table>
<thead>
<tr>
<th>Method</th>
<th>Type</th>
<th>Behavior</th>
</tr>
</thead>
<tbody>
<tr>
<td>Manually created</td>
<td>Static</td>
<td>Members defined manually; do not change unless edited</td>
</tr>
<tr>
<td>Auto-discovered (Service Discovery)</td>
<td>Dynamic</td>
<td>Members discovered and updated automatically based on traffic flows</td>
</tr>
</tbody>
</table>
<p><strong>Application structure:</strong><br />
Applications can contain <strong>Tiers</strong> (optional). Tiers group VMs by function within the application (e.g., Web Tier, App Tier, DB Tier). Tiers are optional — a single-tier application is valid.</p>
<p><strong>Application metadata:</strong></p>
<table>
<thead>
<tr>
<th>Field</th>
<th>Options</th>
</tr>
</thead>
<tbody>
<tr>
<td>Business Criticality</td>
<td>Low / Medium / Critical</td>
</tr>
<tr>
<td>Environment</td>
<td>DR / Dev / Prod / Staging / Test</td>
</tr>
</tbody>
</table>
<p>These metadata fields feed into alert prioritization and cost segmentation.</p>
<p><strong>Service Discovery (auto-discovery):</strong><br />
Requires VCF Operations for Networks integration. Traffic flows between VMs are analyzed to infer application topology and group VMs automatically into discovered applications.</p>
<p><strong>Why Applications matter:</strong></p>
<ul>
<li>Application-level health views consolidate VM alerts into a single application health score</li>
<li>Cost management can be segmented per application</li>
<li>SLA-based alerting can target specific business criticality levels</li>
</ul>
<h3>Exam Decision Points</h3>
<ul>
<li>Manually created applications = <strong>static</strong> (no auto-update)</li>
<li>Service Discovery applications = <strong>dynamic</strong> (auto-update based on traffic flows)</li>
<li>Tiers are <strong>optional</strong> in application definitions</li>
<li>Business Criticality values: <strong>Low / Medium / Critical</strong> (not High)</li>
<li>Environment values: <strong>DR / Dev / Prod / Staging / Test</strong></li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.43 — Configure Alerts in VCF Operations</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Alerts in VCF Operations are composed of <strong>symptoms</strong> and <strong>recommendations</strong>.</p>
<p><strong>Alert Definition structure:</strong></p>
<ul>
<li><strong>Symptoms</strong> — conditions that must be true to trigger the alert (metric threshold, property condition, message event)</li>
<li><strong>Recommendations</strong> — actions suggested to the operator when the alert fires</li>
<li><strong>Alert criticality</strong> — Info / Warning / Immediate / Critical</li>
<li><strong>Subtype</strong> — Application / Compliance / Efficiency / Availability / Performance / Capacity</li>
</ul>
<p><strong>Compliance subtype:</strong><br />
Use the <strong>Compliance</strong> subtype for alerts that contribute to the compliance scoring dashboard. This is the only subtype that feeds into compliance scoring — choosing a different subtype (e.g., Performance) for compliance-related alerts will not appear in compliance scores.</p>
<p><strong>Symptom-Based Criticality:</strong><br />
When a single alert definition has multiple symptom conditions at different severity thresholds (e.g., CPU &gt; 80% = Warning, CPU &gt; 95% = Critical), use <strong>Symptom Based criticality</strong> in the alert definition. This allows the alert to dynamically reflect the criticality of the triggered symptom rather than a fixed alert level.</p>
<p><strong>Wait/Cancel Cycle:</strong><br />
Controls how long VCF Operations waits before firing or canceling an alert:<br />
- <strong>Wait Cycle</strong> — number of analysis cycles (typically 5-minute intervals) the symptom must be true before the alert fires<br />
- <strong>Cancel Cycle</strong> — number of cycles the symptom must be false before the alert is cancelled</p>
<p>Default recommendation: set Wait and Cancel to <strong>1</strong> cycle for most alerts to avoid both false positives (too low) and delayed detection (too high).</p>
<p><strong>Navigation:</strong><br />
<code>VCF Operations &gt; Configure &gt; Alerts &gt; Alert Definitions &gt; Add</code></p>
<p><strong>Notification configuration:</strong><br />
Connect VCF Operations to notification systems via:<br />
<code>Administration &gt; Management &gt; Outbound Settings</code> — configure SMTP, SNMP trap receivers, ServiceNow, Slack webhook, etc.</p>
<p><code>Administration &gt; Alerts &gt; Alert Plugins</code> — configure which alerts route to which notification endpoints.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Compliance scoring = requires <strong>Compliance subtype</strong> in alert definition</li>
<li>Wait/Cancel Cycle = <strong>1</strong> is the typical recommended starting value</li>
<li>Symptom-Based criticality = when single alert has <strong>multiple threshold tiers</strong></li>
<li>Alert definition = symptoms + recommendations (both required)</li>
<li>Outbound Settings = where notification plugins (email, SNMP, webhooks) are configured</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.44 — Scale Fleet Management Lifecycle Components</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Fleet Management in VCF 9 handles lifecycle operations (upgrades, patching, configuration) for all VCF components at scale.</p>
<p><strong>Fleet Management components that can be scaled:</strong></p>
<ul>
<li>VCF Operations nodes (add Analytics or Data nodes for larger environments)</li>
<li>VCF Operations for Logs nodes (add worker nodes for higher log throughput)</li>
<li>SDDC Manager (HA configuration for SDDC Manager is configurable)</li>
</ul>
<p><strong>Scaling VCF Operations:</strong><br />
<code>VCF Operations &gt; Administration &gt; Cluster Management &gt; Add Node</code></p>
<p>Node types:</p>
<ul>
<li><strong>Data Node</strong> — increases analytics processing capacity</li>
<li><strong>Remote Collector Node</strong> — extends data collection to remote sites without routing all traffic through primary cluster</li>
</ul>
<p>Scaling VCF Operations for Logs:</p>
<ul>
<li>Add worker nodes to increase log ingestion capacity</li>
<li>Workers registered to the master VCF Ops for Logs instance</li>
</ul>
<p><strong>When to scale:</strong></p>
<ul>
<li>VCF Operations showing high CPU/memory utilization on the primary analytics node</li>
<li>Log ingestion rates exceeding VCF Ops for Logs capacity (dropped messages)</li>
<li>Remote sites with high latency to primary VCF Operations cluster</li>
</ul>
<h3>Exam Decision Points</h3>
<ul>
<li>VCF Operations scaling = add <strong>Data Node</strong> (analytics) or <strong>Remote Collector Node</strong> (remote sites)</li>
<li>VCF Ops for Logs scaling = add <strong>worker nodes</strong></li>
<li>Remote Collector = <strong>reduces WAN traffic</strong> between remote site and VCF Operations primary</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.45 — Upgrade VCF Fleet Components using Fleet Management</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Fleet Management provides a centralized upgrade orchestration capability for all VCF components.</p>
<p><strong>VCF 9 upgrade order (must be followed):</strong><br />
1. Fleet Management appliance (SDDC Manager)<br />
2. VCF Operations instance<br />
3. Remaining management components (VCF Automation, VCF Ops for Logs, VCF Ops for Networks)<br />
4. SDDC Manager (if separate from Fleet Management)<br />
5. NSX Manager<br />
6. vCenter Server<br />
7. ESXi hosts<br />
8. vSAN (post-ESXi; disk format upgrades)</p>
<p><strong>Critical: NSX host components upgrade with ESXi:</strong><br />
NSX VIBs on ESXi hosts are bundled with the ESXi upgrade bundle in the SDDC Manager LCM. They are upgraded as part of the host upgrade — not as a separate NSX-side operation.</p>
<p><strong>VCF Operations for Logs — cannot be upgraded in-place:</strong><br />
Unlike VCF Operations and other components, VCF Operations for Logs <strong>must be redeployed</strong> for major version upgrades. There is no in-place upgrade path.</p>
<p><strong>Fleet Management upgrade path:</strong><br />
<code>Fleet Management &gt; Lifecycle &gt; Bundle Management &gt; Import Bundle</code><br />
Then: <code>Upgrade &gt; Select components &gt; Schedule</code></p>
<p><strong>Bundle types:</strong></p>
<ul>
<li><strong>Update Bundle</strong> — patch/minor update</li>
<li><strong>Upgrade Bundle</strong> — major version upgrade (e.g., VCF 8.x → 9.x)</li>
</ul>
<p><strong>SDDC Manager backup before upgrade:</strong><br />
Always take an SDDC Manager backup before any upgrade operation:<br />
<code>SDDC Manager &gt; Administration &gt; Backup and Restore &gt; Backup Now</code></p>
<p><strong>Health pre-checks:</strong><br />
Fleet Management runs pre-checks before starting an upgrade. ERROR-level pre-checks block the upgrade. WARNING-level pre-checks are advisory — upgrade can proceed but investigate the warning.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Upgrade order: Fleet Mgmt → VCF Operations → other mgmt → SDDC Manager → <strong>NSX → vCenter → ESXi → vSAN</strong></li>
<li>VCF Ops for Logs = <strong>redeploy only</strong> (no in-place upgrade)</li>
<li>NSX host VIBs upgrade with <strong>ESXi host</strong> (not separate NSX operation)</li>
<li>ERROR pre-check = <strong>blocks upgrade</strong>; WARNING = non-blocking</li>
<li>Always backup SDDC Manager <strong>before</strong> upgrading</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.46 — Configure VCF SSO using Identity and Access Management</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>VCF SSO provides unified authentication across VCF components via the <strong>Identity Broker</strong>, replacing Enhanced Linked Mode (ELM).</p>
<p><strong>Identity Broker configuration path:</strong><br />
<code>VCF Operations &gt; Fleet Management &gt; Identity and Access &gt; Identity Broker</code></p>
<p><strong>Supported Identity Providers (via Identity Broker):</strong></p>
<ul>
<li>Okta</li>
<li>Ping Identity</li>
<li>Microsoft Entra ID (Azure AD)</li>
<li>Microsoft ADFS</li>
<li>Generic SAML 2.0</li>
<li>Active Directory via AD/LDAP</li>
</ul>
<p><strong>Supported protocols:</strong> SAML, OIDC, LDAP</p>
<p><strong>User provisioning methods:</strong></p>
<ul>
<li><strong>SCIM 2.0</strong> (System for Cross-domain Identity Management) — automated provisioning/deprovisioning</li>
<li><strong>JIT (Just-in-Time)</strong> — user account created dynamically on first login</li>
</ul>
<p><strong>Identity Broker supports one IdP at a time</strong> — you cannot configure multiple simultaneous IdPs.</p>
<p><strong>VCF SSO scope:</strong><br />
SSO applies to: vSphere (vCenter), NSX, VCF Operations, VCF Automation.<br />
VCF SSO does <strong>NOT</strong> apply to: SDDC Manager, ESXi hosts.</p>
<p><strong>Post-SSO role assignment:</strong><br />
After configuring Identity Broker SSO, users must have service roles assigned in the Provider Management Portal before they can access VCF component consoles:<br />
<code>Provider Portal &gt; Access Control &gt; Import Users &gt; Assign Role</code></p>
<p><strong>Group-based access:</strong><br />
Groups can be dynamically created based on SAML assertion attributes on first login (JIT group provisioning). Groups can also be pre-provisioned via SCIM.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Identity Broker supports <strong>one IdP at a time</strong></li>
<li>SDDC Manager and ESXi = <strong>outside VCF SSO scope</strong></li>
<li>SCIM = automated provisioning; JIT = dynamic on-login creation</li>
<li>Post-SSO: users must have <strong>service roles assigned</strong> in Provider Portal before accessing components</li>
<li>JIT users: must <strong>log in once</strong> before roles can be assigned (account doesn't exist until first login)</li>
<li>ELM must be <strong>deactivated before</strong> connecting vCenter to Identity Broker SSO</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.47 — Monitor Configuration Drift using VCF Operations</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>VCF Operations provides two drift monitoring mechanisms:</p>
<p><strong>1. Configuration Templates (vCenter-level drift):</strong><br />
Templates define the desired configuration state for vCenter objects (clusters, hosts, datastores, networks). VCF Operations compares live configuration against the template and flags deviations as drift.</p>
<p><code>VCF Operations &gt; Configure &gt; Configuration &gt; Templates</code></p>
<p>Requirements:</p>
<ul>
<li>Template must have an <strong>active policy</strong> applied to it — templates without an active policy don't generate drift alerts</li>
</ul>
<p><strong>2. vSphere Configuration Profiles (cluster-level drift):</strong><br />
vSphere Configuration Profiles (managed in vSphere Client) enforce consistent cluster-level configuration (host profiles at cluster scope). VCF Operations monitors for profile drift.</p>
<blockquote>
<p>⚠️ There is an <strong>8-hour delay</strong> before cluster-level drift from vSphere Configuration Profiles is reflected in VCF Operations. This is not a real-time check.</p>
</blockquote>
<p><strong>Git Integration:</strong><br />
VCF Operations supports exporting configuration snapshots to Git for version control and drift tracking over time.</p>
<p>Supported Git providers:</p>
<ul>
<li><strong>GitLab</strong></li>
<li><strong>GitHub</strong></li>
</ul>
<p>Not supported: Azure DevOps Repos, Bitbucket (as of VCF 9.0 — verify in current docs).</p>
<p><strong>Configuration path for Git integration:</strong><br />
<code>VCF Operations &gt; Administration &gt; Repository Settings &gt; Git Configuration</code></p>
<p><strong>Required permissions for drift monitoring:</strong></p>
<ul>
<li><strong>Manage</strong> privilege — full read/write to templates and drift remediation</li>
<li><strong>View</strong> privilege — read-only visibility into drift status</li>
</ul>
<h3>Exam Decision Points</h3>
<ul>
<li>Configuration Templates require an <strong>active policy</strong> to generate drift alerts</li>
<li>vSphere Configuration Profile drift has <strong>8-hour delay</strong> in VCF Operations</li>
<li>Git integration supports <strong>GitLab and GitHub</strong> (not Azure DevOps by default)</li>
<li>Manage vs View = <strong>Manage</strong> required for remediation; <strong>View</strong> for drift visibility only</li>
<li>Configuration Templates = <strong>vCenter objects</strong>; vSphere Configuration Profiles = <strong>cluster/host level</strong></li>
</ul>
</div>
</details>

---

*Previous: [Supervisor & VKS — Objectives 4.30–4.34]({% post_url 2026-03-20-vcap-admin-vcf9-s4-supervisor %}) · Next: [NSX — Objectives 4.48–4.60]({% post_url 2026-03-20-vcap-admin-vcf9-s6-nsx %})*
