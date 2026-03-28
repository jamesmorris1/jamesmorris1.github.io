---
layout: post
title: "VCAP-ADMIN VCF 9.0 — VCF Automation Provider & Org: Objectives 4.19–4.29"
date: 2026-03-20
categories: [certification, vcf, vcap]
tags: [vcap, vcf9, vcf-automation, 3v0-11-26, study-guide]
series: "VCAP-ADMIN VCF 9.0 Study Guide (3V0-11.26)"
series_part: 3
---

# Section: VCF Automation — Provider & Organisation Management — Objectives 4.19–4.29

Part of the [VCAP-ADMIN VCF 9.0 Study Guide series]({% post_url 2026-03-20-vcap-admin-vcf9-study-guide-index %}).

---

<details>
<summary><strong>4.19 — Configure the Identity Provider for the Provider Management Portal</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>The Provider Management Portal (<code>https://&lt;vcf-automation-fqdn&gt;/provider</code>) uses its <strong>own</strong> identity provider configuration — separate from the vSphere Identity Broker (objective 4.17).</p>
<p><strong>Two configuration levels:</strong></p>
<table>
<thead>
<tr>
<th>Level</th>
<th>Scope</th>
<th>Who Configures</th>
<th>What Can Be Configured</th>
</tr>
</thead>
<tbody>
<tr>
<td>System (Provider)</td>
<td>All organisations</td>
<td>Provider/System Admin</td>
<td>LDAP only</td>
</tr>
<tr>
<td>Organisation</td>
<td>Specific org only</td>
<td>Provider Admin or Org Admin</td>
<td>LDAP (custom), SAML, OIDC</td>
</tr>
</tbody>
</table>
<p><strong>Critical exam rule:</strong> You can have <strong>one integration per protocol type</strong> — one LDAP, one SAML, one OIDC simultaneously. You cannot have two LDAP connections at the same level.</p>
<p><strong>System LDAP (shared across all orgs):</strong><br />
<code>Provider Portal &gt; Administration &gt; Identity Providers &gt; LDAP</code><br />
Organisations can inherit the system LDAP or override with a private custom LDAP.</p>
<p><strong>SAML (organisation level):</strong><br />
Exchange metadata between VCF Automation (SP) and external IdP. VCF Automation extracts group/role attributes from the SAML token.</p>
<p><strong>OIDC (organisation level):</strong><br />
Supports Configuration Discovery (well-known endpoint auto-fill). Supports <strong>PKCE</strong> (Proof Key for Code Exchange) for additional auth code security.<br />
<code>Provider Portal &gt; Administration &gt; Identity Providers &gt; OIDC &gt; Enable PKCE</code></p>
<p><strong>OIDC vs VCF SSO conflict:</strong><br />
If an OIDC connection is configured for the provider org, you <strong>cannot also configure VCF SSO</strong>. Must remove OIDC first before connecting VCF SSO (and vice versa). When VCF SSO is configured, the OIDC tab is replaced by a VCF SSO tab.</p>
<p><strong>User remapping:</strong><br />
When migrating from one IdP to another, use <strong>Remap Users</strong> to preserve roles and resource ownership.<br />
<code>Provider Portal &gt; Administration &gt; Identity Providers &gt; Remap Users</code></p>
<p><strong>First login:</strong> Default local admin user is <code>admin</code> — password set during installation. After 25 minutes of inactivity, session expires (does not interrupt long-running operations).</p>
<h3>Exam Decision Points</h3>
<ul>
<li>System LDAP = provider-level, <strong>shared</strong> across all orgs</li>
<li>Custom LDAP = org-level, <strong>private</strong> to that org</li>
<li>OIDC + VCF SSO = <strong>mutually exclusive</strong> — cannot coexist; remove one before configuring the other</li>
<li>One integration per protocol type (1 LDAP + 1 SAML + 1 OIDC max per org)</li>
<li>Remap Users = preserves existing roles/resources during IdP migration</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.20 — Create and Manage Provider Content Libraries within the Provider Management Portal</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Provider Content Libraries make VM images (OVA/OVF) and ISOs available to organisation tenants for use in blueprints and IaaS provisioning.</p>
<p><strong>Supported content types:</strong> ISO and OVA/OVF only. <strong>Existing VMs cannot be captured</strong> into a content library — you must export to OVA first.</p>
<p><strong>Provider Content Library vs Organisation Content Library:</strong></p>
<table>
<thead>
<tr>
<th>Library Type</th>
<th>Creator</th>
<th>Visibility to Orgs</th>
</tr>
</thead>
<tbody>
<tr>
<td>Provider Content Library</td>
<td>Provider Admin</td>
<td>Read-only</td>
</tr>
<tr>
<td>Organisation Content Library</td>
<td>Org Admin</td>
<td>Private to that org</td>
</tr>
</tbody>
</table>
<p><strong>Creating a Provider Content Library:</strong><br />
<code>Provider Portal &gt; Content Libraries &gt; New</code><br />
1. Name the library<br />
2. Select backing vCenter content library (must be a <strong>published</strong> vCenter content library)<br />
3. Select regions and storage class per region<br />
4. Select whether to assign to all current/future namespaces — <strong>cannot be changed after creation</strong></p>
<p><strong>Subscribed Content Libraries (org-level):</strong></p>
<ul>
<li>Provider must explicitly enable this capability</li>
<li>Subscribed libraries are <strong>read-only</strong> — cannot add or delete items</li>
<li><strong>No automatic or scheduled synchronisation</strong> — manual refresh only</li>
<li>Sync respects org storage quota</li>
</ul>
<p><strong>Supervisor Release Images (VCF 9 — critical new feature):</strong><br />
Starting in VCF 9, Kubernetes release images for Supervisor are delivered via Broadcom's CDN, not bundled with vCenter.<br />
- Create a subscribed content library pointing to Broadcom CDN<br />
- Immediate or on-demand sync modes<br />
- <strong>Do not delete or edit</strong> a content library assigned with Supervisor release images — unassign first<br />
- Air-gapped: create local publisher → subscribe internally</p>
<p><strong>API operations:</strong></p>
<pre><code class="language-bash"># List accessible content libraries for an org
GET https://&lt;vcfa-fqdn&gt;/cloudapi/vcf/contentLibraries?page=1&amp;pageSize=128

# Refresh a catalog
POST https://&lt;vcfa-fqdn&gt;/cloudapi/1.0.0/catalogs/&lt;urn&gt;/refresh
</code></pre>
<h3>Exam Decision Points</h3>
<ul>
<li>Region assignment (namespace assignment) = <strong>cannot be changed after library creation</strong></li>
<li>Subscribed org libraries = <strong>manual refresh only</strong> (no auto-sync)</li>
<li>Existing VMs = <strong>not capturable</strong> directly into content library (must export to OVA)</li>
<li>Supervisor images = <strong>subscribed CDN library</strong> required in VCF 9 (not bundled with vCenter install)</li>
<li>Air-gapped Supervisor images = local publisher first, then internal subscribed library</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.21 — Configure Access Control Settings within the Provider Management Portal</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>VCF Automation uses a <strong>three-tier RBAC model</strong>: Provider → Organisation → Project.</p>
<p><strong>Tier 1 — Provider Level (Global Roles):</strong><br />
Global roles are created in the Provider Admin Portal and <strong>automatically published to all organisations</strong>.</p>
<table>
<thead>
<tr>
<th>Role</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>Defer to Identity Provider</td>
<td>Rights determined by IdP token claims</td>
</tr>
<tr>
<td>Organisation Administrator</td>
<td>Full org management</td>
</tr>
<tr>
<td>Organisation Auditor</td>
<td>Read-only org access</td>
</tr>
<tr>
<td>Organisation User</td>
<td>Standard user rights</td>
</tr>
<tr>
<td>Custom Role</td>
<td>Created by provider; published selectively</td>
</tr>
</tbody>
</table>
<p><code>Provider Portal &gt; Administration &gt; Access Control &gt; Roles &gt; New</code></p>
<p>Create custom roles <strong>before</strong> creating an organisation if you need to assign a custom role to the first org user.</p>
<p><strong>Tier 2 — Organisation Level:</strong><br />
Orgs inherit provider global roles. Org admins can create <strong>custom org-level roles</strong>, but these <strong>cannot be shared to other organisations</strong>.<br />
Default inherited roles cannot be edited at the org level.</p>
<p><strong>Tier 3 — Project Level:</strong><br />
Users need a project-level role to access resources — org role alone is not enough.</p>
<table>
<thead>
<tr>
<th>Project Role</th>
<th>Rights</th>
</tr>
</thead>
<tbody>
<tr>
<td>Project Administrator</td>
<td>Manage members and catalog content</td>
</tr>
<tr>
<td>Project Advanced User</td>
<td>Request catalog items and use IaaS services</td>
</tr>
<tr>
<td>Project User</td>
<td>Request catalog items only</td>
</tr>
<tr>
<td>Project Auditor</td>
<td>View all project content, no requests</td>
</tr>
</tbody>
</table>
<p><strong>Rights Bundles:</strong><br />
Extend capabilities of global roles for add-on services (e.g., VCF Operations Orchestrator).<br />
<code>Provider Portal &gt; Administration &gt; Access Control &gt; Rights Bundles</code></p>
<p>To publish Orchestrator to an org:<br />
1. Enable <strong>Advanced Rights Bundle Mode</strong> in Feature Flags<br />
2. Publish Orchestrator Rights Bundle to the org</p>
<p>If a rights bundle belonging to a VCF service is accidentally deleted, VCF Automation <strong>auto-recreates it within minutes</strong>.</p>
<p><strong>Service Accounts:</strong></p>
<ul>
<li>API-only access; token rotates on <strong>every use</strong> (RFC 6749 section 6)</li>
<li>Request flow: application requests via device code → admin reviews and grants via <code>Access Control &gt; Service Accounts &gt; Review Access Requests</code></li>
<li>Revoking a token: terminates all sessions but leaves the service account in "Created" state (not deleted)</li>
<li>Admins <strong>never see the actual token</strong> — only the application that requested it does</li>
</ul>
<p><strong>Post-VCF SSO role assignment:</strong><br />
After Identity Broker SSO is configured, import users and assign service roles before they can log in:<br />
<code>Access Control &gt; Import Users &gt; Assign Role</code><br />
JIT-provisioned users must <strong>attempt a login first</strong> before their account exists to assign a role.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Org-level custom roles = <strong>cannot be shared</strong> to other orgs</li>
<li>Provider global roles = <strong>auto-published</strong> to all orgs</li>
<li>Project role is required — org role alone <strong>does not grant resource access</strong></li>
<li>Service account token <strong>rotates on every use</strong></li>
<li>JIT users must <strong>log in once</strong> before you can assign their role</li>
<li>Deleted VCF service rights bundle = <strong>auto-recreated</strong> by VCF Automation</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.22 — Create a New Organisation in the Provider Management Portal</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Organisations are the top-level multi-tenancy boundary in VCF Automation. Each organisation has its own users, projects, catalog, and networking.</p>
<p><strong>Creating an organisation:</strong><br />
<code>Provider Portal &gt; Infrastructure &gt; Organisations &gt; New Organisation</code></p>
<p>Key configuration during creation:</p>
<ul>
<li><strong>Organisation name</strong> — must be unique; becomes part of the organisation's URL</li>
<li><strong>Default organisation policy</strong> — resource limits and governance defaults</li>
<li><strong>Administrator user</strong> — assign the first org admin (from the configured IdP or system LDAP)</li>
<li><strong>Regional resources</strong> — which VCF regions/compute resources are available to this org</li>
</ul>
<p><strong>Organisation types in VCF 9:</strong></p>
<ul>
<li><strong>All Apps Organisation</strong> — general-purpose org for VM and Kubernetes workloads (standard)</li>
<li><strong>Provider Consumption Organisation (PCO)</strong> — special org used by the provider to share catalog content across multiple All Apps orgs</li>
</ul>
<p><strong>After creation:</strong></p>
<ul>
<li>Assign compute resources (vCenter, clusters) to the org via Provider Portal &gt; Infrastructure</li>
<li>Assign regional networking (Provider Gateway, IP Space allocations) so org users can create VPCs</li>
<li>Configure identity provider for the org (objective 4.24)</li>
<li>Set up governance policies (objective 4.27)</li>
</ul>
<p><strong>Local user limitations:</strong><br />
Local users (non-IdP, non-SSO) in VCF Automation have limited capabilities. All production deployments should use an external identity provider at either system or org level.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Organisation name is <strong>immutable</strong> after creation — cannot rename</li>
<li>All Apps org = standard workload org</li>
<li>PCO = special provider-managed org for cross-org content sharing</li>
<li>Regional resources must be <strong>explicitly allocated</strong> to the org by provider admin — not automatic</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.23 — Enable and Configure a Provider Consumption Organisation (PCO)</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>A <strong>Provider Consumption Organisation (PCO)</strong> is a special VCF Automation organisation type that lets the provider share catalog content (workflows, automation items) across multiple All Apps organisations.</p>
<p><strong>Purpose:</strong> Instead of duplicating automation content in every tenant org, the provider publishes it once via the PCO and it becomes available in the target orgs' catalogs.</p>
<p><strong>What can be shared via PCO:</strong></p>
<ul>
<li>VCF Operations Orchestrator workflows published as catalog items</li>
<li>Automation templates</li>
<li>Standard infrastructure runbooks</li>
</ul>
<p><strong>PCO vs Standard Org:</strong></p>
<ul>
<li>PCO is managed by the <strong>provider admin</strong> (not tenant admins)</li>
<li>PCO content is published <strong>to</strong> All Apps orgs, not consumed by PCO users</li>
<li>PCO does not provision VMs directly — it is a content distribution mechanism</li>
</ul>
<p><strong>Enabling PCO:</strong><br />
<code>Provider Portal &gt; Infrastructure &gt; Organisations &gt; [Create or select org] &gt; Type: Provider Consumption Organisation</code></p>
<p>Once the PCO has catalog items, the provider publishes them to target orgs:<br />
<code>PCO Portal &gt; Catalog &gt; [Item] &gt; Publish to Organisations</code></p>
<h3>Exam Decision Points</h3>
<ul>
<li>PCO = <strong>provider publishes content to tenant orgs</strong> — not a workload org</li>
<li>PCO content is consumed in target org catalogs; PCO users don't directly deploy resources</li>
<li>PCO workflows are typically Orchestrator-based automation items</li>
<li>Only <strong>one PCO</strong> exists in a VCF Automation instance (it's a system-level construct)</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.24 — Configure Identity Providers within a VCF All Apps Organisation</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Configuring the identity provider for an All Apps Organisation is done from within the Organisation Portal (not the Provider Portal).</p>
<p><strong>Access the organisation portal:</strong><br />
<code>Provider Portal &gt; Infrastructure &gt; Organisations &gt; [Org] &gt; three-dot menu &gt; Launch Organisational Portal</code></p>
<p>Or directly: <code>https://&lt;vcf-automation-fqdn&gt;/organization/&lt;org-name&gt;</code></p>
<p><strong>IdP configuration path within org portal:</strong><br />
<code>Org Portal &gt; Administer &gt; Connections &gt; Identity Providers</code></p>
<p><strong>Options:</strong></p>
<ul>
<li><strong>Inherit system LDAP</strong> — uses the provider-level LDAP connection shared across all orgs</li>
<li><strong>Custom LDAP</strong> — private LDAP connection for this org only</li>
<li><strong>SAML</strong> — federate with a SAML 2.0 IdP</li>
<li><strong>OIDC</strong> — OpenID Connect federation (Okta, Entra ID, Ping, etc.)</li>
<li><strong>VCF SSO</strong> — connect to the Identity Broker (replaces OIDC tab when configured)</li>
</ul>
<p><strong>Same rules apply as 4.19:</strong></p>
<ul>
<li>One integration per protocol type</li>
<li>OIDC and VCF SSO are mutually exclusive</li>
</ul>
<p><strong>After IdP is configured:</strong><br />
Import users/groups from the IdP and assign org-level roles.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Org IdP configuration is in <strong>Org Portal &gt; Administer</strong> — not Provider Portal</li>
<li>Same one-per-protocol rule applies at org level</li>
<li>VCF SSO at org level also conflicts with OIDC</li>
<li>Org admin can configure org-level IdP; provider admin can also do this</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.25 — Configure Access Control within a VCF All Apps Organisation</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Organisation-level access control covers user/group assignment to organisation roles and project membership.</p>
<p><strong>Assigning org roles:</strong><br />
<code>Org Portal &gt; Administer &gt; Users and Groups &gt; Import</code> (from configured IdP)<br />
Then <code>Assign Role</code> to select Organisation Administrator, Auditor, or User.</p>
<p><strong>Organisation-level custom roles:</strong><br />
Org admins can create custom roles within the org, but these are private to the org and cannot be shared to other orgs.</p>
<p><strong>Project assignment:</strong><br />
Even with an org role, users need project-level membership to access resources.<br />
<code>Org Portal &gt; Manage &amp; Govern &gt; Projects &gt; [Project] &gt; Users and Groups &gt; Add</code></p>
<p><strong>Entitling users to catalog content:</strong><br />
Content Sharing Policies (a governance policy type) control catalog item visibility. Even if a user has a Project Advanced User role, catalog items must be explicitly shared via content sharing policy.</p>
<p><strong>Key difference from 4.21:</strong> Objective 4.21 is about <strong>provider-level</strong> access control (global roles, rights bundles, service accounts). Objective 4.25 is about <strong>organisation-level</strong> access control (which users can log into the org, their roles within the org, and project memberships).</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Org custom roles = <strong>private to org</strong>, not shareable</li>
<li>Project membership is <strong>required</strong> in addition to org role for resource access</li>
<li>Content sharing policy controls catalog item visibility within a project</li>
<li>Org admin manages user roles within the org; Provider admin manages global roles</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.26 — Create and Manage Projects within a VCF All Apps Organisation</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Projects are the operational boundary within an organisation where resources are deployed, governed, and accessed.</p>
<p><strong>Creating a project:</strong><br />
<code>Org Portal &gt; Manage &amp; Govern &gt; Projects &gt; New Project</code></p>
<p>Key configuration:</p>
<ul>
<li><strong>Name</strong></li>
<li><strong>Members</strong> — users and groups with assigned project roles</li>
<li><strong>Infrastructure</strong> — which zones (vSphere clusters) are available to the project</li>
<li><strong>Networking</strong> — which VPCs are available to the project</li>
<li><strong>Storage</strong> — which storage classes are available</li>
</ul>
<p><strong>Project zones:</strong><br />
Zones map to vSphere compute resources (clusters). When a VM is deployed via a project, it lands in one of the project's zones. Zone selection can be: specific cluster, or any available zone (cloud-agnostic placement).</p>
<p><strong>Namespace Classes (4.28 connection):</strong><br />
Projects can be associated with Namespace Classes, which define the compute, storage, and networking constraints for Supervisor namespaces created within the project.</p>
<p><strong>Resource quotas at project level:</strong><br />
Projects can have quota policies applied (via governance) limiting CPU, memory, and storage allocation.</p>
<p><strong>Sharing a VPC across projects:</strong><br />
A single VPC can be associated with multiple projects — this enables shared networking infrastructure across teams.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Projects = <strong>operational boundary</strong> for resource deployment and governance</li>
<li>Project zones = <strong>vSphere compute resources</strong> (clusters) available to the project</li>
<li>One VPC can be shared by <strong>multiple projects</strong></li>
<li>Project roles (Administrator/Advanced User/User/Auditor) are assigned per project, not org-wide</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.27 — Create and Manage Governance Policies within a VCF All Apps Organisation</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Governance policies apply guardrails to provisioning requests within an organisation.</p>
<p><strong>Policy types:</strong></p>
<table>
<thead>
<tr>
<th>Type</th>
<th>What it controls</th>
</tr>
</thead>
<tbody>
<tr>
<td>Approval</td>
<td>Whether deployment requires human approval before proceeding</td>
</tr>
<tr>
<td>Lease</td>
<td>Time-bounds deployments; auto-shutdown + delete after expiry</td>
</tr>
<tr>
<td>Day 2 Actions</td>
<td>Which users/roles can run post-deployment actions (resize, delete, etc.)</td>
</tr>
<tr>
<td>Content Sharing</td>
<td>Which catalog items are visible to which users/projects</td>
</tr>
</tbody>
</table>
<p><strong>Configuration path:</strong> <code>Org Portal &gt; Manage &amp; Govern &gt; Policies &gt; Definitions</code></p>
<p><strong>Hard vs Soft Enforcement — the most critical concept:</strong></p>
<table>
<thead>
<tr>
<th>Scenario</th>
<th>Result</th>
</tr>
</thead>
<tbody>
<tr>
<td>Hard org policy + Soft project policy</td>
<td><strong>Only the hard policy applies</strong>; soft is ignored</td>
</tr>
<tr>
<td>Soft org policy + Soft project policy</td>
<td>Policies <strong>merge</strong>; most restrictive values from each apply</td>
</tr>
<tr>
<td>Multiple project soft policies</td>
<td>Merged; most restrictive values combined</td>
</tr>
</tbody>
</table>
<p><strong>Policy ranking:</strong> Org-level &gt; Project-level. Older creation date wins as tiebreaker.</p>
<p><strong>Approval policy specifics:</strong></p>
<ul>
<li>Approvers = <strong>union</strong> of all enforced policies' approver lists</li>
<li>Auto-expiry = <strong>Reject</strong> if any policy uses Reject (most restrictive wins)</li>
<li>Expiry = <strong>minimum</strong> of all enforced policy expiry values</li>
</ul>
<p><strong>Day 2 Actions policy — critical gotcha:</strong><br />
When the <strong>first Day 2 Actions policy</strong> is activated, it enforces for <strong>all users in VCF Automation</strong>. Any users not explicitly entitled by a policy are <strong>excluded</strong> from running those actions. This is a common surprise — enabling one Day 2 policy locks out everyone not covered by it.</p>
<p><strong>Lease policy specifics:</strong></p>
<ul>
<li>Lease = initial deployment duration (days)</li>
<li>Total Lease = maximum cumulative lease including renewals</li>
<li>Grace period = days between shutdown and deletion</li>
<li>If a VM within an expired deployment is manually powered back on, it is not auto-powered-off again until the grace period elapses and the deployment is deleted</li>
</ul>
<p><strong>Checking policy enforcement:</strong><br />
<code>Org Portal &gt; Manage &amp; Govern &gt; Policies &gt; Enforcement &gt; [deployment] &gt; Decision Notes</code></p>
<p><strong>Policy-As-Code (IaaS VMs and Kubernetes — VCF 9 New):</strong><br />
YAML-based governance policies for resources provisioned via the self-service IaaS console (not catalog-based). Allows org admins to define quotas and constraints declaratively per project.</p>
<h3>Exam Decision Points</h3>
<ul>
<li>First Day 2 Actions policy = <strong>everyone else is excluded by default</strong></li>
<li>Hard policy + Soft policy = <strong>only hard applies</strong>; soft ignored entirely when hard exists</li>
<li>Soft + Soft = <strong>merge</strong>, most restrictive values win per dimension</li>
<li>Approval: Reject auto-expiry wins over Approve if any policy uses Reject</li>
<li>Lease + Lease merge = minimum Lease value applies</li>
<li>Policy-As-Code = <strong>IaaS console VMs/K8s</strong> governance (not catalog-based)</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.28 — Configure Namespace Classes within a VCF All Apps Organisation</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>Namespace Classes define the <strong>template of constraints</strong> for Supervisor Namespaces created within a project. They abstract the underlying vSphere/Supervisor configuration from the org user.</p>
<p><strong>What a Namespace Class defines:</strong></p>
<ul>
<li>Compute: VM Classes available (CPU/memory profiles)</li>
<li>Storage: Storage Classes available</li>
<li>Networking: VPC or network configuration constraints</li>
<li>Resource limits: default CPU, memory, and storage quotas per namespace</li>
</ul>
<p><strong>Creating a Namespace Class:</strong><br />
<code>Org Portal &gt; Manage &amp; Govern &gt; Namespace Classes &gt; New</code></p>
<p>The provider administrator must have allocated Supervisor resources to the organisation first. The org admin then builds Namespace Classes from those allocated resources.</p>
<p><strong>Usage:</strong><br />
When a project member creates a namespace via the self-service interface, they select a Namespace Class. The resulting namespace is provisioned with the constraints defined in the class — users don't configure raw vSphere parameters.</p>
<p><strong>Relationship to projects:</strong><br />
Namespace Classes are associated with projects. A project can use multiple Namespace Classes (different profiles for different workload types).</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Namespace Classes abstract Supervisor configuration from org users</li>
<li>Provider must allocate Supervisor resources to org <strong>before</strong> org admin can create Namespace Classes</li>
<li>Namespace Class = <strong>template</strong>; Namespace = <strong>instance</strong> created from the template</li>
<li>Multiple Namespace Classes per project are supported</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.29 — Create and Manage VPCs and VPC Connectivity Profiles within a VCF All Apps Organisation</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>VPCs are the networking building blocks for All Apps organisations, enabling self-service network isolation for VM and Kubernetes workloads.</p>
<p><strong>VPC subnet types:</strong></p>
<table>
<thead>
<tr>
<th>Subnet Type</th>
<th>Connectivity</th>
<th>Use For</th>
</tr>
</thead>
<tbody>
<tr>
<td>Private – VPC</td>
<td>Internal to VPC only; NAT needed for external</td>
<td>Isolated workloads</td>
</tr>
<tr>
<td>Private – Transit Gateway</td>
<td>Inter-VPC routing via TGW; no external access without additional config</td>
<td>Cross-VPC communication</td>
</tr>
<tr>
<td>Public</td>
<td>Accessible externally and from other VPCs</td>
<td>Internet-facing workloads</td>
</tr>
</tbody>
</table>
<p><strong>External IPs:</strong> 1:1 NAT — assigns an IP from the External IP block directly to a VM. Distinct from Default Outbound NAT (which is shared SNAT).</p>
<p><strong>Creating a VPC:</strong><br />
<code>Org Portal &gt; Build &amp; Deploy &gt; Services &gt; Network &gt; Virtual Private Clouds &gt; New VPC</code></p>
<p>Steps:<br />
1. Name, select region<br />
2. Select or create a <strong>Connectivity Profile</strong> (defines TGW, external IP blocks, private TGW IP blocks)<br />
3. Specify private VPC CIDR blocks</p>
<p><strong>Connectivity Profiles:</strong><br />
Connectivity profiles are reusable templates for VPC northbound connectivity. A default profile is created when the provider allocates regional networking to the org. Org admin can create additional profiles.</p>
<p><code>NSX Manager &gt; Project &gt; VPCs &gt; Profiles &gt; VPC Connectivity Profile &gt; Add</code></p>
<p>Key fields: Transit Gateway, External IP Blocks, Private-TGW IP Blocks, Edge Cluster, N-S Services toggle (enables stateful NAT/firewall), Default Outbound NAT (auto-SNAT; requires N-S Services ON).</p>
<p><strong>Transit Gateway types:</strong></p>
<table>
<thead>
<tr>
<th>Type</th>
<th>Characteristic</th>
</tr>
</thead>
<tbody>
<tr>
<td>Centralised TGW (CTGW)</td>
<td>Routes through NSX edge nodes; supports stateful N-S services; T0 must be Active/Standby</td>
</tr>
<tr>
<td>Distributed TGW (DTGW)</td>
<td>Direct host-to-fabric via VLAN; better throughput; DHCP Relay NOT supported</td>
</tr>
</tbody>
</table>
<p><strong>Subnet Sets:</strong><br />
Org portal construct — abstracts subnets from namespaces. Namespaces use subnet sets instead of being assigned raw subnets. Enables consistent IP management across multiple namespaces sharing a VPC.</p>
<p><strong>IP block scopes:</strong></p>
<table>
<thead>
<tr>
<th>Block Type</th>
<th>Who Creates</th>
<th>Scope</th>
</tr>
</thead>
<tbody>
<tr>
<td>External IP Blocks</td>
<td>Provider admin allocates to org</td>
<td>Public/external access</td>
</tr>
<tr>
<td>Private-TGW IP Blocks</td>
<td>Org admin creates</td>
<td>Inter-VPC routing</td>
</tr>
<tr>
<td>Private-VPC CIDRs</td>
<td>Org admin defines per VPC</td>
<td>VPC-internal only; can overlap across VPCs</td>
</tr>
</tbody>
</table>
<h3>Exam Decision Points</h3>
<ul>
<li>DTGW = <strong>no DHCP Relay</strong> support</li>
<li>Default Outbound NAT requires <strong>N-S Services enabled</strong> on connectivity profile</li>
<li>External IP = <strong>1:1 NAT</strong> (not shared SNAT)</li>
<li>Private-VPC CIDRs <strong>can overlap</strong> across different VPCs (they're isolated)</li>
<li>VPC can be shared by <strong>multiple projects/namespaces</strong></li>
<li>Connectivity profile changes apply to <strong>all VPCs using that profile</strong></li>
<li>Delete VPC: must have <strong>no associated namespaces</strong> first</li>
</ul>
</div>
</details>

---

*Previous: [vSphere — Objectives 4.10–4.18]({% post_url 2026-03-20-vcap-admin-vcf9-s2-vsphere %}) · Next: [Supervisor & VKS — Objectives 4.30–4.34]({% post_url 2026-03-20-vcap-admin-vcf9-s4-supervisor %})*
