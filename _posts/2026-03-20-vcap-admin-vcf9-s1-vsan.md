---
layout: post
title: "VCAP-ADMIN VCF 9.0 — vSAN: Objectives 4.1–4.9"
date: 2026-03-20
categories: [certification, vcf, vcap]
tags: [vcap, vcf9, vsan, 3v0-11-26, study-guide]
---

# Section: vSAN — Objectives 4.1–4.9

Part of the [VCAP-ADMIN VCF 9.0 Study Guide series]({% post_url 2026-03-20-vcap-admin-vcf9-study-guide-index %}).

---

<details>
<summary><strong>4.1 — Deploy a vSAN Cluster within a VCF Workload Domain</strong></summary>

### Key Concepts

In VCF 9, vSAN is the default storage platform for workload domains. When deploying a workload domain via SDDC Manager, vSAN is configured as part of the wizard.

**vSAN cluster types in VCF 9:**
- **vSAN OSA (Original Storage Architecture)** — disk groups with cache + capacity tiers
- **vSAN ESA (Express Storage Architecture)** — single-tier NVMe, no disk groups, all-flash required

**Deployment path (SDDC Manager):**
`SDDC Manager > Workload Domains > Add Domain > Configure Cluster > vSAN Storage`

**Key requirements:**
- Minimum 3 hosts for standard vSAN cluster (4 recommended for FTT=1 RAID-5)
- All hosts must have contributed disks that are not partitioned
- VMkernel adapter on dedicated vSAN VLAN (vmk tag: `vsan`)
- vSAN license applied to the cluster

**vSAN OSA disk group structure:**
- 1 cache disk (SSD/NVMe) + 1–7 capacity disks per disk group
- 1–5 disk groups per host
- Cache disk is write-back cache (70% write, 30% read buffer)

**vSAN ESA structure:**
- No disk groups — single pool of NVMe/SSD devices
- Requires NVMe or high-performance SAS/SATA SSDs
- Supports RAID-5/6 at lower overhead than OSA

**SDDC Manager claims disks automatically** when hosts are added to the workload domain if auto-claim is enabled. Otherwise, claim manually via vSphere Client > Cluster > Configure > vSAN > Disk Management.

### Exam Decision Points
- Minimum hosts for vSAN: **3** (OSA and ESA)
- RAID-5 erasure coding requires **4 hosts** minimum
- RAID-6 requires **6 hosts** minimum
- FTT=1 RAID-1 mirroring = **3 hosts** minimum
- ESA: requires **NVMe or equivalent high-perf disks**; no cache/capacity split
- OSA: cache disk must be **faster** than capacity disks; do not mix SSD/HDD in ESA

</details>

---

<details>
<summary><strong>4.2 — Deploy a vSAN Stretched Cluster</strong></summary>

### Key Concepts

A vSAN stretched cluster spans **two geographic sites** (preferred and secondary) with a **witness host** at a third site. It provides an RPO=0 (synchronous replication) active-active storage configuration with automatic failover.

**Topology:**
- **Preferred site** — primary workload site
- **Secondary site** — mirror site, same number of hosts as preferred
- **Witness host** — lightweight VM or physical host at a 3rd site; stores metadata only (no data)

**Requirements:**
- Equal number of data hosts per site (e.g., 3+3)
- Witness host must be reachable from both sites
- Inter-site latency: ≤5ms RTT between data sites; ≤200ms RTT to witness
- vSAN stretched cluster license required
- Dedicated witness traffic vmkernel (separate from vSAN vmkernel)

**Deployment path (SDDC Manager / vSphere Client):**
`vSphere Client > Cluster > Configure > vSAN > Fault Domains > Configure Stretched Cluster`

Specify:
1. Preferred site fault domain (hosts)
2. Secondary site fault domain (hosts)
3. Witness host (select from inventory)

**Storage policy for stretched cluster:**
Use **PFTT=1** (Primary Failures to Tolerate = 1 between sites) with **SFTT=1** (Secondary FTT within each site) for full protection. Default policy `vSAN Default Storage Policy` does not set PFTT — you must create a stretched-cluster aware policy.

**Witness traffic:**
Witness vmkernel (`vmk` tagged `witness`) must be routable to the witness host. Configure via: `Host > Configure > Networking > VMkernel Adapters`

### Exam Decision Points
- Witness host **stores metadata only** — no VM data goes to witness
- Preferred site failure → VMs failover to secondary automatically (if quorum maintained via witness)
- Both sites fail simultaneously → cluster goes offline (no quorum)
- Witness host failure alone → cluster continues but **no tolerance for another failure**
- Inter-site latency must be ≤**5ms** for data sites; witness allows up to **200ms**
- Stretched cluster requires **separate witness vmkernel** adapter

</details>

---

<details>
<summary><strong>4.3 — Configure Non-vSAN Storage within a VCF Workload Domain</strong></summary>

### Key Concepts

VCF supports supplemental storage types alongside vSAN for workload domains. Non-vSAN storage can be used for specific workloads that require it (e.g., shared file storage, FC-attached legacy workloads).

**Supported supplemental storage types in VCF 9:**
- **NFS (v3 and v4.1)** — network file storage, shared across hosts
- **vVols (Virtual Volumes)** — VASA provider-based, per-VM storage objects
- **Fibre Channel (FC)** — block storage via FC fabric; requires FC HBAs and zoning
- **iSCSI** — block storage over IP network

**Configuration path for NFS:**
`vSphere Client > Cluster > Configure > Storage > Datastores > Add Datastore > NFS`

**NFS vmkernel requirements:**
- Dedicated vmkernel for NFS traffic recommended
- MTU 9000 (jumbo frames) recommended for NFS performance

**Fibre Channel:**
- FC datastores appear automatically after zoning and masking at the storage array
- VMFS 6 is the filesystem for FC and iSCSI block LUNs
- Claim via `Hosts > Configure > Storage > Storage Adapters > Rescan`

**vVols:**
- Requires storage vendor VASA provider registered in vCenter
- `vCenter > Configure > Storage Providers > Add` to register VASA provider
- Storage containers and protocol endpoints configure automatically after VASA registration

**Key consideration in VCF:** Non-vSAN supplemental storage is **not managed by SDDC Manager** — it is configured directly in vSphere Client and is out of the LCM (lifecycle management) scope.

### Exam Decision Points
- NFS and FC are **supplemental** — vSAN remains the principal datastore for the workload domain
- vVols require **VASA provider** registered in vCenter
- FC requires **HBAs and SAN zoning** — not software-only
- Supplemental datastores are **not lifecycle managed** by SDDC Manager

</details>

---

<details>
<summary><strong>4.4 — Configure vSAN Cross-Cluster Services / Capacity Sharing</strong></summary>

### Key Concepts

**vSAN HCI Mesh** (Cross-cluster Capacity Sharing) allows a vSAN cluster to mount and consume datastore capacity from a **remote vSAN cluster** over the network, without physically sharing hardware.

**Terminology:**
- **Server cluster** — the cluster that owns and provides vSAN datastore capacity
- **Client cluster** — the cluster that mounts and consumes capacity from the server cluster

**Requirements:**
- Both clusters managed by the same vCenter (or linked vCenter)
- Network connectivity between server and client cluster hosts on the vSAN vmkernel
- vSAN Enterprise license on server cluster

**Configuration path:**
`vSphere Client > Server Cluster > Configure > vSAN > Services > Remote Datastores`
Enable Remote Datastore Access → select the server cluster

On the client cluster side:
`vSphere Client > Client Cluster > Configure > Datastores > Mount Remote Datastore`

**Storage policy on client cluster:**
VMs running on the client cluster and using the remote datastore are governed by the **server cluster's policy capabilities**, not the client cluster's. The vSAN storage policy applied to the VM must be compatible with the server cluster.

**Performance implications:**
- All I/O traverses the network between client and server hosts
- Latency-sensitive workloads should remain on local vSAN
- Intended for less latency-sensitive secondary workloads

### Exam Decision Points
- HCI Mesh is a **read/write** mount (not read-only)
- Server cluster must have **vSAN Enterprise** license
- Network between clusters must support **vSAN vmkernel** traffic
- VM storage policy compatibility is dictated by the **server cluster**
- Not a replacement for stretched cluster — no site-level HA via HCI Mesh

</details>

---

<details>
<summary><strong>4.5 — Configure vSAN Encryption</strong></summary>

### Key Concepts

vSAN supports **Data-at-Rest Encryption (D@RE)** for cluster-wide encryption of all vSAN datastore data. This is distinct from VM-level encryption (objective 4.18).

**Two encryption modes:**
- **vSAN Data-at-Rest Encryption** — encrypts data at the vSAN layer (all objects on vSAN)
- **vSAN Data-in-Transit Encryption** — encrypts replication traffic between vSAN nodes

**Prerequisites:**
- External KMS (KMIP 1.1) **or** Native Key Provider configured in vCenter
- vSphere Enterprise Plus license (for VM encryption) — vSAN encryption uses the same key infrastructure
- All hosts in the cluster must support encryption

**Enabling D@RE:**
`vSphere Client > Cluster > Configure > vSAN > Services > Data-At-Rest Encryption > Edit`

Toggle on Data-at-Rest Encryption. Select the key provider. vCenter will push keys to all hosts. The cluster will **reformat all disks** — this is a **disruptive operation** if data already exists.

> ⚠️ **Enabling encryption on an existing populated cluster requires a disk reformat** — all data must be migrated off first or the cluster must be redeployed.

**Enabling Data-in-Transit Encryption:**
Same path > Data-In-Transit Encryption toggle. This encrypts host-to-host vSAN traffic using AES-256-GCM. Non-disruptive to enable/disable.

**Key rotation:**
`Cluster > Configure > vSAN > Services > Data-At-Rest Encryption > Rekey`
- **Shallow rekey** — replaces KEK only (online, non-disruptive)
- **Deep rekey** — replaces DEK and KEK (disruptive, requires disk reformat)

### Exam Decision Points
- vSAN D@RE is **cluster-wide** — cannot encrypt individual VMs differently via vSAN encryption (use VM encryption for per-VM)
- Enabling D@RE on **existing data** = **disruptive** (data must be migrated off or cluster rebuilt)
- Data-in-Transit encryption = **non-disruptive** to enable
- Requires KMS or Native Key Provider — same infrastructure as VM encryption
- **Shallow rekey** = online; **deep rekey** = disruptive

</details>

---

<details>
<summary><strong>4.6 — Configure vSAN Data Protection</strong></summary>

### Key Concepts

vSAN data protection encompasses snapshot capabilities (native and third-party), backup integration, and file services protection.

**vSAN Snapshots (vSphere Snapshots on vSAN):**
- Standard vSphere snapshots work on vSAN, stored as sparse delta VMDKs
- vSAN ESA introduces **vSAN Native Snapshots** — block-level, space-efficient, better performance than traditional sparse snapshots
- vSAN ESA native snapshots are managed via storage policy

**vSAN Native Snapshot policy capability (ESA only):**
Configure via storage policy: enable `vSAN Snapshot` capability, specify snapshot interval and retention.

`vSphere Client > Home > Policies and Profiles > VM Storage Policies > Create Policy > vSAN Rules > Snapshot`

**Third-party backup integration:**
vSAN supports VADP (vStorage APIs for Data Protection) — VMware's backup API used by products like Veeam, Commvault, Cohesity. VMs on vSAN are backed up at the VMDK level using VADP Changed Block Tracking (CBT).

**vSAN File Services:**
vSAN can expose NFS v3/v4.1 and SMB file shares from the vSAN datastore for use by non-VM workloads.

`vSphere Client > Cluster > Configure > vSAN > Services > File Services`

Requires:
- vSAN Enterprise license
- Dedicated File Service Agent VMs (FAs) deployed per host
- IP addresses for FA VMs on a designated network

### Exam Decision Points
- vSAN ESA Native Snapshots = **block-level**, space-efficient, managed via **storage policy**
- vSAN OSA snapshots = traditional vSphere sparse delta VMDKs
- VADP (CBT) is the standard backup API — vendor-agnostic
- File Services requires **Enterprise license** and FA VMs deployed
- Data protection ≠ encryption — these are independent features

</details>

---

<details>
<summary><strong>4.7 — Perform Day 2 Operations on a vSAN Cluster</strong></summary>

### Key Concepts

Day 2 operations cover the ongoing management tasks after initial vSAN cluster deployment.

**Adding capacity:**
- Add new hosts via SDDC Manager (preferred in VCF) — auto-claims disks
- Add disks to existing hosts: `vSphere Client > Host > Configure > vSAN > Disk Management > Claim Disks`
- Capacity rebalances automatically when new disks are claimed

**Removing a host from a vSAN cluster:**
1. Place host in Maintenance Mode — select **Full Data Migration** (moves all data off)
2. Wait for full data migration to complete (can take hours)
3. Remove host from cluster
4. Remove from vSAN (SDDC Manager > Workload Domain > Remove Host)

**Maintenance Mode data migration options:**
| Option | Data Movement | Use When |
|---|---|---|
| Full Data Migration | All data moved off host | Permanent removal |
| Ensure Accessibility | Minimum data moved for compliance | Short maintenance |
| No Data Migration | No movement | Temporary maintenance with sufficient redundancy |

**Replacing a failed disk:**
1. Failed disk shown in `Cluster > Monitor > vSAN > Physical Disks`
2. Remove disk from disk group: `Configure > vSAN > Disk Management > Remove`
3. Physically replace disk
4. Re-claim the new disk

**Rebalancing:**
`Cluster > Configure > vSAN > Services > Rebalance`
Manual rebalance redistributes data more evenly across disks. Automatic rebalancing triggers when imbalance exceeds 30%.

**vSAN Health:**
`Cluster > Monitor > vSAN > Health` — provides hardware compatibility, network, performance, and data integrity checks. Run health check before and after any maintenance operation.

**Resynchronization monitoring:**
`Cluster > Monitor > vSAN > Resyncing Components` — view active resync tasks and estimated completion.

### Exam Decision Points
- Always use **Full Data Migration** when permanently removing a host
- **Ensure Accessibility** is safe for short-term maintenance if cluster has sufficient redundancy
- **No Data Migration** is risky — only use if you're confident about redundancy
- Rebalancing is **non-disruptive** but generates I/O — avoid during peak hours
- vSAN Health check is the first tool to use for any vSAN issue

</details>

---

<details>
<summary><strong>4.8 — Perform Day 2 Operations on a vSAN Stretched Cluster</strong></summary>

### Key Concepts

Stretched cluster Day 2 operations have additional considerations due to the site topology.

**Adding hosts to a stretched cluster:**
- Add to either preferred or secondary fault domain, maintaining **equal host counts** per site
- `vSphere Client > Cluster > Configure > vSAN > Fault Domains` to assign new hosts

**Witness host replacement:**
If the witness host fails or needs replacement:
1. Deploy a new witness host (OVA from Broadcom support portal or CDN)
2. Configure witness vmkernel with correct routing to both sites
3. `Cluster > Configure > vSAN > Fault Domains > Edit Stretched Cluster` to swap witness

> ⚠️ During witness replacement, the cluster has **no tolerance for a site failure** — plan carefully.

**Maintenance on a site:**
- Place all hosts in one site into Maintenance Mode simultaneously (they must all use **Ensure Accessibility** or **No Data Migration** together — **not Full Data Migration** for stretched clusters)
- Cluster will run from surviving site + witness
- Avoid extended single-site operation

**Storage policy for stretched clusters:**
Ensure VMs use a policy with `PFTT=1` (Primary Failures to Tolerate between sites). Without this, a full site failure will cause data loss.

**Failure scenarios:**
| Failure | Result |
|---|---|
| Preferred site fails | VMs restart on secondary; cluster runs on secondary + witness |
| Secondary site fails | VMs continue on preferred; cluster runs on preferred + witness |
| Witness fails | Cluster continues but no tolerance for site failure |
| Preferred site + witness fail | Cluster goes read-only (no quorum) |
| Both data sites fail | Cluster unavailable |

### Exam Decision Points
- Stretched clusters must have **equal hosts per site**
- Witness replacement = **zero tolerance window** for site failure during procedure
- Do **not** use Full Data Migration in maintenance mode for stretched clusters
- `PFTT=1` in storage policy is required for true site-level HA
- Witness handles **quorum only** — no VM data stored on witness

</details>

---

<details>
<summary><strong>4.9 — Manage vSAN Storage Policies</strong></summary>

### Key Concepts

vSAN storage policies are the primary mechanism to define **reliability, performance, and service level** for VMs and VMDKs on vSAN.

**Policy-Driven Storage:**
Every object on vSAN (VM home, each VMDK, swap file) has a storage policy applied. The default is `vSAN Default Storage Policy`.

**Key policy capabilities:**

| Capability | Description |
|---|---|
| **Failures to Tolerate (FTT)** | How many host/disk failures can be tolerated |
| **RAID type** | RAID-1 (mirroring) or RAID-5/6 (erasure coding) |
| **Number of Disk Stripes** | Stripes per object across capacity disks (performance) |
| **Flash Read Cache Reservation** | % of object size reserved as read cache (OSA only) |
| **Object Space Reservation** | % of space provisioned as thick (0% = thin) |
| **Checksum Disabled** | Disables end-to-end checksum (not recommended) |
| **Force Provisioning** | Provision even if policy cannot be satisfied |
| **IOPS Limit** | Per-disk IOPS cap for QoS |

**FTT and RAID combinations:**

| FTT | RAID | Min Hosts | Space Overhead |
|---|---|---|---|
| 1 | RAID-1 | 3 | 2x |
| 1 | RAID-5 | 4 | 1.33x |
| 2 | RAID-1 | 5 | 3x |
| 2 | RAID-6 | 6 | 1.5x |

**Creating a policy:**
`vSphere Client > Home > Policies and Profiles > VM Storage Policies > Create`

1. Select vSAN rules
2. Configure FTT, RAID type, and other capabilities
3. Check compatibility (which datastores satisfy the policy)
4. Apply to VMs via right-click > VM Policies > Edit VM Storage Policy

**Checking compliance:**
`Cluster > Monitor > vSAN > Compliance` — shows which VMs/objects are compliant, non-compliant, or out of date.

**Policy non-compliance:**
Occurs when the cluster cannot satisfy the policy (e.g., not enough hosts for FTT=2). The object continues to run but with reduced redundancy. Resolve by adding hosts, adjusting policy, or fixing failed hardware.

**Applying policies in bulk:**
`Home > Policies and Profiles > VM Storage Policies > [Policy] > VMs and Templates` tab to see all objects using a policy. Use PowerCLI for bulk reassignment:

```powershell
$policy = Get-SpbmStoragePolicy -Name "vSAN FTT=1 RAID-5"
Get-VM | Set-SpbmEntityConfiguration -StoragePolicy $policy
```

### Exam Decision Points
- **FTT=1 RAID-5 requires 4 hosts** (not 3)
- **RAID-6 requires 6 hosts minimum**
- Force Provisioning = VMs provision even if non-compliant (use cautiously)
- Object Space Reservation 0% = thin provisioned; 100% = thick
- Flash Read Cache Reservation is **OSA only** — no equivalent in ESA
- Always check **Compliance** view after policy changes or host maintenance

</details>

---

*Next: [vSphere Compute, Network & Security — Objectives 4.10–4.18]({% post_url 2026-03-20-vcap-admin-vcf9-s2-vsphere %})*
