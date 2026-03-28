---
layout: post
title: "VCAP-ADMIN VCF 9.0 — vSphere Compute, Network & Security: Objectives 4.10–4.18"
date: 2026-03-20
categories: [certification, vcf, vcap]
tags: [vcap, vcf9, vsphere, 3v0-11-26, study-guide]
series: "VCAP-ADMIN VCF 9.0 Study Guide (3V0-11.26)"
series_part: 2
---

# Section: vSphere Compute, Network & Security — Objectives 4.10–4.18

Part of the [VCAP-ADMIN VCF 9.0 Study Guide series]({% post_url 2026-03-20-vcap-admin-vcf9-study-guide-index %}).

---

<details>
<summary><strong>4.10 — Configure vSphere Advanced Settings within a VCF Workload Domain</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Advanced settings in VCF workload domains span vCenter and ESXi host-level parameters. These are configured outside the standard wizard flows — via vSphere Client advanced settings panels or PowerCLI.</p>
<p><strong>Key vCenter advanced settings:</strong></p>
<table>
<thead>
<tr>
<th>Setting</th>
<th>Purpose</th>
<th>Default</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>VirtualCenter.VimPasswordExpirationInDays</code></td>
<td>Controls <code>vpxuser</code> password rotation interval on ESXi hosts</td>
<td>30 days</td>
</tr>
<tr>
<td><code>config.vpxd.stats.maxQueryMetrics</code></td>
<td>Limits metrics returned per stats API query (prevents monitoring tool overload)</td>
<td>64</td>
</tr>
<tr>
<td><code>config.vpxd.heartbeat.notRespondingTimeout</code></td>
<td>How long before vCenter marks a host non-responsive</td>
<td>60s</td>
</tr>
<tr>
<td><code>VPXD_PersistVnuma</code></td>
<td>Retains vNUMA topology across vMotion for consistent NUMA performance</td>
<td>false</td>
</tr>
</tbody>
</table>
<p><strong>Path:</strong> <code>vCenter &gt; Configure &gt; Settings &gt; Advanced Settings</code></p>
<p><strong>Key ESXi advanced settings:</strong></p>
<table>
<thead>
<tr>
<th>Setting</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>DCUI.Access</code></td>
<td>Comma-separated list of local users who can access DCUI even in lockdown mode</td>
</tr>
<tr>
<td><code>Syslog.global.logHost</code></td>
<td>Syslog target (e.g., <code>udp://loghost:514</code>)</td>
</tr>
<tr>
<td><code>UserVars.SuppressShellWarning</code></td>
<td>Suppresses SSH shell warning in vSphere Client</td>
</tr>
<tr>
<td><code>Net.TcpipHeapSize</code></td>
<td>TCP/IP heap size for VMkernel networking</td>
</tr>
</tbody>
</table>
<p><strong>Path:</strong> <code>vSphere Client &gt; Host &gt; Configure &gt; System &gt; Advanced System Settings</code></p>
<p><strong>Host Profiles in VCF:</strong><br />
Host Profiles enforce consistent configuration across all hosts in a cluster. In VCF 9, SDDC Manager uses host profiles internally for cluster conformance. You can also create and apply custom host profiles for workload domain clusters.</p>
<p><code>vSphere Client &gt; Home &gt; Policies and Profiles &gt; Host Profiles &gt; Extract Profile from Host</code></p>
<p><strong>VirtualCenter.VimPasswordExpirationInDays = 0:</strong><br />
Setting to 0 disables <code>vpxuser</code> password rotation entirely. This is sometimes done in environments where external KMS or security tooling manages credentials externally. Not recommended unless there is an explicit security exception.</p>
<h3>Exam Decision Points</h3>
<ul>
<li><code>VimPasswordExpirationInDays = 0</code> disables host password rotation — hosts may get disconnected if set to a low value and rotation fails</li>
<li><code>DCUI.Access</code> can include only <strong>local ESXi users</strong> (not AD users)</li>
<li><code>VPXD_PersistVnuma</code> is configured at the <strong>cluster level</strong> (Advanced Settings), not per-host</li>
<li><code>maxQueryMetrics</code> is the go-to setting when monitoring tools are causing vCenter slowness</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.11 — Manage Virtual Machines within a VCF Workload Domain</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Day-to-day VM management in a VCF workload domain uses standard vSphere mechanisms, but there are VCF-specific considerations around content libraries, templates, and lifecycle.</p>
<p><strong>VM Templates and Content Libraries:</strong></p>
<ul>
<li>Content Libraries store OVA/OVF templates and ISO images</li>
<li>Preferred template workflow in VCF: publish OVA to a Content Library, deploy VMs from it (ensures consistent, version-controlled images)</li>
<li><code>vSphere Client &gt; Content Libraries &gt; [Library] &gt; OVF &amp; OVA Templates &gt; Deploy</code></li>
</ul>
<p><strong>Snapshots:</strong></p>
<ul>
<li>Snapshots capture VM state at a point in time (memory optional)</li>
<li>On vSAN: snapshots use sparse delta VMDKs (OSA) or native block snapshots (ESA)</li>
<li>Best practice: limit snapshot chains — multiple snapshots degrade write performance</li>
<li>Delete snapshots via <code>VM &gt; Snapshots &gt; Snapshot Manager &gt; Delete All</code> to consolidate</li>
</ul>
<p><strong>Clones:</strong></p>
<ul>
<li>Full clone: independent copy; <code>Right-click VM &gt; Clone &gt; Clone to Virtual Machine</code></li>
<li>Linked clone: shares base VMDK with parent; smaller disk but parent dependency</li>
<li>In VCF, prefer deploying from Content Library OVA over manual cloning</li>
</ul>
<p><strong>vMotion in VCF:</strong></p>
<ul>
<li>Requires shared storage or vSAN (always shared in VCF workload domains)</li>
<li>vMotion network must be configured on all hosts (<code>vmk</code> tag: <code>vmotion</code>)</li>
<li>Enhanced vMotion: live migrate VM + storage simultaneously across hosts/datastores</li>
</ul>
<p><strong>VM Hardware Version:</strong><br />
VCF 9 / vSphere 9 introduces new VM hardware versions — ensure templates use the correct hardware version for the target vSphere version. Higher hardware versions enable new features (new device types, vNUMA improvements) but reduce compatibility with older vCenter.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Content Library OVA deployment = <strong>preferred</strong> approach for consistent VMs in VCF</li>
<li>Snapshot chains &gt; 3 deep = performance degradation warning</li>
<li>Linked clones are <strong>not supported</strong> for vSAN in all configurations — use with caution</li>
<li>vMotion requires vmkernel tagged <code>vmotion</code> on all participating hosts</li>
<li>Hardware version is tied to the vSphere version — do not downgrade hardware version</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.12 — Configure Advanced Network Operations within a VCF Workload Domain</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>"Advanced network operations" in vSphere exam language refers to post-deployment vDS features beyond basic port group and uplink configuration.</p>
<p><strong>Network I/O Control (NIOC):</strong><br />
Allocates bandwidth guarantees and shares across traffic types on the vDS.</p>
<p><code>vSphere Client &gt; [vDS] &gt; Configure &gt; Resource Allocation &gt; System Traffic</code></p>
<p>Traffic categories: vSAN, vMotion, Management, VM traffic, iSCSI, NFS, vSphere Replication, HBR.</p>
<ul>
<li><strong>Shares</strong> — relative weighting during contention</li>
<li><strong>Reservation</strong> — guaranteed bandwidth (Mbps) — must not exceed physical capacity</li>
<li><strong>Limit</strong> — hard cap (Mbps)</li>
</ul>
<p>NIOC v3 (vSphere 7+) allows per-VM traffic shaping in addition to system traffic.</p>
<p><strong>LACP (Link Aggregation Control Protocol):</strong><br />
Bundles multiple physical NICs into a single logical uplink for higher bandwidth and redundancy.<br />
- In VCF, LACP-enabled vDS <strong>must be created via SDDC Manager API</strong> — not through the GUI wizard<br />
- Post-deployment LACP LAG configuration: <code>vDS &gt; Configure &gt; Link Aggregation Groups &gt; Add</code><br />
- LAG must be added to the host's uplink configuration and associated with a port group teaming policy</p>
<p><strong>Port Mirroring:</strong><br />
Mirrors traffic from source ports/VMs to a destination port for monitoring/security tools (e.g., IDS sensors).<br />
<code>vDS &gt; Configure &gt; Port Mirroring &gt; New Session</code></p>
<p>Session types:</p>
<ul>
<li><strong>Distributed Port Mirroring</strong> — source and destination on same vDS</li>
<li><strong>Remote Mirroring Source</strong> — sends encapsulated frames to a remote destination</li>
<li><strong>Encapsulated Remote Mirroring</strong> — sends to IP destination (RSPAN equivalent)</li>
</ul>
<p><strong>Private VLANs (PVLANs):</strong><br />
Subdivide a VLAN into isolated segments while sharing the same IP subnet.</p>
<p>PVLAN port types:</p>
<ul>
<li><strong>Promiscuous</strong> — can communicate with all ports in the PVLAN (gateway port)</li>
<li><strong>Isolated</strong> — can only communicate with promiscuous port; not to other isolated or community</li>
<li><strong>Community</strong> — can communicate within its community group and with promiscuous port</li>
</ul>
<p><code>vDS &gt; Configure &gt; Private VLAN &gt; Add</code> (define primary and secondary PVLANs)</p>
<p><strong>vDS Health Check:</strong><br />
Validates physical switch configuration matches vDS expectations.<br />
<code>vDS &gt; Configure &gt; Health Check &gt; Edit</code> — enable VLAN/MTU and Teaming/Failover checks.<br />
Identifies MTU mismatches, VLAN mismatches, and teaming policy inconsistencies.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>LACP in VCF = <strong>must use SDDC Manager API</strong> for workload domain creation</li>
<li>NIOC reservation must not exceed total physical NIC capacity</li>
<li>PVLAN <strong>Isolated</strong> ports cannot communicate with each other — only with Promiscuous</li>
<li>Port Mirroring sessions have a <strong>bandwidth impact</strong> on the source host — monitor overhead</li>
<li>vDS Health Check requires physical switch to support LLDP or CDP — verify support</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.13 — Optimize CPU Performance in a VCF Workload Domain</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>CPU optimization operates at host, cluster, and VM levels. The primary reference is the <strong>vSphere 9.0 Performance Best Practices</strong> guide.</p>
<p><strong>NUMA Awareness:</strong><br />
Modern multi-socket servers have NUMA (Non-Uniform Memory Access) — memory local to a CPU socket is faster than remote memory. ESX NUMA scheduling keeps vCPUs and their memory on the same NUMA node.</p>
<ul>
<li>VMs with fewer vCPUs than a single NUMA node = <strong>single NUMA node scheduling</strong> (optimal)</li>
<li>VMs wider than a NUMA node = <strong>Wide VMs</strong> — ESX schedules across nodes but with overhead</li>
<li>Keep VMs at or below physical core count per NUMA node where possible</li>
</ul>
<p><strong>vNUMA (Virtual NUMA):</strong><br />
Exposes NUMA topology to the guest OS so NUMA-aware applications (SQL Server, Java, etc.) can allocate memory on the correct node.</p>
<blockquote>
<p>⚠️ <strong>Enabling CPU Hot-Add disables vNUMA</strong> — the guest OS sees a flat UMA topology. This is a common exam gotcha. Hot-Add trades NUMA performance for online flexibility.</p>
</blockquote>
<p><strong>Latency Sensitivity:</strong><br />
Setting a VM's Latency Sensitivity to <strong>High</strong> gives each vCPU exclusive access to a physical CPU core — no sharing with other VMs or VMkernel threads.</p>
<ul>
<li>Requires <strong>full CPU and memory reservations</strong> on the VM</li>
<li>Achieves near-zero CPU ready time</li>
<li>Disables some DRS-based balancing</li>
<li>Use for: financial trading, real-time analytics, telco NFV workloads</li>
</ul>
<p><strong>DRS CPU Optimization:</strong></p>
<ul>
<li>Migration Threshold: 1 (most aggressive) to 5 (conservative) — tune for workload sensitivity</li>
<li>DRS at level 3 (default) is suitable for most workloads</li>
<li>Monitor: <code>%RDY</code> per VM in esxtop — if consistently &gt; 5%, CPU contention is impacting performance</li>
</ul>
<p><strong>Key esxtop CPU counters:</strong></p>
<table>
<thead>
<tr>
<th>Counter</th>
<th>Meaning</th>
<th>Threshold</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>%RDY</code></td>
<td>Time vCPU waited for physical CPU</td>
<td>&gt; 5% is concerning</td>
</tr>
<tr>
<td><code>%CSTP</code></td>
<td>Co-stop — SMP VMs waiting for all vCPUs</td>
<td>&gt; 3% is concerning</td>
</tr>
<tr>
<td><code>%MLMTD</code></td>
<td>CPU limit hit — artificial throttling</td>
<td>Any % = misconfiguration</td>
</tr>
<tr>
<td><code>%USED</code></td>
<td>Actual CPU utilization</td>
<td>Monitor trend</td>
</tr>
</tbody>
</table>
<p><strong>Host power management:</strong><br />
Set BIOS to <strong>OS Controlled Mode</strong> so ESX manages CPU power states (C-states and P-states). ESX default policy = Balanced. For latency-sensitive: set to Maximum Performance (disables C-states).</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Hot-Add enabled → vNUMA disabled → guest sees flat UMA → potential NUMA penalty</li>
<li>Latency Sensitivity High → requires <strong>full reservations</strong> on CPU and memory</li>
<li><code>%RDY &gt; 5%</code> = CPU contention; <code>%CSTP &gt; 3%</code> = SMP scheduling issue</li>
<li><code>%MLMTD &gt; 0%</code> = CPU limit is throttling the VM — remove or raise the limit</li>
<li>Wide VMs (vCPUs &gt; cores per NUMA node) = NUMA penalty — resize or accept overhead</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.14 — Optimize Memory Performance in a VCF Workload Domain</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>vSphere uses a hierarchy of memory reclamation techniques when hosts are under memory pressure. Understanding the order and tradeoffs is essential.</p>
<p><strong>Memory Reclamation Hierarchy (in order, least to most disruptive):</strong></p>
<ol>
<li>
<p><strong>Transparent Page Sharing (TPS)</strong> — deduplicates identical memory pages. Since vSphere 6.0, inter-VM TPS is <strong>disabled by default</strong> for security. TPS only occurs within a single VM (intra-VM). Cannot be relied upon for cross-VM memory savings.</p>
</li>
<li>
<p><strong>Ballooning</strong> — VMware Tools balloon driver inflates inside the guest, pressuring the guest OS to reclaim its own pages (using guest's native memory management). Requires VMware Tools. Performance is close to native under memory pressure.</p>
</li>
<li>
<p><strong>Memory Compression</strong> — before swapping, ESX compresses pages into a compression cache (2KB compressed = stored in RAM cache instead of swap). Faster than disk swap.</p>
</li>
<li>
<p><strong>Host-Level Swapping</strong> — VMkernel swaps pages to disk (<code>.vswp</code> file on datastore). Worst performance. If using SSD-backed swap cache (<code>/scratch</code>), significantly better than HDD swap.</p>
</li>
</ol>
<blockquote>
<p>⚠️ <strong>Avoid overcommitment that causes regular host swapping</strong> — this is the primary goal of memory management.</p>
</blockquote>
<p><strong>Memory Tiering over NVMe (vSphere 9.0 — New Feature):</strong><br />
NVMe devices can serve as a second memory tier. Cold pages are staged to NVMe instead of traditional swap.</p>
<ul>
<li>Requires high-performance NVMe: ≥100,000 IOPS write, ≥3 DWPD endurance</li>
<li>Configuration: ESXCLI to designate NVMe device, then enable at cluster level</li>
<li>Host must be in <strong>Maintenance Mode</strong> to enable/disable/reconfigure tiering</li>
<li><strong>Suspend-to-memory is NOT supported</strong> when Memory Tiering is active (suspend-to-disk still works)</li>
<li>Optane PMem (NVDIMM) is <strong>not supported</strong> in vSphere 9.0 — must be removed</li>
</ul>
<pre><code class="language-bash"># Designate NVMe device for memory tiering
esxcli system tierdevice create -d /vmfs/devices/disks/&lt;nvme-device&gt;
</code></pre>
<p><strong>Key esxtop memory counters:</strong></p>
<table>
<thead>
<tr>
<th>Counter</th>
<th>Meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>MCTLSZ</code></td>
<td>Balloon driver active size (MB)</td>
</tr>
<tr>
<td><code>SWCUR</code></td>
<td>Current swap usage (MB)</td>
</tr>
<tr>
<td><code>ZIP/s</code></td>
<td>Pages being compressed per second</td>
</tr>
<tr>
<td><code>UNZIP/s</code></td>
<td>Pages decompressed per second</td>
</tr>
<tr>
<td><code>GRANT</code></td>
<td>Memory granted to VM</td>
</tr>
</tbody>
</table>
<p><strong>Memory Reservations vs Limits vs Shares:</strong></p>
<ul>
<li><strong>Reservation</strong> — guaranteed memory; retained even if idle (reduces flexibility for DRS)</li>
<li><strong>Limit</strong> — hard cap; causes ballooning even if host has free memory (common misconfiguration)</li>
<li><strong>Shares</strong> — only matter during contention</li>
</ul>
<blockquote>
<p>⚠️ A VM with a memory limit below its configured RAM will experience ballooning even when the host has plenty of free memory. This is a very common scenario question.</p>
</blockquote>
<h3>Exam Decision Points</h3>
<ul>
<li>Inter-VM TPS = <strong>disabled by default</strong> (intra-VM only)</li>
<li>Ballooning requires <strong>VMware Tools</strong></li>
<li>Memory limit below VM's RAM = <strong>ballooning on a healthy host</strong> — remove limit</li>
<li>Memory Tiering requires <strong>Maintenance Mode</strong> to enable/change</li>
<li>Suspend-to-memory <strong>not supported</strong> with Memory Tiering enabled</li>
<li>Optane PMem = <strong>not supported</strong> in vSphere 9.0</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.15 — Optimize the Performance of vCenter Server</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>This objective focuses on vCenter Server itself as a managed component — not the workloads it manages.</p>
<p><strong>vpxd is the dominant vCenter CPU consumer:</strong><br />
The <code>vmware-vpxd</code> service (vCenter Server daemon) handles all vSphere API requests, DRS calculations, and management operations. All other vCenter services consume minimal CPU by comparison.</p>
<p>If vCenter is slow: check <code>vpxd</code> CPU and memory first via <code>vimtop</code> on the VCSA:</p>
<pre><code class="language-bash"># SSH to VCSA, then:
vimtop
</code></pre>
<p><strong>Statistics Collection — Key Lever:</strong></p>
<table>
<thead>
<tr>
<th>Collection Interval</th>
<th>Default Retention</th>
</tr>
</thead>
<tbody>
<tr>
<td>5 minutes</td>
<td>1 day</td>
</tr>
<tr>
<td>30 minutes</td>
<td>1 week</td>
</tr>
<tr>
<td>2 hours</td>
<td>1 month</td>
</tr>
<tr>
<td>1 day</td>
<td>1 year</td>
</tr>
</tbody>
</table>
<p>All intervals default to <strong>Level 1</strong>. Raising statistics level:</p>
<ul>
<li>Level 1: Cluster Services, CPU, Disk, Memory, Network, System, VM Operations</li>
<li>Level 4: Everything (debugging only — do not leave enabled)</li>
<li><strong>Rule:</strong> Level must be ≤ the level of the preceding interval</li>
<li><strong>Risk:</strong> Stats levels 3/4 can cause <code>vpxd</code> memory growth and eventual crash if it can't persist data fast enough</li>
</ul>
<p><strong>Path:</strong> <code>vCenter &gt; Configure &gt; Settings &gt; General &gt; Statistics</code></p>
<p><strong>vPostgres (embedded database):</strong><br />
vCenter uses an embedded PostgreSQL instance. Key optimization: ensure fast, low-latency storage for the VCSA. Avoid hosting VCSA on a congested vSAN datastore in small environments.</p>
<p><strong>Advanced Settings for performance:</strong></p>
<table>
<thead>
<tr>
<th>Setting</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>config.vpxd.stats.maxQueryMetrics</code></td>
<td>Limits metrics per API query — set lower if monitoring tools are causing load</td>
</tr>
<tr>
<td><code>config.vpxd.heartbeat.notRespondingTimeout</code></td>
<td>Host non-response timeout</td>
</tr>
</tbody>
</table>
<p><strong>Periodic network spikes:</strong><br />
Every 5 minutes, vCenter polls all ESXi hosts for performance stats and writes them to vPostgres — causes predictable network spikes. Design management network accordingly.</p>
<p><strong>vMotion concurrency:</strong><br />
Control concurrent migrations: <code>config.migrate.max</code> advanced setting. Excessive concurrent vMotions generate significant vCenter management plane load.</p>
<p><strong>VCSA Management Interface (port 5480):</strong><br />
Monitor VCSA resource usage, database health, and disk usage at <code>https://&lt;vcsa-fqdn&gt;:5480</code>.</p>
<h3>Exam Decision Points</h3>
<ul>
<li><code>vpxd</code> = <strong>primary CPU consumer</strong> in vCenter — start here for performance issues</li>
<li>Raising stats above Level 1 = <strong>database growth risk and vpxd crash risk</strong></li>
<li><code>maxQueryMetrics</code> = tool to throttle monitoring systems hitting the vCenter API</li>
<li>5-minute network spikes = <strong>expected behavior</strong> from stats collection, not a network problem</li>
<li>vimtop = run on <strong>VCSA SSH session</strong> for per-service resource view</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.16 — Monitor vSphere Components within a VCF Workload Domain</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p><strong>Performance Charts:</strong></p>
<ul>
<li>Real-time data updates every <strong>20 seconds</strong> (pulled live from host, not stored in DB)</li>
<li>Historical data from vCenter DB at configured statistics intervals</li>
<li>Advanced charts: any counter at any collection interval</li>
<li>Cannot sample faster than 20s via performance charts — use esxtop for sub-20s sampling</li>
</ul>
<p><strong>esxtop / resxtop modes:</strong></p>
<table>
<thead>
<tr>
<th>Mode</th>
<th>Command</th>
<th>Use Case</th>
</tr>
</thead>
<tbody>
<tr>
<td>Interactive</td>
<td><code>esxtop</code></td>
<td>Real-time monitoring on host</td>
</tr>
<tr>
<td>Batch</td>
<td><code>esxtop -b -n 60 -d 2 &gt; output.csv</code></td>
<td>Capture data over time to CSV</td>
</tr>
<tr>
<td>Replay</td>
<td><code>esxtop -R output.csv</code></td>
<td>Offline analysis of captured batch data</td>
</tr>
</tbody>
</table>
<ul>
<li>resxtop runs <strong>remotely</strong> on a Linux workstation (connects via vSphere API)</li>
<li>esxtop runs <strong>locally</strong> on the ESXi host via SSH</li>
</ul>
<p><strong>Critical esxtop panels and counters:</strong></p>
<table>
<thead>
<tr>
<th>Panel</th>
<th>Key</th>
<th>Critical Counters</th>
</tr>
</thead>
<tbody>
<tr>
<td>CPU</td>
<td><code>c</code></td>
<td><code>%RDY</code>, <code>%CSTP</code>, <code>%MLMTD</code></td>
</tr>
<tr>
<td>Memory</td>
<td><code>m</code></td>
<td><code>MCTLSZ</code> (balloon), <code>SWCUR</code> (swap), <code>ZIP/UNZIP</code></td>
</tr>
<tr>
<td>Disk</td>
<td><code>d</code></td>
<td><code>DAVG</code> (device latency), <code>KAVG</code> (kernel latency), <code>GAVG</code> (total)</td>
</tr>
<tr>
<td>Network</td>
<td><code>n</code></td>
<td><code>%DRPTX/%DRPRX</code> (drops), <code>MbTX/MbRX</code> (throughput)</td>
</tr>
</tbody>
</table>
<p><strong>Storage latency thresholds:</strong></p>
<ul>
<li><code>DAVG</code> &gt; 10ms = storage issue for normal workloads</li>
<li><code>DAVG</code> &gt; 20ms = significant problem</li>
</ul>
<p><strong>Batch mode workflow:</strong><br />
1. Configure columns in interactive mode, save with <code>W</code><br />
2. Run batch: <code>esxtop -b -n &lt;count&gt; -d &lt;interval&gt; &gt; data.csv</code><br />
3. Replay: <code>esxtop -R data.csv</code> for offline analysis</p>
<p><strong>vimtop:</strong><br />
Monitors vCenter's internal services (vpxd, vpostgres, rhttpproxy).</p>
<pre><code class="language-bash"># On VCSA via SSH:
vimtop
</code></pre>
<p><strong>Alarms:</strong></p>
<ul>
<li>Metric/condition-based (e.g., CPU &gt; 90%) or event-based</li>
<li>Configure SMTP for email alerts: <code>Administration &gt; vCenter Server Settings &gt; Mail</code></li>
<li>Configure SNMP: <code>Administration &gt; vCenter Server Settings &gt; SNMP</code></li>
<li>Best practice: <strong>don't modify built-in alarms</strong> — disable and create custom ones</li>
</ul>
<p><strong>Key log file locations:</strong></p>
<table>
<thead>
<tr>
<th>Component</th>
<th>Log Path</th>
</tr>
</thead>
<tbody>
<tr>
<td>vCenter vpxd</td>
<td><code>/var/log/vmware/vpxd/vpxd.log</code></td>
</tr>
<tr>
<td>ESXi hostd</td>
<td><code>/var/log/hostd.log</code></td>
</tr>
<tr>
<td>ESXi vmkernel</td>
<td><code>/var/log/vmkernel.log</code></td>
</tr>
<tr>
<td>ESXi vpxa</td>
<td><code>/var/log/vpxa.log</code></td>
</tr>
<tr>
<td>Authentication</td>
<td><code>/var/log/auth.log</code></td>
</tr>
</tbody>
</table>
<p><strong>ESXi syslog forwarding:</strong><br />
<code>esxcli system syslog config set --loghost=udp://10.10.0.x:514</code><br />
Or via vSphere Client: <code>Host &gt; Configure &gt; System &gt; Advanced System Settings &gt; Syslog.global.logHost</code></p>
<h3>Exam Decision Points</h3>
<ul>
<li>Real-time charts = <strong>20 second</strong> sampling (not configurable lower via charts)</li>
<li>esxtop batch → save config with <code>W</code> in interactive mode first</li>
<li>resxtop = remote; esxtop = local SSH only</li>
<li><strong>Don't modify built-in alarms</strong> — disable and create new</li>
<li>DAVG &gt; 10ms = investigate storage; &gt; 20ms = problem</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.17 — Configure vSphere Security and Access Control in VCF</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p><strong>VCF 9 SSO Architecture (Big Change):</strong><br />
Enhanced Linked Mode (ELM) is removed. Unified SSO is now delivered via the <strong>Identity Broker</strong>.</p>
<ul>
<li>Identity Broker manages connections between VCF components and an external IdP</li>
<li>Supported for: vSphere, NSX, VCF Operations, VCF Automation</li>
<li><strong>SDDC Manager and ESXi are NOT in the Identity Broker SSO scope</strong></li>
<li>Supported protocols: SAML, OIDC, AD/LDAP</li>
<li>Path: <code>VCF Operations &gt; Fleet Management &gt; Identity and Access</code></li>
</ul>
<p><strong>vSphere RBAC Model:</strong></p>
<table>
<thead>
<tr>
<th>Building Block</th>
<th>Definition</th>
</tr>
</thead>
<tbody>
<tr>
<td>Privilege</td>
<td>A single action (e.g., <code>VirtualMachine.Interact.PowerOn</code>)</td>
</tr>
<tr>
<td>Role</td>
<td>A named collection of privileges</td>
</tr>
<tr>
<td>Permission</td>
<td>Role + User/Group bound to an inventory object</td>
</tr>
</tbody>
</table>
<p><strong>Key built-in roles:</strong></p>
<table>
<thead>
<tr>
<th>Role</th>
<th>What it can do</th>
</tr>
</thead>
<tbody>
<tr>
<td>Administrator</td>
<td>Full access</td>
</tr>
<tr>
<td>Read-only</td>
<td>View only</td>
</tr>
<tr>
<td>No access</td>
<td>Cannot see or do anything</td>
</tr>
<tr>
<td>No Cryptography Administrator</td>
<td>Full admin except crypto operations</td>
</tr>
</tbody>
</table>
<p><strong>Permission inheritance:</strong> Propagates down the inventory hierarchy (DC → Cluster → Host → VM). Override by assigning a more restrictive permission at a lower level.</p>
<p><strong>Global Permissions vs vCenter Permissions:</strong></p>
<ul>
<li>Global Permissions: apply across all vCenters in the SSO domain</li>
<li>vCenter Permissions: scoped to one vCenter's inventory</li>
</ul>
<p><strong>ESXi Lockdown Mode:</strong></p>
<table>
<thead>
<tr>
<th>Feature</th>
<th>Normal Lockdown</th>
<th>Strict Lockdown</th>
</tr>
</thead>
<tbody>
<tr>
<td>DCUI service</td>
<td>Running</td>
<td><strong>Stopped</strong></td>
</tr>
<tr>
<td>DCUI access</td>
<td>Exception Users + DCUI.Access list</td>
<td>Nobody</td>
</tr>
<tr>
<td>SSH/Shell</td>
<td>Disabled (unless Exception User with admin)</td>
<td>Same</td>
</tr>
<tr>
<td>Configure via DCUI</td>
<td>Yes</td>
<td>No — use vSphere Client only</td>
</tr>
</tbody>
</table>
<p><strong>Exception Users:</strong> Local ESXi users or AD users with locally defined privileges. <strong>AD group members lose permissions in lockdown mode</strong> — must be individual exception users.</p>
<p><strong>DCUI.Access:</strong> Advanced setting listing users who can change lockdown mode regardless of privileges. Local users only. Enabling/disabling lockdown via DCUI <strong>discards host permissions</strong> — always use vSphere Client to preserve permissions.</p>
<p><strong>PowerCLI for permissions:</strong></p>
<pre><code class="language-powershell"># Get all roles
Get-VIRole

# Create a new custom role
New-VIRole -Name &quot;VM_Operator&quot; -Privilege (Get-VIPrivilege -Id &quot;VirtualMachine.Interact.PowerOn&quot;,&quot;VirtualMachine.Interact.PowerOff&quot;)

# Assign permission
New-VIPermission -Entity (Get-VM &quot;WebServer01&quot;) -Principal &quot;domain\vmoperators&quot; -Role &quot;VM_Operator&quot; -Propagate $true
</code></pre>
<h3>Exam Decision Points</h3>
<ul>
<li>VCF SSO (Identity Broker) excludes <strong>SDDC Manager and ESXi</strong></li>
<li>AD group members = <strong>lose permissions</strong> in lockdown mode (use individual Exception Users)</li>
<li>Strict lockdown = <strong>DCUI stopped</strong> — host may require reinstall if vCenter lost with no Exception Users</li>
<li>Toggle lockdown via <strong>DCUI</strong> = <strong>discards host permissions</strong> — always use vSphere Client</li>
<li><code>DCUI.Access</code> = local users only; AD users not supported</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.18 — Configure VM Encryption and vSphere Trusted Environments</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p><strong>Three Key Provider Types:</strong></p>
<table>
<thead>
<tr>
<th>Provider</th>
<th>External KMS?</th>
<th>vTPM</th>
<th>VM Encryption</th>
<th>ESXi TPM Req?</th>
<th>Use For</th>
</tr>
</thead>
<tbody>
<tr>
<td>Native Key Provider (NKP)</td>
<td>No</td>
<td>✅ All editions</td>
<td>✅ Enterprise Plus</td>
<td>Preferred, not mandatory</td>
<td>Simple deployments, no KMS</td>
</tr>
<tr>
<td>Standard Key Provider</td>
<td>Yes (KMIP 1.1)</td>
<td>✅</td>
<td>✅ Enterprise Plus</td>
<td>No</td>
<td>Existing KMS infrastructure</td>
</tr>
<tr>
<td>Trusted Key Provider (vTA)</td>
<td>Yes (KMIP 1.1)</td>
<td>✅</td>
<td>✅ Enterprise Plus</td>
<td>Yes, on Trusted Hosts</td>
<td>Hardware attestation required</td>
</tr>
</tbody>
</table>
<p><strong>KEK / DEK Hierarchy:</strong></p>
<ul>
<li><strong>KMS → KEK</strong> (Key Encryption Key): AES-256, held by vCenter (ID only, not the key itself)</li>
<li><strong>ESXi → DEK</strong> (Data Encryption Key): XTS-AES-256, encrypts VMDK data</li>
<li>NKP: vCenter generates KDK, pushes to ESXi hosts — no external KMS</li>
</ul>
<p><strong>What is encrypted:</strong></p>
<ul>
<li>VM home files: <code>.vmx</code>, <code>.nvram</code>, <code>.vswp</code></li>
<li>Virtual disk flat files: <code>-flat.vmdk</code></li>
<li>Core dumps (if host encryption mode active)</li>
</ul>
<p><strong>Not encrypted:</strong> <code>.log</code>, <code>.vmdk</code> descriptor, <code>.vmsd</code> snapshots</p>
<p><strong>Rekeying:</strong></p>
<table>
<thead>
<tr>
<th>Type</th>
<th>Online?</th>
<th>Snapshots OK?</th>
<th>What changes?</th>
</tr>
</thead>
<tbody>
<tr>
<td>Shallow Recrypt</td>
<td>✅ Yes</td>
<td>✅ (single branch)</td>
<td>KEK only</td>
</tr>
<tr>
<td>Deep Recrypt</td>
<td>❌ No</td>
<td>❌ Must remove</td>
<td>KEK + DEK</td>
</tr>
</tbody>
</table>
<p><strong>NKP must be backed up immediately after creation</strong> — vCenter raises an alarm and the NKP cannot be used until a backup has been downloaded.</p>
<p><strong>VM Encryption workflow:</strong><br />
1. Configure Key Provider (<code>Home &gt; Key Providers &gt; Add</code>)<br />
2. Create VM Storage Policy with encryption (<code>Host Based Rules &gt; Default Encryption Properties</code>)<br />
3. Apply policy to VM</p>
<p><strong>vTPM requirements:</strong></p>
<ul>
<li>Key provider configured</li>
<li>VM must use <strong>EFI firmware</strong> (not BIOS)</li>
<li>VM must be powered off to add vTPM</li>
<li>vTPM data stored in <code>.nvram</code> file</li>
</ul>
<p><strong>vSphere Trust Authority (vTA):</strong></p>
<ul>
<li><strong>Trust Authority Cluster</strong> — runs Attestation Service and Key Provider Service (management)</li>
<li><strong>Trusted Cluster</strong> — workload VMs; <strong>all ESXi hosts require TPM 2.0</strong></li>
<li>Attestation: TPM on ESXi host measures software stack → sends to Attestation Service → service verifies against blessed images and known TPM endorsement keys → issues JWT token</li>
<li>vTA config is in ESXi ConfigStore — <strong>not backed up by vCenter file-based backup</strong></li>
</ul>
<p><strong>No Cryptography Administrator role:</strong> Full vCenter admin privileges but <strong>zero cryptographic operations</strong>. Use to delegate to operators who must never touch encrypted VMs.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>NKP: backup required <strong>before</strong> it can be used</li>
<li>vTPM requires <strong>EFI firmware</strong> — BIOS VMs cannot use vTPM</li>
<li>Deep recrypt: powered off + <strong>no snapshots</strong></li>
<li>Shallow recrypt: online + single snapshot branch OK</li>
<li>vTA: Trusted Cluster hosts <strong>must have physical TPM 2.0</strong></li>
<li>vTA config backup: <strong>separate procedure</strong> — NOT covered by standard vCenter backup</li>
<li>No Cryptography Administrator = delegate admin without crypto access</li>
</ul>
</div>
</details>

---

*Previous: [vSAN — Objectives 4.1–4.9]({% post_url 2026-03-20-vcap-admin-vcf9-s1-vsan %}) · Next: [VCF Automation — Objectives 4.19–4.29]({% post_url 2026-03-20-vcap-admin-vcf9-s3-automation %})*
