---
layout: post
title: "VCAP-ADMIN VCF 9.0 — VCF Automation Provider & Org: Objectives 4.19–4.29"
date: 2026-03-20
categories: [certification, vcf, vcap]
tags: [vcap, vcf9, vcf-automation, 3v0-11-26, study-guide]
---

# Section: VCF Automation — Provider & Organisation Management — Objectives 4.19–4.29

Part of the [VCAP-ADMIN VCF 9.0 Study Guide series]({% post_url 2026-03-20-vcap-admin-vcf9-study-guide-index %}).

---

<details>
<summary><strong>4.19 — Configure the Identity Provider for the Provider Management Portal</strong></summary>

### Key Concepts

The Provider Management Portal (`https://<vcf-automation-fqdn>/provider`) uses its **own** identity provider configuration — separate from the vSphere Identity Broker (objective 4.17).

**Two configuration levels:**

| Level | Scope | Who Configures | What Can Be Configured |
|---|---|---|---|
| System (Provider) | All organisations | Provider/System Admin | LDAP only |
| Organisation | Specific org only | Provider Admin or Org Admin | LDAP (custom), SAML, OIDC |

**Critical exam rule:** You can have **one integration per protocol type** — one LDAP, one SAML, one OIDC simultaneously. You cannot have two LDAP connections at the same level.

**System LDAP (shared across all orgs):**
`Provider Portal > Administration > Identity Providers > LDAP`
Organisations can inherit the system LDAP or override with a private custom LDAP.

**SAML (organisation level):**
Exchange metadata between VCF Automation (SP) and external IdP. VCF Automation extracts group/role attributes from the SAML token.

**OIDC (organisation level):**
Supports Configuration Discovery (well-known endpoint auto-fill). Supports **PKCE** (Proof Key for Code Exchange) for additional auth code security.
`Provider Portal > Administration > Identity Providers > OIDC > Enable PKCE`

**OIDC vs VCF SSO conflict:**
If an OIDC connection is configured for the provider org, you **cannot also configure VCF SSO**. Must remove OIDC first before connecting VCF SSO (and vice versa). When VCF SSO is configured, the OIDC tab is replaced by a VCF SSO tab.

**User remapping:**
When migrating from one IdP to another, use **Remap Users** to preserve roles and resource ownership.
`Provider Portal > Administration > Identity Providers > Remap Users`

**First login:** Default local admin user is `admin` — password set during installation. After 25 minutes of inactivity, session expires (does not interrupt long-running operations).

### Exam Decision Points
- System LDAP = provider-level, **shared** across all orgs
- Custom LDAP = org-level, **private** to that org
- OIDC + VCF SSO = **mutually exclusive** — cannot coexist; remove one before configuring the other
- One integration per protocol type (1 LDAP + 1 SAML + 1 OIDC max per org)
- Remap Users = preserves existing roles/resources during IdP migration

</details>

---

<details>
<summary><strong>4.20 — Create and Manage Provider Content Libraries within the Provider Management Portal</strong></summary>

### Key Concepts

Provider Content Libraries make VM images (OVA/OVF) and ISOs available to organisation tenants for use in blueprints and IaaS provisioning.

**Supported content types:** ISO and OVA/OVF only. **Existing VMs cannot be captured** into a content library — you must export to OVA first.

**Provider Content Library vs Organisation Content Library:**

| Library Type | Creator | Visibility to Orgs |
|---|---|---|
| Provider Content Library | Provider Admin | Read-only |
| Organisation Content Library | Org Admin | Private to that org |

**Creating a Provider Content Library:**
`Provider Portal > Content Libraries > New`
1. Name the library
2. Select backing vCenter content library (must be a **published** vCenter content library)
3. Select regions and storage class per region
4. Select whether to assign to all current/future namespaces — **cannot be changed after creation**

**Subscribed Content Libraries (org-level):**
- Provider must explicitly enable this capability
- Subscribed libraries are **read-only** — cannot add or delete items
- **No automatic or scheduled synchronisation** — manual refresh only
- Sync respects org storage quota

**Supervisor Release Images (VCF 9 — critical new feature):**
Starting in VCF 9, Kubernetes release images for Supervisor are delivered via Broadcom's CDN, not bundled with vCenter.
- Create a subscribed content library pointing to Broadcom CDN
- Immediate or on-demand sync modes
- **Do not delete or edit** a content library assigned with Supervisor release images — unassign first
- Air-gapped: create local publisher → subscribe internally

**API operations:**
```bash
# List accessible content libraries for an org
GET https://<vcfa-fqdn>/cloudapi/vcf/contentLibraries?page=1&pageSize=128

# Refresh a catalog
POST https://<vcfa-fqdn>/cloudapi/1.0.0/catalogs/<urn>/refresh
```

### Exam Decision Points
- Region assignment (namespace assignment) = **cannot be changed after library creation**
- Subscribed org libraries = **manual refresh only** (no auto-sync)
- Existing VMs = **not capturable** directly into content library (must export to OVA)
- Supervisor images = **subscribed CDN library** required in VCF 9 (not bundled with vCenter install)
- Air-gapped Supervisor images = local publisher first, then internal subscribed library

</details>

---

<details>
<summary><strong>4.21 — Configure Access Control Settings within the Provider Management Portal</strong></summary>

### Key Concepts

VCF Automation uses a **three-tier RBAC model**: Provider → Organisation → Project.

**Tier 1 — Provider Level (Global Roles):**
Global roles are created in the Provider Admin Portal and **automatically published to all organisations**.

| Role | Description |
|---|---|
| Defer to Identity Provider | Rights determined by IdP token claims |
| Organisation Administrator | Full org management |
| Organisation Auditor | Read-only org access |
| Organisation User | Standard user rights |
| Custom Role | Created by provider; published selectively |

`Provider Portal > Administration > Access Control > Roles > New`

Create custom roles **before** creating an organisation if you need to assign a custom role to the first org user.

**Tier 2 — Organisation Level:**
Orgs inherit provider global roles. Org admins can create **custom org-level roles**, but these **cannot be shared to other organisations**.
Default inherited roles cannot be edited at the org level.

**Tier 3 — Project Level:**
Users need a project-level role to access resources — org role alone is not enough.

| Project Role | Rights |
|---|---|
| Project Administrator | Manage members and catalog content |
| Project Advanced User | Request catalog items and use IaaS services |
| Project User | Request catalog items only |
| Project Auditor | View all project content, no requests |

**Rights Bundles:**
Extend capabilities of global roles for add-on services (e.g., VCF Operations Orchestrator).
`Provider Portal > Administration > Access Control > Rights Bundles`

To publish Orchestrator to an org:
1. Enable **Advanced Rights Bundle Mode** in Feature Flags
2. Publish Orchestrator Rights Bundle to the org

If a rights bundle belonging to a VCF service is accidentally deleted, VCF Automation **auto-recreates it within minutes**.

**Service Accounts:**
- API-only access; token rotates on **every use** (RFC 6749 section 6)
- Request flow: application requests via device code → admin reviews and grants via `Access Control > Service Accounts > Review Access Requests`
- Revoking a token: terminates all sessions but leaves the service account in "Created" state (not deleted)
- Admins **never see the actual token** — only the application that requested it does

**Post-VCF SSO role assignment:**
After Identity Broker SSO is configured, import users and assign service roles before they can log in:
`Access Control > Import Users > Assign Role`
JIT-provisioned users must **attempt a login first** before their account exists to assign a role.

### Exam Decision Points
- Org-level custom roles = **cannot be shared** to other orgs
- Provider global roles = **auto-published** to all orgs
- Project role is required — org role alone **does not grant resource access**
- Service account token **rotates on every use**
- JIT users must **log in once** before you can assign their role
- Deleted VCF service rights bundle = **auto-recreated** by VCF Automation

</details>

---

<details>
<summary><strong>4.22 — Create a New Organisation in the Provider Management Portal</strong></summary>

### Key Concepts

Organisations are the top-level multi-tenancy boundary in VCF Automation. Each organisation has its own users, projects, catalog, and networking.

**Creating an organisation:**
`Provider Portal > Infrastructure > Organisations > New Organisation`

Key configuration during creation:
- **Organisation name** — must be unique; becomes part of the organisation's URL
- **Default organisation policy** — resource limits and governance defaults
- **Administrator user** — assign the first org admin (from the configured IdP or system LDAP)
- **Regional resources** — which VCF regions/compute resources are available to this org

**Organisation types in VCF 9:**
- **All Apps Organisation** — general-purpose org for VM and Kubernetes workloads (standard)
- **Provider Consumption Organisation (PCO)** — special org used by the provider to share catalog content across multiple All Apps orgs

**After creation:**
- Assign compute resources (vCenter, clusters) to the org via Provider Portal > Infrastructure
- Assign regional networking (Provider Gateway, IP Space allocations) so org users can create VPCs
- Configure identity provider for the org (objective 4.24)
- Set up governance policies (objective 4.27)

**Local user limitations:**
Local users (non-IdP, non-SSO) in VCF Automation have limited capabilities. All production deployments should use an external identity provider at either system or org level.

### Exam Decision Points
- Organisation name is **immutable** after creation — cannot rename
- All Apps org = standard workload org
- PCO = special provider-managed org for cross-org content sharing
- Regional resources must be **explicitly allocated** to the org by provider admin — not automatic

</details>

---

<details>
<summary><strong>4.23 — Enable and Configure a Provider Consumption Organisation (PCO)</strong></summary>

### Key Concepts

A **Provider Consumption Organisation (PCO)** is a special VCF Automation organisation type that lets the provider share catalog content (workflows, automation items) across multiple All Apps organisations.

**Purpose:** Instead of duplicating automation content in every tenant org, the provider publishes it once via the PCO and it becomes available in the target orgs' catalogs.

**What can be shared via PCO:**
- VCF Operations Orchestrator workflows published as catalog items
- Automation templates
- Standard infrastructure runbooks

**PCO vs Standard Org:**
- PCO is managed by the **provider admin** (not tenant admins)
- PCO content is published **to** All Apps orgs, not consumed by PCO users
- PCO does not provision VMs directly — it is a content distribution mechanism

**Enabling PCO:**
`Provider Portal > Infrastructure > Organisations > [Create or select org] > Type: Provider Consumption Organisation`

Once the PCO has catalog items, the provider publishes them to target orgs:
`PCO Portal > Catalog > [Item] > Publish to Organisations`

### Exam Decision Points
- PCO = **provider publishes content to tenant orgs** — not a workload org
- PCO content is consumed in target org catalogs; PCO users don't directly deploy resources
- PCO workflows are typically Orchestrator-based automation items
- Only **one PCO** exists in a VCF Automation instance (it's a system-level construct)

</details>

---

<details>
<summary><strong>4.24 — Configure Identity Providers within a VCF All Apps Organisation</strong></summary>

### Key Concepts

Configuring the identity provider for an All Apps Organisation is done from within the Organisation Portal (not the Provider Portal).

**Access the organisation portal:**
`Provider Portal > Infrastructure > Organisations > [Org] > three-dot menu > Launch Organisational Portal`

Or directly: `https://<vcf-automation-fqdn>/organization/<org-name>`

**IdP configuration path within org portal:**
`Org Portal > Administer > Connections > Identity Providers`

**Options:**
- **Inherit system LDAP** — uses the provider-level LDAP connection shared across all orgs
- **Custom LDAP** — private LDAP connection for this org only
- **SAML** — federate with a SAML 2.0 IdP
- **OIDC** — OpenID Connect federation (Okta, Entra ID, Ping, etc.)
- **VCF SSO** — connect to the Identity Broker (replaces OIDC tab when configured)

**Same rules apply as 4.19:**
- One integration per protocol type
- OIDC and VCF SSO are mutually exclusive

**After IdP is configured:**
Import users/groups from the IdP and assign org-level roles.

### Exam Decision Points
- Org IdP configuration is in **Org Portal > Administer** — not Provider Portal
- Same one-per-protocol rule applies at org level
- VCF SSO at org level also conflicts with OIDC
- Org admin can configure org-level IdP; provider admin can also do this

</details>

---

<details>
<summary><strong>4.25 — Configure Access Control within a VCF All Apps Organisation</strong></summary>

### Key Concepts

Organisation-level access control covers user/group assignment to organisation roles and project membership.

**Assigning org roles:**
`Org Portal > Administer > Users and Groups > Import` (from configured IdP)
Then `Assign Role` to select Organisation Administrator, Auditor, or User.

**Organisation-level custom roles:**
Org admins can create custom roles within the org, but these are private to the org and cannot be shared to other orgs.

**Project assignment:**
Even with an org role, users need project-level membership to access resources.
`Org Portal > Manage & Govern > Projects > [Project] > Users and Groups > Add`

**Entitling users to catalog content:**
Content Sharing Policies (a governance policy type) control catalog item visibility. Even if a user has a Project Advanced User role, catalog items must be explicitly shared via content sharing policy.

**Key difference from 4.21:** Objective 4.21 is about **provider-level** access control (global roles, rights bundles, service accounts). Objective 4.25 is about **organisation-level** access control (which users can log into the org, their roles within the org, and project memberships).

### Exam Decision Points
- Org custom roles = **private to org**, not shareable
- Project membership is **required** in addition to org role for resource access
- Content sharing policy controls catalog item visibility within a project
- Org admin manages user roles within the org; Provider admin manages global roles

</details>

---

<details>
<summary><strong>4.26 — Create and Manage Projects within a VCF All Apps Organisation</strong></summary>

### Key Concepts

Projects are the operational boundary within an organisation where resources are deployed, governed, and accessed.

**Creating a project:**
`Org Portal > Manage & Govern > Projects > New Project`

Key configuration:
- **Name**
- **Members** — users and groups with assigned project roles
- **Infrastructure** — which zones (vSphere clusters) are available to the project
- **Networking** — which VPCs are available to the project
- **Storage** — which storage classes are available

**Project zones:**
Zones map to vSphere compute resources (clusters). When a VM is deployed via a project, it lands in one of the project's zones. Zone selection can be: specific cluster, or any available zone (cloud-agnostic placement).

**Namespace Classes (4.28 connection):**
Projects can be associated with Namespace Classes, which define the compute, storage, and networking constraints for Supervisor namespaces created within the project.

**Resource quotas at project level:**
Projects can have quota policies applied (via governance) limiting CPU, memory, and storage allocation.

**Sharing a VPC across projects:**
A single VPC can be associated with multiple projects — this enables shared networking infrastructure across teams.

### Exam Decision Points
- Projects = **operational boundary** for resource deployment and governance
- Project zones = **vSphere compute resources** (clusters) available to the project
- One VPC can be shared by **multiple projects**
- Project roles (Administrator/Advanced User/User/Auditor) are assigned per project, not org-wide

</details>

---

<details>
<summary><strong>4.27 — Create and Manage Governance Policies within a VCF All Apps Organisation</strong></summary>

### Key Concepts

Governance policies apply guardrails to provisioning requests within an organisation.

**Policy types:**

| Type | What it controls |
|---|---|
| Approval | Whether deployment requires human approval before proceeding |
| Lease | Time-bounds deployments; auto-shutdown + delete after expiry |
| Day 2 Actions | Which users/roles can run post-deployment actions (resize, delete, etc.) |
| Content Sharing | Which catalog items are visible to which users/projects |

**Configuration path:** `Org Portal > Manage & Govern > Policies > Definitions`

**Hard vs Soft Enforcement — the most critical concept:**

| Scenario | Result |
|---|---|
| Hard org policy + Soft project policy | **Only the hard policy applies**; soft is ignored |
| Soft org policy + Soft project policy | Policies **merge**; most restrictive values from each apply |
| Multiple project soft policies | Merged; most restrictive values combined |

**Policy ranking:** Org-level > Project-level. Older creation date wins as tiebreaker.

**Approval policy specifics:**
- Approvers = **union** of all enforced policies' approver lists
- Auto-expiry = **Reject** if any policy uses Reject (most restrictive wins)
- Expiry = **minimum** of all enforced policy expiry values

**Day 2 Actions policy — critical gotcha:**
When the **first Day 2 Actions policy** is activated, it enforces for **all users in VCF Automation**. Any users not explicitly entitled by a policy are **excluded** from running those actions. This is a common surprise — enabling one Day 2 policy locks out everyone not covered by it.

**Lease policy specifics:**
- Lease = initial deployment duration (days)
- Total Lease = maximum cumulative lease including renewals
- Grace period = days between shutdown and deletion
- If a VM within an expired deployment is manually powered back on, it is not auto-powered-off again until the grace period elapses and the deployment is deleted

**Checking policy enforcement:**
`Org Portal > Manage & Govern > Policies > Enforcement > [deployment] > Decision Notes`

**Policy-As-Code (IaaS VMs and Kubernetes — VCF 9 New):**
YAML-based governance policies for resources provisioned via the self-service IaaS console (not catalog-based). Allows org admins to define quotas and constraints declaratively per project.

### Exam Decision Points
- First Day 2 Actions policy = **everyone else is excluded by default**
- Hard policy + Soft policy = **only hard applies**; soft ignored entirely when hard exists
- Soft + Soft = **merge**, most restrictive values win per dimension
- Approval: Reject auto-expiry wins over Approve if any policy uses Reject
- Lease + Lease merge = minimum Lease value applies
- Policy-As-Code = **IaaS console VMs/K8s** governance (not catalog-based)

</details>

---

<details>
<summary><strong>4.28 — Configure Namespace Classes within a VCF All Apps Organisation</strong></summary>

### Key Concepts

Namespace Classes define the **template of constraints** for Supervisor Namespaces created within a project. They abstract the underlying vSphere/Supervisor configuration from the org user.

**What a Namespace Class defines:**
- Compute: VM Classes available (CPU/memory profiles)
- Storage: Storage Classes available
- Networking: VPC or network configuration constraints
- Resource limits: default CPU, memory, and storage quotas per namespace

**Creating a Namespace Class:**
`Org Portal > Manage & Govern > Namespace Classes > New`

The provider administrator must have allocated Supervisor resources to the organisation first. The org admin then builds Namespace Classes from those allocated resources.

**Usage:**
When a project member creates a namespace via the self-service interface, they select a Namespace Class. The resulting namespace is provisioned with the constraints defined in the class — users don't configure raw vSphere parameters.

**Relationship to projects:**
Namespace Classes are associated with projects. A project can use multiple Namespace Classes (different profiles for different workload types).

### Exam Decision Points
- Namespace Classes abstract Supervisor configuration from org users
- Provider must allocate Supervisor resources to org **before** org admin can create Namespace Classes
- Namespace Class = **template**; Namespace = **instance** created from the template
- Multiple Namespace Classes per project are supported

</details>

---

<details>
<summary><strong>4.29 — Create and Manage VPCs and VPC Connectivity Profiles within a VCF All Apps Organisation</strong></summary>

### Key Concepts

VPCs are the networking building blocks for All Apps organisations, enabling self-service network isolation for VM and Kubernetes workloads.

**VPC subnet types:**

| Subnet Type | Connectivity | Use For |
|---|---|---|
| Private – VPC | Internal to VPC only; NAT needed for external | Isolated workloads |
| Private – Transit Gateway | Inter-VPC routing via TGW; no external access without additional config | Cross-VPC communication |
| Public | Accessible externally and from other VPCs | Internet-facing workloads |

**External IPs:** 1:1 NAT — assigns an IP from the External IP block directly to a VM. Distinct from Default Outbound NAT (which is shared SNAT).

**Creating a VPC:**
`Org Portal > Build & Deploy > Services > Network > Virtual Private Clouds > New VPC`

Steps:
1. Name, select region
2. Select or create a **Connectivity Profile** (defines TGW, external IP blocks, private TGW IP blocks)
3. Specify private VPC CIDR blocks

**Connectivity Profiles:**
Connectivity profiles are reusable templates for VPC northbound connectivity. A default profile is created when the provider allocates regional networking to the org. Org admin can create additional profiles.

`NSX Manager > Project > VPCs > Profiles > VPC Connectivity Profile > Add`

Key fields: Transit Gateway, External IP Blocks, Private-TGW IP Blocks, Edge Cluster, N-S Services toggle (enables stateful NAT/firewall), Default Outbound NAT (auto-SNAT; requires N-S Services ON).

**Transit Gateway types:**

| Type | Characteristic |
|---|---|
| Centralised TGW (CTGW) | Routes through NSX edge nodes; supports stateful N-S services; T0 must be Active/Standby |
| Distributed TGW (DTGW) | Direct host-to-fabric via VLAN; better throughput; DHCP Relay NOT supported |

**Subnet Sets:**
Org portal construct — abstracts subnets from namespaces. Namespaces use subnet sets instead of being assigned raw subnets. Enables consistent IP management across multiple namespaces sharing a VPC.

**IP block scopes:**

| Block Type | Who Creates | Scope |
|---|---|---|
| External IP Blocks | Provider admin allocates to org | Public/external access |
| Private-TGW IP Blocks | Org admin creates | Inter-VPC routing |
| Private-VPC CIDRs | Org admin defines per VPC | VPC-internal only; can overlap across VPCs |

### Exam Decision Points
- DTGW = **no DHCP Relay** support
- Default Outbound NAT requires **N-S Services enabled** on connectivity profile
- External IP = **1:1 NAT** (not shared SNAT)
- Private-VPC CIDRs **can overlap** across different VPCs (they're isolated)
- VPC can be shared by **multiple projects/namespaces**
- Connectivity profile changes apply to **all VPCs using that profile**
- Delete VPC: must have **no associated namespaces** first

</details>

---

*Previous: [vSphere — Objectives 4.10–4.18]({% post_url 2026-03-20-vcap-admin-vcf9-s2-vsphere %}) · Next: [Supervisor & VKS — Objectives 4.30–4.34]({% post_url 2026-03-20-vcap-admin-vcf9-s4-supervisor %})*
