---
layout: post
title: "VCAP-ADMIN VCF 9.0 — vSphere Compute, Network & Security: Objectives 4.10–4.18"
date: 2026-03-20
categories: [certification, vcf, vcap]
tags: [vcap, vcf9, vsphere, 3v0-11-26, study-guide]
---

# Section: vSphere Compute, Network & Security — Objectives 4.10–4.18

Part of the [VCAP-ADMIN VCF 9.0 Study Guide series]({% post_url 2026-03-20-vcap-admin-vcf9-study-guide-index %}).

---

<details>
<summary><strong>4.10 — Configure vSphere Advanced Settings within a VCF Workload Domain</strong></summary>

### Key Concepts

Advanced settings in VCF workload domains span vCenter and ESXi host-level parameters. These are configured outside the standard wizard flows — via vSphere Client advanced settings panels or PowerCLI.

**Key vCenter advanced settings:**

| Setting | Purpose | Default |
|---|---|---|
| `VirtualCenter.VimPasswordExpirationInDays` | Controls `vpxuser` password rotation interval on ESXi hosts | 30 days |
| `config.vpxd.stats.maxQueryMetrics` | Limits metrics returned per stats API query (prevents monitoring tool overload) | 64 |
| `config.vpxd.heartbeat.notRespondingTimeout` | How long before vCenter marks a host non-responsive | 60s |
| `VPXD_PersistVnuma` | Retains vNUMA topology across vMotion for consistent NUMA performance | false |

**Path:** `vCenter > Configure > Settings > Advanced Settings`

**Key ESXi advanced settings:**

| Setting | Purpose |
|---|---|
| `DCUI.Access` | Comma-separated list of local users who can access DCUI even in lockdown mode |
| `Syslog.global.logHost` | Syslog target (e.g., `udp://loghost:514`) |
| `UserVars.SuppressShellWarning` | Suppresses SSH shell warning in vSphere Client |
| `Net.TcpipHeapSize` | TCP/IP heap size for VMkernel networking |

**Path:** `vSphere Client > Host > Configure > System > Advanced System Settings`

**Host Profiles in VCF:**
Host Profiles enforce consistent configuration across all hosts in a cluster. In VCF 9, SDDC Manager uses host profiles internally for cluster conformance. You can also create and apply custom host profiles for workload domain clusters.

`vSphere Client > Home > Policies and Profiles > Host Profiles > Extract Profile from Host`

**VirtualCenter.VimPasswordExpirationInDays = 0:**
Setting to 0 disables `vpxuser` password rotation entirely. This is sometimes done in environments where external KMS or security tooling manages credentials externally. Not recommended unless there is an explicit security exception.

### Exam Decision Points
- `VimPasswordExpirationInDays = 0` disables host password rotation — hosts may get disconnected if set to a low value and rotation fails
- `DCUI.Access` can include only **local ESXi users** (not AD users)
- `VPXD_PersistVnuma` is configured at the **cluster level** (Advanced Settings), not per-host
- `maxQueryMetrics` is the go-to setting when monitoring tools are causing vCenter slowness

</details>

---

<details>
<summary><strong>4.11 — Manage Virtual Machines within a VCF Workload Domain</strong></summary>

### Key Concepts

Day-to-day VM management in a VCF workload domain uses standard vSphere mechanisms, but there are VCF-specific considerations around content libraries, templates, and lifecycle.

**VM Templates and Content Libraries:**
- Content Libraries store OVA/OVF templates and ISO images
- Preferred template workflow in VCF: publish OVA to a Content Library, deploy VMs from it (ensures consistent, version-controlled images)
- `vSphere Client > Content Libraries > [Library] > OVF & OVA Templates > Deploy`

**Snapshots:**
- Snapshots capture VM state at a point in time (memory optional)
- On vSAN: snapshots use sparse delta VMDKs (OSA) or native block snapshots (ESA)
- Best practice: limit snapshot chains — multiple snapshots degrade write performance
- Delete snapshots via `VM > Snapshots > Snapshot Manager > Delete All` to consolidate

**Clones:**
- Full clone: independent copy; `Right-click VM > Clone > Clone to Virtual Machine`
- Linked clone: shares base VMDK with parent; smaller disk but parent dependency
- In VCF, prefer deploying from Content Library OVA over manual cloning

**vMotion in VCF:**
- Requires shared storage or vSAN (always shared in VCF workload domains)
- vMotion network must be configured on all hosts (`vmk` tag: `vmotion`)
- Enhanced vMotion: live migrate VM + storage simultaneously across hosts/datastores

**VM Hardware Version:**
VCF 9 / vSphere 9 introduces new VM hardware versions — ensure templates use the correct hardware version for the target vSphere version. Higher hardware versions enable new features (new device types, vNUMA improvements) but reduce compatibility with older vCenter.

### Exam Decision Points
- Content Library OVA deployment = **preferred** approach for consistent VMs in VCF
- Snapshot chains > 3 deep = performance degradation warning
- Linked clones are **not supported** for vSAN in all configurations — use with caution
- vMotion requires vmkernel tagged `vmotion` on all participating hosts
- Hardware version is tied to the vSphere version — do not downgrade hardware version

</details>

---

<details>
<summary><strong>4.12 — Configure Advanced Network Operations within a VCF Workload Domain</strong></summary>

### Key Concepts

"Advanced network operations" in vSphere exam language refers to post-deployment vDS features beyond basic port group and uplink configuration.

**Network I/O Control (NIOC):**
Allocates bandwidth guarantees and shares across traffic types on the vDS.

`vSphere Client > [vDS] > Configure > Resource Allocation > System Traffic`

Traffic categories: vSAN, vMotion, Management, VM traffic, iSCSI, NFS, vSphere Replication, HBR.
- **Shares** — relative weighting during contention
- **Reservation** — guaranteed bandwidth (Mbps) — must not exceed physical capacity
- **Limit** — hard cap (Mbps)

NIOC v3 (vSphere 7+) allows per-VM traffic shaping in addition to system traffic.

**LACP (Link Aggregation Control Protocol):**
Bundles multiple physical NICs into a single logical uplink for higher bandwidth and redundancy.
- In VCF, LACP-enabled vDS **must be created via SDDC Manager API** — not through the GUI wizard
- Post-deployment LACP LAG configuration: `vDS > Configure > Link Aggregation Groups > Add`
- LAG must be added to the host's uplink configuration and associated with a port group teaming policy

**Port Mirroring:**
Mirrors traffic from source ports/VMs to a destination port for monitoring/security tools (e.g., IDS sensors).
`vDS > Configure > Port Mirroring > New Session`

Session types:
- **Distributed Port Mirroring** — source and destination on same vDS
- **Remote Mirroring Source** — sends encapsulated frames to a remote destination
- **Encapsulated Remote Mirroring** — sends to IP destination (RSPAN equivalent)

**Private VLANs (PVLANs):**
Subdivide a VLAN into isolated segments while sharing the same IP subnet.

PVLAN port types:
- **Promiscuous** — can communicate with all ports in the PVLAN (gateway port)
- **Isolated** — can only communicate with promiscuous port; not to other isolated or community
- **Community** — can communicate within its community group and with promiscuous port

`vDS > Configure > Private VLAN > Add` (define primary and secondary PVLANs)

**vDS Health Check:**
Validates physical switch configuration matches vDS expectations.
`vDS > Configure > Health Check > Edit` — enable VLAN/MTU and Teaming/Failover checks.
Identifies MTU mismatches, VLAN mismatches, and teaming policy inconsistencies.

### Exam Decision Points
- LACP in VCF = **must use SDDC Manager API** for workload domain creation
- NIOC reservation must not exceed total physical NIC capacity
- PVLAN **Isolated** ports cannot communicate with each other — only with Promiscuous
- Port Mirroring sessions have a **bandwidth impact** on the source host — monitor overhead
- vDS Health Check requires physical switch to support LLDP or CDP — verify support

</details>

---

<details>
<summary><strong>4.13 — Optimize CPU Performance in a VCF Workload Domain</strong></summary>

### Key Concepts

CPU optimization operates at host, cluster, and VM levels. The primary reference is the **vSphere 9.0 Performance Best Practices** guide.

**NUMA Awareness:**
Modern multi-socket servers have NUMA (Non-Uniform Memory Access) — memory local to a CPU socket is faster than remote memory. ESX NUMA scheduling keeps vCPUs and their memory on the same NUMA node.

- VMs with fewer vCPUs than a single NUMA node = **single NUMA node scheduling** (optimal)
- VMs wider than a NUMA node = **Wide VMs** — ESX schedules across nodes but with overhead
- Keep VMs at or below physical core count per NUMA node where possible

**vNUMA (Virtual NUMA):**
Exposes NUMA topology to the guest OS so NUMA-aware applications (SQL Server, Java, etc.) can allocate memory on the correct node.

> ⚠️ **Enabling CPU Hot-Add disables vNUMA** — the guest OS sees a flat UMA topology. This is a common exam gotcha. Hot-Add trades NUMA performance for online flexibility.

**Latency Sensitivity:**
Setting a VM's Latency Sensitivity to **High** gives each vCPU exclusive access to a physical CPU core — no sharing with other VMs or VMkernel threads.

- Requires **full CPU and memory reservations** on the VM
- Achieves near-zero CPU ready time
- Disables some DRS-based balancing
- Use for: financial trading, real-time analytics, telco NFV workloads

**DRS CPU Optimization:**
- Migration Threshold: 1 (most aggressive) to 5 (conservative) — tune for workload sensitivity
- DRS at level 3 (default) is suitable for most workloads
- Monitor: `%RDY` per VM in esxtop — if consistently > 5%, CPU contention is impacting performance

**Key esxtop CPU counters:**

| Counter | Meaning | Threshold |
|---|---|---|
| `%RDY` | Time vCPU waited for physical CPU | > 5% is concerning |
| `%CSTP` | Co-stop — SMP VMs waiting for all vCPUs | > 3% is concerning |
| `%MLMTD` | CPU limit hit — artificial throttling | Any % = misconfiguration |
| `%USED` | Actual CPU utilization | Monitor trend |

**Host power management:**
Set BIOS to **OS Controlled Mode** so ESX manages CPU power states (C-states and P-states). ESX default policy = Balanced. For latency-sensitive: set to Maximum Performance (disables C-states).

### Exam Decision Points
- Hot-Add enabled → vNUMA disabled → guest sees flat UMA → potential NUMA penalty
- Latency Sensitivity High → requires **full reservations** on CPU and memory
- `%RDY > 5%` = CPU contention; `%CSTP > 3%` = SMP scheduling issue
- `%MLMTD > 0%` = CPU limit is throttling the VM — remove or raise the limit
- Wide VMs (vCPUs > cores per NUMA node) = NUMA penalty — resize or accept overhead

</details>

---

<details>
<summary><strong>4.14 — Optimize Memory Performance in a VCF Workload Domain</strong></summary>

### Key Concepts

vSphere uses a hierarchy of memory reclamation techniques when hosts are under memory pressure. Understanding the order and tradeoffs is essential.

**Memory Reclamation Hierarchy (in order, least to most disruptive):**

1. **Transparent Page Sharing (TPS)** — deduplicates identical memory pages. Since vSphere 6.0, inter-VM TPS is **disabled by default** for security. TPS only occurs within a single VM (intra-VM). Cannot be relied upon for cross-VM memory savings.

2. **Ballooning** — VMware Tools balloon driver inflates inside the guest, pressuring the guest OS to reclaim its own pages (using guest's native memory management). Requires VMware Tools. Performance is close to native under memory pressure.

3. **Memory Compression** — before swapping, ESX compresses pages into a compression cache (2KB compressed = stored in RAM cache instead of swap). Faster than disk swap.

4. **Host-Level Swapping** — VMkernel swaps pages to disk (`.vswp` file on datastore). Worst performance. If using SSD-backed swap cache (`/scratch`), significantly better than HDD swap.

> ⚠️ **Avoid overcommitment that causes regular host swapping** — this is the primary goal of memory management.

**Memory Tiering over NVMe (vSphere 9.0 — New Feature):**
NVMe devices can serve as a second memory tier. Cold pages are staged to NVMe instead of traditional swap.

- Requires high-performance NVMe: ≥100,000 IOPS write, ≥3 DWPD endurance
- Configuration: ESXCLI to designate NVMe device, then enable at cluster level
- Host must be in **Maintenance Mode** to enable/disable/reconfigure tiering
- **Suspend-to-memory is NOT supported** when Memory Tiering is active (suspend-to-disk still works)
- Optane PMem (NVDIMM) is **not supported** in vSphere 9.0 — must be removed

```bash
# Designate NVMe device for memory tiering
esxcli system tierdevice create -d /vmfs/devices/disks/<nvme-device>
```

**Key esxtop memory counters:**

| Counter | Meaning |
|---|---|
| `MCTLSZ` | Balloon driver active size (MB) |
| `SWCUR` | Current swap usage (MB) |
| `ZIP/s` | Pages being compressed per second |
| `UNZIP/s` | Pages decompressed per second |
| `GRANT` | Memory granted to VM |

**Memory Reservations vs Limits vs Shares:**
- **Reservation** — guaranteed memory; retained even if idle (reduces flexibility for DRS)
- **Limit** — hard cap; causes ballooning even if host has free memory (common misconfiguration)
- **Shares** — only matter during contention

> ⚠️ A VM with a memory limit below its configured RAM will experience ballooning even when the host has plenty of free memory. This is a very common scenario question.

### Exam Decision Points
- Inter-VM TPS = **disabled by default** (intra-VM only)
- Ballooning requires **VMware Tools**
- Memory limit below VM's RAM = **ballooning on a healthy host** — remove limit
- Memory Tiering requires **Maintenance Mode** to enable/change
- Suspend-to-memory **not supported** with Memory Tiering enabled
- Optane PMem = **not supported** in vSphere 9.0

</details>

---

<details>
<summary><strong>4.15 — Optimize the Performance of vCenter Server</strong></summary>

### Key Concepts

This objective focuses on vCenter Server itself as a managed component — not the workloads it manages.

**vpxd is the dominant vCenter CPU consumer:**
The `vmware-vpxd` service (vCenter Server daemon) handles all vSphere API requests, DRS calculations, and management operations. All other vCenter services consume minimal CPU by comparison.

If vCenter is slow: check `vpxd` CPU and memory first via `vimtop` on the VCSA:
```bash
# SSH to VCSA, then:
vimtop
```

**Statistics Collection — Key Lever:**

| Collection Interval | Default Retention |
|---|---|
| 5 minutes | 1 day |
| 30 minutes | 1 week |
| 2 hours | 1 month |
| 1 day | 1 year |

All intervals default to **Level 1**. Raising statistics level:
- Level 1: Cluster Services, CPU, Disk, Memory, Network, System, VM Operations
- Level 4: Everything (debugging only — do not leave enabled)
- **Rule:** Level must be ≤ the level of the preceding interval
- **Risk:** Stats levels 3/4 can cause `vpxd` memory growth and eventual crash if it can't persist data fast enough

**Path:** `vCenter > Configure > Settings > General > Statistics`

**vPostgres (embedded database):**
vCenter uses an embedded PostgreSQL instance. Key optimization: ensure fast, low-latency storage for the VCSA. Avoid hosting VCSA on a congested vSAN datastore in small environments.

**Advanced Settings for performance:**

| Setting | Purpose |
|---|---|
| `config.vpxd.stats.maxQueryMetrics` | Limits metrics per API query — set lower if monitoring tools are causing load |
| `config.vpxd.heartbeat.notRespondingTimeout` | Host non-response timeout |

**Periodic network spikes:**
Every 5 minutes, vCenter polls all ESXi hosts for performance stats and writes them to vPostgres — causes predictable network spikes. Design management network accordingly.

**vMotion concurrency:**
Control concurrent migrations: `config.migrate.max` advanced setting. Excessive concurrent vMotions generate significant vCenter management plane load.

**VCSA Management Interface (port 5480):**
Monitor VCSA resource usage, database health, and disk usage at `https://<vcsa-fqdn>:5480`.

### Exam Decision Points
- `vpxd` = **primary CPU consumer** in vCenter — start here for performance issues
- Raising stats above Level 1 = **database growth risk and vpxd crash risk**
- `maxQueryMetrics` = tool to throttle monitoring systems hitting the vCenter API
- 5-minute network spikes = **expected behavior** from stats collection, not a network problem
- vimtop = run on **VCSA SSH session** for per-service resource view

</details>

---

<details>
<summary><strong>4.16 — Monitor vSphere Components within a VCF Workload Domain</strong></summary>

### Key Concepts

**Performance Charts:**
- Real-time data updates every **20 seconds** (pulled live from host, not stored in DB)
- Historical data from vCenter DB at configured statistics intervals
- Advanced charts: any counter at any collection interval
- Cannot sample faster than 20s via performance charts — use esxtop for sub-20s sampling

**esxtop / resxtop modes:**

| Mode | Command | Use Case |
|---|---|---|
| Interactive | `esxtop` | Real-time monitoring on host |
| Batch | `esxtop -b -n 60 -d 2 > output.csv` | Capture data over time to CSV |
| Replay | `esxtop -R output.csv` | Offline analysis of captured batch data |

- resxtop runs **remotely** on a Linux workstation (connects via vSphere API)
- esxtop runs **locally** on the ESXi host via SSH

**Critical esxtop panels and counters:**

| Panel | Key | Critical Counters |
|---|---|---|
| CPU | `c` | `%RDY`, `%CSTP`, `%MLMTD` |
| Memory | `m` | `MCTLSZ` (balloon), `SWCUR` (swap), `ZIP/UNZIP` |
| Disk | `d` | `DAVG` (device latency), `KAVG` (kernel latency), `GAVG` (total) |
| Network | `n` | `%DRPTX/%DRPRX` (drops), `MbTX/MbRX` (throughput) |

**Storage latency thresholds:**
- `DAVG` > 10ms = storage issue for normal workloads
- `DAVG` > 20ms = significant problem

**Batch mode workflow:**
1. Configure columns in interactive mode, save with `W`
2. Run batch: `esxtop -b -n <count> -d <interval> > data.csv`
3. Replay: `esxtop -R data.csv` for offline analysis

**vimtop:**
Monitors vCenter's internal services (vpxd, vpostgres, rhttpproxy).
```bash
# On VCSA via SSH:
vimtop
```

**Alarms:**
- Metric/condition-based (e.g., CPU > 90%) or event-based
- Configure SMTP for email alerts: `Administration > vCenter Server Settings > Mail`
- Configure SNMP: `Administration > vCenter Server Settings > SNMP`
- Best practice: **don't modify built-in alarms** — disable and create custom ones

**Key log file locations:**

| Component | Log Path |
|---|---|
| vCenter vpxd | `/var/log/vmware/vpxd/vpxd.log` |
| ESXi hostd | `/var/log/hostd.log` |
| ESXi vmkernel | `/var/log/vmkernel.log` |
| ESXi vpxa | `/var/log/vpxa.log` |
| Authentication | `/var/log/auth.log` |

**ESXi syslog forwarding:**
`esxcli system syslog config set --loghost=udp://10.10.0.x:514`
Or via vSphere Client: `Host > Configure > System > Advanced System Settings > Syslog.global.logHost`

### Exam Decision Points
- Real-time charts = **20 second** sampling (not configurable lower via charts)
- esxtop batch → save config with `W` in interactive mode first
- resxtop = remote; esxtop = local SSH only
- **Don't modify built-in alarms** — disable and create new
- DAVG > 10ms = investigate storage; > 20ms = problem

</details>

---

<details>
<summary><strong>4.17 — Configure vSphere Security and Access Control in VCF</strong></summary>

### Key Concepts

**VCF 9 SSO Architecture (Big Change):**
Enhanced Linked Mode (ELM) is removed. Unified SSO is now delivered via the **Identity Broker**.

- Identity Broker manages connections between VCF components and an external IdP
- Supported for: vSphere, NSX, VCF Operations, VCF Automation
- **SDDC Manager and ESXi are NOT in the Identity Broker SSO scope**
- Supported protocols: SAML, OIDC, AD/LDAP
- Path: `VCF Operations > Fleet Management > Identity and Access`

**vSphere RBAC Model:**

| Building Block | Definition |
|---|---|
| Privilege | A single action (e.g., `VirtualMachine.Interact.PowerOn`) |
| Role | A named collection of privileges |
| Permission | Role + User/Group bound to an inventory object |

**Key built-in roles:**

| Role | What it can do |
|---|---|
| Administrator | Full access |
| Read-only | View only |
| No access | Cannot see or do anything |
| No Cryptography Administrator | Full admin except crypto operations |

**Permission inheritance:** Propagates down the inventory hierarchy (DC → Cluster → Host → VM). Override by assigning a more restrictive permission at a lower level.

**Global Permissions vs vCenter Permissions:**
- Global Permissions: apply across all vCenters in the SSO domain
- vCenter Permissions: scoped to one vCenter's inventory

**ESXi Lockdown Mode:**

| Feature | Normal Lockdown | Strict Lockdown |
|---|---|---|
| DCUI service | Running | **Stopped** |
| DCUI access | Exception Users + DCUI.Access list | Nobody |
| SSH/Shell | Disabled (unless Exception User with admin) | Same |
| Configure via DCUI | Yes | No — use vSphere Client only |

**Exception Users:** Local ESXi users or AD users with locally defined privileges. **AD group members lose permissions in lockdown mode** — must be individual exception users.

**DCUI.Access:** Advanced setting listing users who can change lockdown mode regardless of privileges. Local users only. Enabling/disabling lockdown via DCUI **discards host permissions** — always use vSphere Client to preserve permissions.

**PowerCLI for permissions:**
```powershell
# Get all roles
Get-VIRole

# Create a new custom role
New-VIRole -Name "VM_Operator" -Privilege (Get-VIPrivilege -Id "VirtualMachine.Interact.PowerOn","VirtualMachine.Interact.PowerOff")

# Assign permission
New-VIPermission -Entity (Get-VM "WebServer01") -Principal "domain\vmoperators" -Role "VM_Operator" -Propagate $true
```

### Exam Decision Points
- VCF SSO (Identity Broker) excludes **SDDC Manager and ESXi**
- AD group members = **lose permissions** in lockdown mode (use individual Exception Users)
- Strict lockdown = **DCUI stopped** — host may require reinstall if vCenter lost with no Exception Users
- Toggle lockdown via **DCUI** = **discards host permissions** — always use vSphere Client
- `DCUI.Access` = local users only; AD users not supported

</details>

---

<details>
<summary><strong>4.18 — Configure VM Encryption and vSphere Trusted Environments</strong></summary>

### Key Concepts

**Three Key Provider Types:**

| Provider | External KMS? | vTPM | VM Encryption | ESXi TPM Req? | Use For |
|---|---|---|---|---|---|
| Native Key Provider (NKP) | No | ✅ All editions | ✅ Enterprise Plus | Preferred, not mandatory | Simple deployments, no KMS |
| Standard Key Provider | Yes (KMIP 1.1) | ✅ | ✅ Enterprise Plus | No | Existing KMS infrastructure |
| Trusted Key Provider (vTA) | Yes (KMIP 1.1) | ✅ | ✅ Enterprise Plus | Yes, on Trusted Hosts | Hardware attestation required |

**KEK / DEK Hierarchy:**
- **KMS → KEK** (Key Encryption Key): AES-256, held by vCenter (ID only, not the key itself)
- **ESXi → DEK** (Data Encryption Key): XTS-AES-256, encrypts VMDK data
- NKP: vCenter generates KDK, pushes to ESXi hosts — no external KMS

**What is encrypted:**
- VM home files: `.vmx`, `.nvram`, `.vswp`
- Virtual disk flat files: `-flat.vmdk`
- Core dumps (if host encryption mode active)

**Not encrypted:** `.log`, `.vmdk` descriptor, `.vmsd` snapshots

**Rekeying:**

| Type | Online? | Snapshots OK? | What changes? |
|---|---|---|---|
| Shallow Recrypt | ✅ Yes | ✅ (single branch) | KEK only |
| Deep Recrypt | ❌ No | ❌ Must remove | KEK + DEK |

**NKP must be backed up immediately after creation** — vCenter raises an alarm and the NKP cannot be used until a backup has been downloaded.

**VM Encryption workflow:**
1. Configure Key Provider (`Home > Key Providers > Add`)
2. Create VM Storage Policy with encryption (`Host Based Rules > Default Encryption Properties`)
3. Apply policy to VM

**vTPM requirements:**
- Key provider configured
- VM must use **EFI firmware** (not BIOS)
- VM must be powered off to add vTPM
- vTPM data stored in `.nvram` file

**vSphere Trust Authority (vTA):**
- **Trust Authority Cluster** — runs Attestation Service and Key Provider Service (management)
- **Trusted Cluster** — workload VMs; **all ESXi hosts require TPM 2.0**
- Attestation: TPM on ESXi host measures software stack → sends to Attestation Service → service verifies against blessed images and known TPM endorsement keys → issues JWT token
- vTA config is in ESXi ConfigStore — **not backed up by vCenter file-based backup**

**No Cryptography Administrator role:** Full vCenter admin privileges but **zero cryptographic operations**. Use to delegate to operators who must never touch encrypted VMs.

### Exam Decision Points
- NKP: backup required **before** it can be used
- vTPM requires **EFI firmware** — BIOS VMs cannot use vTPM
- Deep recrypt: powered off + **no snapshots**
- Shallow recrypt: online + single snapshot branch OK
- vTA: Trusted Cluster hosts **must have physical TPM 2.0**
- vTA config backup: **separate procedure** — NOT covered by standard vCenter backup
- No Cryptography Administrator = delegate admin without crypto access

</details>

---

*Previous: [vSAN — Objectives 4.1–4.9]({% post_url 2026-03-20-vcap-admin-vcf9-s1-vsan %}) · Next: [VCF Automation — Objectives 4.19–4.29]({% post_url 2026-03-20-vcap-admin-vcf9-s3-automation %})*
