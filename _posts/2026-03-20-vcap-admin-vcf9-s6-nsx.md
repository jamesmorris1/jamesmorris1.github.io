---
layout: post
title: "VCAP-ADMIN VCF 9.0 — NSX: Objectives 4.48–4.60"
date: 2026-03-20
categories: [certification, vcf, vcap]
tags: [vcap, vcf9, nsx, 3v0-11-26, study-guide]
---

# Section: NSX — Objectives 4.48–4.60

Part of the [VCAP-ADMIN VCF 9.0 Study Guide series]({% post_url 2026-03-20-vcap-admin-vcf9-study-guide-index %}).

---

<details>
<summary><strong>4.48 — Deploy NSX Federation (Global Manager)</strong></summary>

### Key Concepts

NSX Federation provides centralized multi-site management through a **Global Manager (GM)** that orchestrates multiple Local Managers (LMs) at each site.

**Key architecture facts:**
- Global Manager is **manually deployed** as an OVA — it is **NOT managed by VCF LCM** (SDDC Manager)
- Requires one **active GM** and one **standby GM** (separate compute, separate site)
- Standby GM: maximum **500ms RTT** from active GM
- Standby GM requires its own separate **compute manager** (vCenter) registration
- Active and standby GMs must NOT be managed by the same vCenter

**Adding a location (Local Manager) to Global Manager:**
1. From the GM UI, go to System > Location Manager > Add
2. Exchange thumbprints for trust establishment
3. GM and LM connect via the NSX messaging bus (TCP 443)

**RTEP (Remote Tunnel Endpoint) VLAN:**
For cross-site stretched segments, each site needs a dedicated RTEP VLAN pre-provisioned on the physical fabric. RTEP VLANs must be **configured before adding sites** to NSX Federation — they are not provisioned automatically.

**Multi-tenancy and Federation:**
NSX multi-tenancy (Projects feature, objective 4.58) is **NOT supported** in an NSX Federation environment. If Federation is required, tenancy must be handled differently (e.g., separate LMs per tenant).

**Federation limitations:**
- Stretched segments require RTEP VLANs at each site
- Some NSX features are Federation-aware; others apply per-LM only
- Not all NSX features available in Federation topology (consult compatibility matrix)

### Exam Decision Points
- GM deployed **manually** (OVA) — not by SDDC Manager
- Standby GM: **≤500ms RTT** from active; requires **separate compute manager**
- Active and standby GM: **must NOT share the same vCenter**
- RTEP VLANs: **pre-provisioned** before adding sites to Federation
- NSX multi-tenancy (Projects) = **NOT supported** in Federation

</details>

---

<details>
<summary><strong>4.49 — Configure NSX Components</strong></summary>

### Key Concepts

This objective covers the general configuration of core NSX components that underpin all other NSX objectives.

**NSX Manager deployment in VCF:**
In VCF 9, NSX is deployed via SDDC Manager as part of workload domain creation. NSX Manager is a 3-node cluster (for production) — one per management domain by default, shared across workload domains in that management domain.

**Compute Manager registration:**
NSX must have vCenter registered as a Compute Manager to discover hosts and clusters.
`NSX Manager > System > Fabric > Compute Managers > Add`

**Transport Zones:**
Transport zones define the scope of NSX overlay or VLAN segments.
- **Overlay Transport Zone** — uses GENEVE encapsulation for logical segments
- **VLAN Transport Zone** — for VLAN-backed segments (used for uplinks, TEP VLANs, external access)

**Transport Nodes:**
ESXi hosts are configured as Transport Nodes — they get NSX VIBs installed and TEP vmkernel adapters configured.

In VCF 9, Transport Node configuration is automated by SDDC Manager during workload domain deployment. The VCF automated process handles:
- NSX VIB installation on ESXi hosts
- TEP vmkernel IP assignment
- Uplink profile assignment
- Mapping to transport zones

**Uplink Profiles:**
Define teaming policies and TEP VLAN for transport node communications.
`NSX Manager > System > Fabric > Profiles > Uplink Profiles`

**Host Switch (N-VDS / vDS):**
In VCF 9, NSX uses the vSphere vDS as the host switch (not a separate N-VDS). This simplifies configuration and is the default in VCF 9 deployments.

### Exam Decision Points
- NSX Manager in VCF = deployed by **SDDC Manager** (not manually)
- NSX in VCF 9 uses **vDS as host switch** (not separate N-VDS)
- Compute Manager = **vCenter** registration in NSX for host/cluster discovery
- Transport Zone: **Overlay** (GENEVE/logical) vs **VLAN** (physical VLAN-backed)
- TEP configuration in VCF = automated by SDDC Manager

</details>

---

<details>
<summary><strong>4.50 — Deploy an NSX Edge Cluster</strong></summary>

### Key Concepts

NSX Edge Clusters host the Edge nodes that provide North-South routing services (T0 gateways, NAT, load balancing, VPN, DHCP relay).

**Edge node form factors:**

| Form Factor | vCPU | Memory | Recommended For |
|---|---|---|---|
| Small | 2 | 4GB | Lab/POC only |
| Medium | 4 | 8GB | Small production |
| Large | 8 | 32GB | Production |
| Bare Metal | Physical | Physical | Highest throughput |

> For VCF production deployments, the **Large** form factor is required/recommended. Edge nodes are deployed with **100% memory reservation**.

**Edge TEP network:**
Edge nodes need TEP IP addresses on a dedicated VLAN for GENEVE encapsulation between the edge and ESXi hosts. Plan this VLAN separately from host TEP VLANs.

**Deploying Edge nodes (two methods in VCF 9):**

1. **Via vCenter** (simplified): `vSphere Client > Workload Management > NSX Edge Clusters > Add`
2. **Via NSX Manager**: `NSX > System > Fabric > Nodes > Edge Transport Nodes > Add Edge VM`

**Edge Cluster creation:**
After Edge nodes are deployed, group them into an Edge Cluster:
`NSX Manager > System > Fabric > Nodes > Edge Clusters > Add`

**Uplink profiles for Edge:**
Edge nodes use an uplink profile defining:
- Teaming policy (active/active or active/standby for uplinks)
- TEP VLAN ID
- MTU (9000 recommended for GENEVE)

**Edge Cluster for T0 gateways:**
A T0 gateway is associated with an Edge Cluster. The Edge Cluster provides the transport nodes where T0 services run.

### Exam Decision Points
- Production Edge nodes = **Large** form factor; 100% memory reservation
- Edge TEP VLAN = **separate from host TEP VLAN**
- Edge Cluster = **group of Edge nodes** (not a vSphere cluster)
- T0 gateways require an **Edge Cluster** association
- VCF 9: simplified Edge deployment via **vCenter or NSX Manager**

</details>

---

<details>
<summary><strong>4.51 — Create an NSX Tier-0 Gateway</strong></summary>

### Key Concepts

The Tier-0 (T0) gateway provides North-South routing between the NSX overlay and the physical network (underlay). It peers with physical routers via BGP (or static routing).

**T0 HA modes:**

| HA Mode | Description | Use For |
|---|---|---|
| Active-Active | Multiple Edge nodes forward traffic simultaneously | High throughput; stateless services |
| Active-Standby | One Edge node active, one standby | Stateful services (NAT, LB, VPN, DHCP); HA VIP required |

**HA VIP (Virtual IP):**
In Active-Standby mode, configure an HA VIP on the T0 uplink — external routers BGP-peer with the VIP, ensuring seamless failover without BGP session drop.

**Uplink interfaces:**
T0 gateways have uplink interfaces connecting to the physical network via Edge node uplink ports. Each uplink interface gets an IP address and is associated with an Edge node and segment (VLAN-backed segment on the external network).

**BGP peering:**
T0 gateways peer with physical routers (e.g., your SRX300 at ASN 65000) for dynamic routing.
- T0 ASN configured at T0 gateway level
- BGP neighbors configured with the physical router's IP and ASN
- Route redistribution: which routes to advertise to physical network (connected, static, T1 subnets)

**Creating a T0 gateway:**
`NSX Manager > Networking > Tier-0 Gateways > Add`

Steps:
1. Name, HA mode, Edge Cluster association
2. Add interfaces (uplink IPs)
3. Configure BGP (ASN, neighbors, route redistribution)
4. Configure static routes if needed (alternative to BGP)

**Lab reference:**
- T0 ASN: 65001
- Physical router (SRX300): ASN 65000
- BGP peer: SRX300 interface IP

### Exam Decision Points
- Active-Active = **stateless** (no NAT, LB, VPN on Active-Active T0)
- Active-Standby = **stateful services** require this mode; needs HA VIP
- HA VIP = configured on **T0 uplink** for seamless BGP failover
- T0 connects to physical network via **VLAN-backed segments** on Edge uplinks
- BGP requires ASN on T0 and physical router; must configure route redistribution

</details>

---

<details>
<summary><strong>4.52 — Configure NSX VRF on a Tier-0 Gateway</strong></summary>

### Key Concepts

NSX VRF (Virtual Routing and Forwarding) provides **data-plane routing isolation** between tenants by creating separate routing table instances on a shared T0 gateway.

**VRF vs separate T0:**
- VRF on T0: shares Edge nodes with parent T0 but has **independent routing table**
- Separate T0: separate Edge cluster, separate BGP sessions, full isolation (more resource-intensive)
- VRF is the preferred approach for tenant isolation when Edges are shared

**VRF Gateway:**
In NSX, a VRF is implemented as a **VRF Gateway** — a logical construct attached to a parent T0 gateway. Each VRF Gateway has:
- Its own uplink interfaces (can use the same Edge but different logical interfaces/VLANs)
- Its own BGP configuration (separate ASN or route targets)
- Its own routing table (no route leaking between VRFs unless explicitly configured)

**Creating a VRF Gateway:**
`NSX Manager > Networking > Tier-0 Gateways > Add > Gateway Type: VRF`

Select the **parent T0 gateway** during creation.

**Use cases:**
- Multi-tenant environments where different tenants need isolated routing (e.g., NSX Projects each get a VRF)
- Separating management, production, and development routing on shared infrastructure
- Connecting to different BGP peers per tenant without separate Edge clusters

**Route leaking:**
VRF-to-VRF route leaking can be configured if specific routes need to be shared between VRFs (e.g., shared services VRF). This is explicit and controlled — no accidental cross-tenant leaking.

### Exam Decision Points
- VRF = **data-plane isolation** (separate routing table) on a shared T0
- VRF is attached to a **parent T0** — not standalone
- VRF = preferred for **tenant routing isolation** when sharing Edge infrastructure
- Route leaking between VRFs = **explicit configuration** (not automatic)
- NSX Projects (4.58) each get their own VRF-like isolation via the project networking model

</details>

---

<details>
<summary><strong>4.53 — Create an NSX Logical Segment</strong></summary>

### Key Concepts

NSX Logical Segments are the virtual network switches (Layer 2 domains) for VMs and containers.

**Segment types:**

| Type | Transport Zone | Use For |
|---|---|---|
| Overlay Segment | Overlay Transport Zone | VM workloads, east-west traffic (GENEVE-encapsulated) |
| VLAN Segment | VLAN Transport Zone | Edge uplinks, TEP VLANs, physical-connected workloads |

**Creating an overlay segment:**
`NSX Manager > Networking > Segments > Add Segment`

Key fields:
- **Name**
- **Transport Zone** — select Overlay TZ
- **Gateway** — optionally connect to a T1 gateway (makes it a routed segment)
- **Subnets** — configure the gateway IP for the segment (e.g., 192.168.100.1/24)
- **DHCP** — configure DHCP server, relay, or none

**Segments connected to T1:**
When a segment's gateway is set to a T1, VMs on that segment get routing via the T1. The T1 handles intra-segment and inter-segment routing (east-west).

**Segments connected directly to T0 (less common):**
Possible for specific topologies but less flexible than T1-connected segments. T1 is generally preferred for workload segments.

**Admin state:**
Segments have an Admin State (Up/Down). A Down segment prevents all traffic — useful for planned maintenance without deleting the segment.

**VLANs on VLAN segments:**
VLAN segments use a VLAN ID or range. They connect to physical VLANs on the underlay via Edge node uplinks.

### Exam Decision Points
- Overlay segment = **GENEVE encapsulated**, logical only
- VLAN segment = **physical VLAN backed**, used for Edge uplinks
- Segment connected to T1 = **routed segment** (VMs get a gateway)
- Segment without T1/T0 = **isolated Layer 2** (no routing)
- DHCP on segment = Local, Gateway, or Relay (see 4.56)

</details>

---

<details>
<summary><strong>4.54 — Create an NSX Tier-1 Gateway</strong></summary>

### Key Concepts

Tier-1 (T1) gateways provide east-west routing between segments and connect upward to T0 gateways for north-south access.

**T1 placement:**
T1 gateways run as distributed routing instances on ESXi hosts (distributed routing = no hairpinning through Edge for east-west). Stateful services (NAT, LB, DHCP local) run on Edge nodes.

**T1 HA mode:**
- **Active-Standby** — required for stateful services (NAT, Load Balancer, DHCP, VPN)
- Distributed routing doesn't have HA mode (it's distributed across all hosts)

**Creating a T1 gateway:**
`NSX Manager > Networking > Tier-1 Gateways > Add`

Key fields:
- **Name**
- **Linked T0 Gateway** — connects T1 upward to T0 for north-south access
- **Edge Cluster** — required if using stateful services (NAT, LB, DHCP Local)
- **Route Advertisement** — which routes to advertise to T0 (connected segments, static routes, LB VIPs, NAT IPs)

**Route advertisement (critical):**
T1 gateways do not automatically advertise all routes to T0. You must configure **Route Advertisement** on the T1 to tell T0 about connected segments.

Common route advertisement settings:
- ✅ Connected Interfaces & Segments — advertise directly connected segment subnets
- ✅ NAT IPs — advertise DNAT/SNAT translated IPs
- ✅ LB VIPs — advertise load balancer virtual IPs
- ❌ Static Routes — usually not needed unless specifically required

**Failover (Active-Standby T1):**
`T1 > Edit > Failover mode: Non-Preemptive (default) or Preemptive`
- Non-preemptive: standby takes over on failure but doesn't fail back automatically
- Preemptive: primary reclaims active role after recovery

### Exam Decision Points
- T1 connected to T0 via **linked T0 gateway** setting
- T1 stateful services require **Edge Cluster** association
- Route Advertisement must be **explicitly configured** (not automatic)
- SNAT on T1 to public IP: T1 must advertise NAT IPs to T0, T0 must then advertise to BGP peers
- Active-Standby required for **stateful services** on T1
- Distributed routing = **no Edge hairpin** for east-west (performance benefit)

</details>

---

<details>
<summary><strong>4.55 — Deploy and Manage a VPC in NSX</strong></summary>

### Key Concepts

NSX VPCs (Virtual Private Clouds) provide self-service multi-tenant networking within NSX Projects (tenants). This is the NSX-side view of what objective 4.29 covers from the VCF Automation org perspective.

**VPC subnet types (same as 4.29):**
- **Private – VPC**: VPC-internal only
- **Private – Transit Gateway**: inter-VPC connectivity via TGW
- **Public**: external access

**Creating a VPC (vSphere Client path):**
`vSphere Client > Virtual Private Clouds > [Project] > New VPC`

Or via NSX Manager:
`NSX Manager > [Project space] > VPCs > Add`

**VPC Connectivity Profile:**
Defines the T0/Transit Gateway, external IP blocks, and service configurations. Created at the project or system level.

**Default Outbound NAT:**
When enabled on the VPC connectivity profile, automatically creates an SNAT rule for private subnet egress. Requires N-S Services to be enabled.

**N-S Services (North-South Services):**
Enabling N-S Services on a VPC allows:
- Stateful NAT (SNAT/DNAT)
- N-S Firewall rules
- Gateway QoS

**External IPs (1:1 NAT):**
Assigns a public IP from the External IP block directly to a VM via 1:1 NAT — different from shared SNAT.

**VPC DNS (Service Profile):**
VPCs don't auto-populate DNS — configure DNS servers in the **VPC Service Profile**:
`NSX Manager > [Project] > VPCs > Profiles > VPC Service Profile > Add DNS`

### Exam Decision Points
- Default Outbound NAT = **auto-SNAT** for private egress; requires N-S Services enabled
- External IP = **1:1 NAT** (not shared SNAT)
- VPC DNS = configure in **VPC Service Profile** (not on the VPC directly)
- VPC delete: must have **no associated namespaces**
- N-S Services must be enabled before NAT or firewall rules can be configured on VPC

</details>

---

<details>
<summary><strong>4.56 — Configure DHCP within NSX</strong></summary>

### Key Concepts

NSX supports three DHCP types with specific constraints on where each can be used.

**Three DHCP types:**

| Type | Description | Where Usable |
|---|---|---|
| Local DHCP Server | DHCP served by the NSX gateway | Overlay segments (T1 connected) and isolated segments |
| Gateway DHCP | DHCP served by the T0/T1 gateway service | Overlay segments only; NOT on VLAN segments |
| DHCP Relay | Forwards DHCP requests to external DHCP server | Overlay and VLAN segments |

**Critical constraints:**
- **VLAN segments** = Local DHCP only (Gateway DHCP not supported on VLAN segments)
- **Isolated segments** (no gateway) = Local DHCP only (no T1/T0 to relay or serve gateway DHCP)
- **DHCP Relay** = cannot use static bindings (DHCP relay doesn't manage IP assignments directly)
- **Gateway DHCP** = NOT supported on VLAN-backed segments

**Configuring DHCP on a segment:**
`NSX Manager > Networking > Segments > [Segment] > Edit > DHCP Config`

Select DHCP type:
- **Local**: creates a DHCP server profile, configure IP pool and options
- **Relay**: specify DHCP relay service and target DHCP server IP
- **Gateway**: served by the connected gateway

**DHCP static bindings:**
For Local and Gateway DHCP, you can create static bindings (MAC → IP):
`Segment > DHCP Static Bindings > Add`
Static bindings are **not available** with DHCP Relay.

**SecretExport for cross-namespace DHCP (VKS context):**
When DHCP configuration needs to be shared across namespaces in Supervisor context, use SecretExport (covered in 4.33). This is specific to Kubernetes/Supervisor networking, not standard NSX VM networking.

**DTGW and DHCP Relay:**
Distributed Transit Gateway (DTGW) does **NOT support DHCP Relay**. Use CTGW (Centralised Transit Gateway) if DHCP Relay is required.

### Exam Decision Points
- VLAN segments = **Local DHCP only** (Gateway DHCP not supported)
- Isolated segments = **Local DHCP only**
- DHCP Relay = **no static bindings**
- DTGW = **no DHCP Relay** support
- Gateway DHCP = only for overlay segments connected to a T1/T0

</details>

---

<details>
<summary><strong>4.57 — Configure NAT Use Cases within NSX</strong></summary>

### Key Concepts

NAT in NSX is configured on T0 or T1 gateways. Understanding which NAT type to use for each scenario is critical.

**NAT rule types:**

| Type | Direction | What changes | Use For |
|---|---|---|---|
| SNAT | Outbound | Source IP translated | Internal VMs → internet |
| DNAT | Inbound | Destination IP translated | External → internal published service |
| Reflexive | Bidirectional | 1:1 bidirectional translation | Full bi-directional NAT (replace SNAT+DNAT pair) |
| NO_SNAT | Bypass outbound NAT | Source preserved | Specific traffic that should skip SNAT |
| NO_DNAT | Bypass inbound NAT | Destination preserved | Specific traffic that should skip DNAT |

**Rule priority:**
Lower priority number = higher precedence. NO_SNAT/NO_DNAT rules should have **higher priority** (lower number) than SNAT/DNAT rules to ensure bypass traffic is evaluated first.

**PAT (Port Address Translation):**
When the translated IP range is smaller than the match range, NSX automatically performs PAT — multiple internal IPs map to fewer external IPs using port differentiation.

**NAT on T1 vs T0:**
- NAT on T1: isolated to that T1's scope — preferred for workload-level NAT
- NAT on T0: applies at the border — for network-wide NAT

**SNAT for T1 to public IP:**
If a T1 SNAT rule uses a public IP as the translated address, the T1 must advertise that NAT IP to the T0 (via Route Advertisement), and the T0 must then advertise it via BGP to the physical network. If Route Advertisement for NAT IPs is not enabled, external hosts cannot reach the SNAT'd address.

**Creating a NAT rule:**
`NSX Manager > Networking > [T0 or T1 Gateway] > NAT > Add NAT Rule`

Fields: Action (SNAT/DNAT/etc.), Source Match (IP/range), Destination Match (IP/range), Translated IP, Priority, Enabled.

### Exam Decision Points
- SNAT = **outbound source** translation
- DNAT = **inbound destination** translation
- Reflexive = **bidirectional 1:1** (replaces SNAT+DNAT pair)
- NO_SNAT/NO_DNAT = **bypass** at higher priority (lower number)
- T1 SNAT to public IP: must enable **NAT IP route advertisement** on T1
- PAT: when translated range < match range → **automatic port-based multiplexing**

</details>

---

<details>
<summary><strong>4.58 — Configure NSX Projects and Tenancy</strong></summary>

### Key Concepts

NSX Projects provide the multi-tenancy model within a single NSX Manager, isolating tenant networking without separate NSX deployments.

**NSX Project structure:**
- **Default Space** — the non-tenant space; owns T0 gateways and Edge Clusters
- **Project** — a tenant namespace; gets its own T1 gateways, segments, VPCs, firewalls
- T0 gateways and Edge Clusters remain in the Default Space and are **allocated** to projects
- T1 gateways must be **created within the project** — a T1 in the default space cannot be assigned to a project

**Project creation:**
`NSX Manager > System > Projects > Add`

Key configuration:
- Name and description
- T0 gateway allocation (which T0 is available to this project)
- Edge Cluster allocation
- Resource limits (quotas for segments, T1 gateways, VMs, etc.)

**Resource shares:**
When resources (T0, Edge Cluster) are allocated to a project, they appear as **read-only resource shares** within the project scope. Project users can use them but cannot modify the underlying configuration.

**Quota management:**
Quotas limit how many NSX objects a project can create (segments, T1s, NAT rules, firewall rules, etc.).
- **Manage** privilege = set and modify quotas
- **View** privilege = read quota status only

**Project-scoped operations:**
All networking within a project (T1 gateways, segments, NAT, firewall policies, VPCs) is isolated to that project. The default space administrator can see all projects but project admins only see their own project.

**Federation and Projects:**
NSX multi-tenancy (Projects) is **NOT supported** in an NSX Federation environment. You cannot combine Federation with Projects.

**VPCs and Projects:**
VPCs can only be created **within an NSX Project** — VPCs do not exist in the Default Space.

### Exam Decision Points
- T0 and Edge Clusters = **owned by Default Space**, allocated (not moved) to projects
- T1 must be **created within the project** (cannot assign existing default-space T1)
- Resource shares = **read-only** to project users
- VPCs exist **only within projects** (not in Default Space)
- Projects NOT supported in **NSX Federation** environments
- Manage privilege = **quota management**; View = read-only quota visibility

</details>

---

<details>
<summary><strong>4.59 — Configure Advanced NSX Integrations</strong></summary>

### Key Concepts

This objective covers VPN, Load Balancing, EVPN, Antrea, and vDefend integrations.

**IPSec VPN:**
NSX supports policy-based and route-based IPSec VPN.

| Type | Description | Use For |
|---|---|---|
| Policy-Based | Traffic matched by policy (source/dest subnets) | Simple site-to-site with defined subnet pairs |
| Route-Based | Uses virtual tunnel interface (VTI), routing protocols supported | Dynamic routing, multiple subnets, BGP over VPN |

**L2 VPN:**
Stretches a Layer 2 segment between sites over an IP tunnel. **Requires IPSec service to be configured first** — L2 VPN runs over the IPSec service.

**NSX Load Balancer:**
- Runs on **T1 gateways only** (not T0)
- Requires Edge Cluster (stateful service)
- Supports Layer 4 (TCP/UDP) and Layer 7 (HTTP/HTTPS) load balancing
- Virtual Servers, Server Pools, and Health Monitors are the three configuration objects

**EVPN (Ethernet VPN) — BGP-based L2 extension:**

| Mode | Description |
|---|---|
| Route-Server Mode | NSX acts as EVPN route server; physical ToR switches are EVPN clients |
| Tunnel Mode | NSX creates VXLAN tunnels to remote VTEP (physical or virtual) |

**Antrea CNI Integration:**
Antrea is the Container Network Interface (CNI) for Kubernetes clusters running on vSphere/Supervisor. NSX integrates with Antrea to extend NSX network policies to Kubernetes pods.

Key point: Antrea runs inside VKS clusters and connects to NSX for policy enforcement. The integration uses NSX network policies that are reflected into Antrea's policy engine.

**vDefend (formerly NSX Firewall with Advanced Threat Prevention):**
- **IDS/IPS** — signature-based intrusion detection/prevention at the gateway or distributed firewall
- **NTA (Network Traffic Analysis)** — behavioral analysis to detect anomalies
- **Malware Prevention** — file inspection at the gateway

**NSX + VCF Operations integration:**
NSX registers with VCF Operations using a **Principal Identity certificate** — not username/password authentication. The principal identity is created in NSX and used to authenticate API calls from VCF Operations.

### Exam Decision Points
- L2 VPN requires **IPSec service configured first**
- NSX Load Balancer = **T1 only** (not T0)
- EVPN Route-Server = NSX as BGP route reflector for ToR switches
- Antrea integration = NSX policies extend to **Kubernetes pod level**
- NSX + VCF Operations integration = **principal identity certificate** (not credentials)
- Route-Based IPSec = supports BGP over the tunnel (dynamic routing)

</details>

---

<details>
<summary><strong>4.60 — Perform NSX Operational Tasks (Syslog, Backup)</strong></summary>

### Key Concepts

**NSX Syslog Configuration:**
NSX syslog is configured via **Node Profiles** — which apply to all NSX nodes at once, rather than configuring each manager individually.

**Path:**
`NSX Manager > System > Fabric > Nodes > All NSX Nodes > Edit > Syslog Servers > Add`

Or via Node Profiles:
`NSX Manager > System > Fabric > Profiles > Node Profiles > [Profile] > Syslog`

**Syslog protocols supported:**
- TCP
- UDP
- LI (Log Insight protocol — used for VCF Operations for Logs Cloud Proxy)
- LI-TLS (encrypted LI)

**Syslog log levels (inclusive downward):**
Emergency → Alert → Critical → Error → Warning → Notice → Info → **Debug** (most verbose)

Setting the log level to `Warning` means you receive Warning, Notice, Info, and Debug messages (all levels equal to or less critical). Wait — this is inverted: setting Warning means you receive Warning **and all more severe** levels (Emergency, Alert, Critical, Error, Warning). Less severe levels (Notice, Info, Debug) are not received.

> Correct behavior: each level includes itself and all **more severe** levels above it.

**NSX Backup:**

| Parameter | Requirement |
|---|---|
| Protocol | **SFTP only** |
| Passphrase | **Required** — used to encrypt backup; needed for restore |
| Frequency | Configurable (on-demand, scheduled) |
| In VCF 9 | Managed via **Fleet Management > SFTP Settings** |

**Path (NSX Manager):**
`NSX Manager > System > Backup & Restore > Backup > Configure`

**Restore procedure — critical sequence:**
1. Power off all **surviving NSX Manager nodes** before starting restore
2. Restore the primary NSX Manager from backup (using SFTP + passphrase)
3. NSX Manager rebuilds its configuration from the backup

> ⚠️ **DNS configuration is NOT restored** from NSX backup — must be manually reconfigured after restore.

> ⚠️ **Power off surviving nodes before restore** — failure to do this causes split-brain.

**VCF 9 NSX backup management:**
In VCF 9, NSX backup SFTP settings are configured via **Fleet Management** (not directly in NSX Manager for VCF-managed deployments).
`Fleet Management > Administration > SFTP Settings`

### Exam Decision Points
- Syslog via Node Profiles = applies to **all NSX nodes** simultaneously
- LI protocol = for **VCF Operations for Logs** / Log Insight integration
- NSX backup = **SFTP only** (no NFS, CIFS, etc.)
- Backup passphrase = **required** for restore; store it securely
- Before NSX restore: **power off surviving Manager nodes**
- DNS = **NOT restored** from NSX backup — reconfigure manually
- VCF 9: NSX backup SFTP configured via **Fleet Management**

</details>

---

*Previous: [VCF Operations & Fleet — Objectives 4.35–4.47]({% post_url 2026-03-20-vcap-admin-vcf9-s5-operations %}) · Back to [Study Guide Index]({% post_url 2026-03-20-vcap-admin-vcf9-study-guide-index %})*
