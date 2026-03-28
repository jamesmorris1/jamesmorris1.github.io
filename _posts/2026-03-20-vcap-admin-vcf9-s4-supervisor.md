---
layout: post
title: "VCAP-ADMIN VCF 9.0 — Supervisor & VKS: Objectives 4.30–4.34"
date: 2026-03-20
categories: [certification, vcf, vcap]
tags: [vcap, vcf9, supervisor, vks, kubernetes, 3v0-11-26, study-guide]
series: "VCAP-ADMIN VCF 9.0 Study Guide (3V0-11.26)"
series_part: 4
---

# Section: Supervisor & VKS — Objectives 4.30–4.34

Part of the [VCAP-ADMIN VCF 9.0 Study Guide series]({% post_url 2026-03-20-vcap-admin-vcf9-study-guide-index %}).

---

<details>
<summary><strong>4.30 — Deploy and Configure a Supervisor Cluster</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>The Supervisor is the control plane for Kubernetes workloads in VCF. It enables namespaces, VKS cluster deployment, and platform services on top of vSphere/NSX.</p>
<p><strong>Prerequisites:</strong></p>
<ul>
<li>VCF Workload Domain with NSX deployed</li>
<li>NSX Tier-0 and Tier-1 gateways configured</li>
<li>NSX Edge Cluster available</li>
<li>Content Library with Supervisor release images (VCF 9: CDN subscribed library required)</li>
<li>vSphere Distributed Switch (vDS) for Supervisor networking</li>
</ul>
<p><strong>Deployment path:</strong><br />
<code>vSphere Client &gt; Workload Management &gt; Supervisor Clusters &gt; Enable</code></p>
<p>Wizard steps:<br />
1. Select vSphere cluster to enable as Supervisor<br />
2. Select vDS and configure Supervisor networking (Supervisor Control Plane IP, Ingress CIDR, Egress CIDR)<br />
3. Select NSX configuration (T0 gateway, Edge Cluster)<br />
4. Select Content Library (must contain Supervisor release images)<br />
5. Configure API server VIP (Supervisor Control Plane endpoint)<br />
6. Set storage policies for etcd and Supervisor control plane VMs</p>
<p><strong>Supervisor networking in VCF 9:</strong><br />
In VCF 9, the Supervisor uses NSX for networking. The Supervisor Control Plane VMs (3 VMs: one per host in the management cluster by default) get IPs from the Supervisor management network.</p>
<p><strong>Supervisor Control Plane VIP:</strong><br />
The Supervisor API server is accessed via a VIP (e.g., 10.10.0.40 in lab). This is the endpoint used for <code>kubectl</code> commands against the Supervisor:</p>
<pre><code class="language-bash">kubectl vsphere login --server=10.10.0.40 --insecure-skip-tls-verify -u administrator@vsphere.local
</code></pre>
<p><strong>Verifying Supervisor health:</strong><br />
<code>vSphere Client &gt; Workload Management &gt; Supervisors &gt; [Supervisor] &gt; Summary</code><br />
All control plane nodes should be Running. If any show ConfigError or Degraded:<br />
- Check <code>vmware-trustmanagement</code> service on SDDC Manager<br />
- Verify spherelet VIBs are installed on ESXi hosts<br />
- Check NSX Tier-0/Tier-1 connectivity</p>
<p><strong>Supervisor services status:</strong></p>
<pre><code class="language-bash"># On SDDC Manager:
systemctl status vmware-trustmanagement
# On ESXi hosts:
esxcli software vib list | grep spherelet
</code></pre>
<h3>Exam Decision Points</h3>
<ul>
<li>VCF 9: Supervisor images come from <strong>CDN subscribed content library</strong> — not bundled with vCenter</li>
<li>Supervisor Control Plane = <strong>3 VMs</strong> deployed across management cluster hosts</li>
<li>Supervisor VIP = <strong>API server endpoint</strong> for kubectl</li>
<li><code>vmware-trustmanagement</code> failure on SDDC Manager can block Supervisor enablement</li>
<li>Spherelet VIBs must be on ESXi hosts before Supervisor can provision namespaces</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.31 — Install, Uninstall, and Manage Supervisor Add-on Services (e.g., Harbor, external-dns)</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>There are <strong>two distinct layers</strong> of add-on services in VCF:</p>
<table>
<thead>
<tr>
<th>Layer</th>
<th>What</th>
<th>Where managed</th>
<th>Examples</th>
</tr>
</thead>
<tbody>
<tr>
<td>Supervisor Services</td>
<td>Cluster-wide services on the Supervisor</td>
<td>vCenter UI or VCF Automation Provider Portal</td>
<td>Harbor (Supervisor Service), Contour, external-dns</td>
</tr>
<tr>
<td>VKS Add-ons (Packages)</td>
<td>Packages on individual VKS clusters</td>
<td><code>vcf package</code> CLI or <code>kubectl</code></td>
<td>Harbor, cert-manager, Velero, Prometheus, Fluent-Bit, Istio</td>
</tr>
</tbody>
</table>
<p><strong>Supervisor Services — Install path (vCenter):</strong><br />
<code>Hosts and Clusters &gt; Supervisor &gt; Configure &gt; Supervisor Services &gt; Add Service</code></p>
<p>Service files: each Supervisor Service comes with:</p>
<ul>
<li>A <code>.yml</code> service definition file (metadata)</li>
<li>A data values/configuration <code>.yml</code> file (customise before applying)</li>
</ul>
<p><strong>Supervisor Services — Install path (VCF Automation Provider Portal):</strong><br />
<code>Provider Portal &gt; Services &gt; Upload service file &gt; Install instance &gt; Publish to organisations</code></p>
<p><strong>Harbor as Supervisor Service — prerequisites:</strong></p>
<ul>
<li>Harbor requires a <strong>load balancer or Ingress controller</strong> (Contour recommended)</li>
<li><strong>Install Contour first</strong>, then Harbor</li>
<li>After Harbor installs, add a <strong>DNS A record</strong> mapping Harbor FQDN to the Envoy ingress IP</li>
<li>Harbor as Supervisor Service = <strong>not suitable for multi-tenant</strong> environments (no tenant isolation)</li>
<li>Multi-tenant Harbor: deploy as a <strong>VKS add-on</strong> on a dedicated shared-services VKS cluster</li>
</ul>
<p><strong>external-dns (Supervisor Service):</strong></p>
<ul>
<li>Publishes DNS records for services automatically</li>
<li>If used with Harbor + Contour: include <code>source=contour-httpproxy</code> in external-dns config values</li>
</ul>
<p><strong>VCF Automation service lifecycle:</strong><br />
VCF services (Broadcom vendor services like Data Services Manager, Secret Store) are self-healing — if you accidentally delete a rights bundle or service element, VCF Automation detects and recreates it within minutes.</p>
<p><strong>VKS Add-ons — <code>vcf package</code> CLI:</strong></p>
<pre><code class="language-bash"># List available packages
vcf package available list -n tkg-system

# Get versions for a package
vcf package available get cert-manager.kubernetes.vmware.com -n tkg-system

# Install a package
vcf package install cert-manager \
  -p cert-manager.kubernetes.vmware.com \
  -v 1.18.3+vmware.1-vks.1 \
  -n package-installed \
  --values-file cert-manager-values.yaml

# Update a package
vcf package installed update cert-manager \
  -p cert-manager.kubernetes.vmware.com \
  -v 1.18.3+vmware.1-vks.1 \
  --namespace package-installed

# Delete a package
vcf package installed delete cert-manager -n package-installed -y
</code></pre>
<p><strong>Install order:</strong> <code>cert-manager</code> is a prerequisite for most packages (Contour, ExternalDNS, Prometheus, Harbor). Install cert-manager first.</p>
<p><strong>Automatically included packages (no install needed):</strong> pinniped, Antrea, kapp-controller, gateway-api, secret-gen-controller, vSphere-pv-csi, metrics-server, vSphere-cpi.</p>
<p><strong>Air-gapped add-ons:</strong><br />
Use <code>imgpkg</code> to copy packages to internal Harbor, then update <code>AddonRepository</code> YAML to point to private registry.</p>
<p><strong>Checking reconciliation status:</strong></p>
<pre><code class="language-bash">kubectl get app cert-manager -n package-installed -o yaml
kubectl get addonrepositoryinstall -n vmware-system-vks-public
</code></pre>
<h3>Exam Decision Points</h3>
<ul>
<li>Harbor as Supervisor Service = <strong>single-tenant only</strong> (no multi-tenant isolation)</li>
<li>Multi-tenant Harbor = <strong>VKS add-on</strong> on dedicated shared-services VKS cluster</li>
<li><code>cert-manager</code> must be installed <strong>before</strong> most other packages</li>
<li><strong>Contour must be installed before Harbor</strong> (as Supervisor Service)</li>
<li>Harbor post-install: <strong>DNS A record</strong> required for FQDN → Envoy IP</li>
<li>VCF service deleted accidentally = <strong>auto-recreated</strong> within minutes</li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.32 — Perform Rolling Updates and Configuration Changes on VKS Clusters</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>VKS cluster updates use a rolling replacement model: new nodes are provisioned, old nodes are drained and deleted one at a time.</p>
<p><strong>Rolling update stages (in order):</strong><br />
1. Add-ons update<br />
2. Control plane nodes update (all control plane nodes first)<br />
3. Worker nodes update (one node at a time, Zone-A first for multi-nodepool)</p>
<p><strong>What triggers a rolling update:</strong></p>
<table>
<thead>
<tr>
<th>Trigger</th>
<th>Type</th>
</tr>
</thead>
<tbody>
<tr>
<td>Change VKr (Kubernetes release) version</td>
<td>Manual</td>
</tr>
<tr>
<td>Change VM class</td>
<td>Manual</td>
</tr>
<tr>
<td>Change storage class</td>
<td>Manual</td>
</tr>
<tr>
<td>Supervisor/vSphere Namespace update</td>
<td>System-initiated</td>
</tr>
<tr>
<td>VKS Controller update (new manifest values)</td>
<td>System-initiated</td>
</tr>
<tr>
<td>VirtualMachineImage renamed/replaced</td>
<td>System-initiated</td>
</tr>
<tr>
<td>New image added to Content Library</td>
<td>❌ Does NOT trigger rolling update</td>
</tr>
</tbody>
</table>
<p><strong>Mechanics:</strong></p>
<ul>
<li>New node is provisioned → comes online with target version → old node is marked for deletion → all pods drained → old node deleted</li>
<li><strong>PodDisruptionBudgets (PDBs)</strong> are respected — node is cordoned but not deleted until PDB allows eviction</li>
<li><strong>Standalone pods</strong> (not in a Deployment/ReplicaSet) are <strong>deleted</strong> during node drain — use Deployments for all workloads</li>
</ul>
<p><strong>Rolling update procedure (v1alpha3 / TanzuKubernetesCluster API):</strong></p>
<pre><code class="language-bash"># Check current state and available updates
kubectl get tanzukubernetescluster
kubectl get cluster CLUSTER-NAME -o json | jq '.status.conditions[] | select(.type==&quot;UpdatesAvailable&quot;) | .message'

# Edit the cluster manifest (change tkr.reference.name in both controlPlane and nodePools)
kubectl edit tanzukubernetescluster/CLUSTER-NAME

# Monitor progress
kubectl get tanzukubernetescluster -w
kubectl get cluster,kubeadmcontrolplane,machinedeployment
</code></pre>
<p><strong>v1beta2 (ClusterClass API):</strong> Edit <code>topology.classRef.version</code> or <code>topology.version</code> field.</p>
<p><strong>API version summary:</strong></p>
<table>
<thead>
<tr>
<th>API</th>
<th>Resource</th>
<th>Edit Command</th>
</tr>
</thead>
<tbody>
<tr>
<td>v1alpha3</td>
<td>tanzukubernetescluster</td>
<td><code>kubectl edit tanzukubernetescluster/NAME</code></td>
</tr>
<tr>
<td>v1beta1/v1beta2</td>
<td>cluster (ClusterClass)</td>
<td><code>kubectl edit cluster/NAME</code></td>
</tr>
</tbody>
</table>
<p><strong>Multi-nodepool update order:</strong> Control plane → Zone-A nodepool first → other nodepools. One worker node rolled at a time — creating extra identical nodepools does NOT speed up upgrades.</p>
<p><strong>Check cluster readiness before updating:</strong></p>
<pre><code class="language-bash">kubectl get cluster CLUSTER-NAME -o json | jq '.status.conditions[] | select(.type==&quot;Ready&quot;) | .status'
# Must return &quot;True&quot;
</code></pre>
<h3>Exam Decision Points</h3>
<ul>
<li>Control plane nodes updated <strong>before</strong> workers</li>
<li>PDBs are <strong>respected</strong> — node is cordoned but not deleted if PDB blocks eviction</li>
<li>Standalone pods = <strong>deleted</strong> during node drain (not restarted)</li>
<li>Adding Content Library images = <strong>does NOT trigger</strong> rolling update</li>
<li>Creating extra nodepools to speed upgrades = <strong>does not work</strong>; use PDBs for protection</li>
<li>Zone-A nodepool updates <strong>first</strong> in multi-nodepool clusters</li>
<li>Cluster shows <code>READY: False</code> during update = <strong>expected</strong></li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.33 — Manage Package Repositories, Standard Packages, Registry Secrets, and Private Registries</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p><strong>AddonRepository (VKS 3.5+):</strong><br />
Replaces the older method of package management. An <code>AddonRepositoryInstall</code> custom resource manages the package repository for the entire Supervisor (not per-cluster).</p>
<pre><code class="language-bash"># Check addon repository status
kubectl get addonrepositoryinstall -n vmware-system-vks-public

# List all available add-ons
kubectl get addons -n vmware-system-vks-public

# List add-on releases (versions)
kubectl get addonreleases -n vmware-system-vks-public
</code></pre>
<blockquote>
<p>⚠️ <strong>Never modify the <code>package-offerings</code> annotation</strong> on an AddonRepositoryInstall — this is managed by the system and will be overwritten.</p>
</blockquote>
<p><strong>Registry Secrets (SecretExport for cross-namespace):</strong><br />
When a VKS cluster needs to pull images from a private registry, you create a Kubernetes Secret with the registry credentials. For secrets to be usable across namespaces, use the <code>SecretExport</code> resource (from the secret-gen-controller package, which is auto-installed).</p>
<pre><code class="language-yaml"># Create registry secret in source namespace
kubectl create secret docker-registry my-registry-secret \
  --docker-server=harbor.vcf.lab \
  --docker-username=robot$myapp \
  --docker-password=&lt;token&gt; \
  -n source-namespace

# SecretExport to allow cross-namespace access
apiVersion: secretgen.carvel.dev/v1alpha1
kind: SecretExport
metadata:
  name: my-registry-secret
  namespace: source-namespace
spec:
  toNamespaces:
  - target-namespace
</code></pre>
<p><strong>Registering a private container registry (vSphere Client):</strong><br />
<code>vSphere Client &gt; Supervisor Management &gt; Container Registries &gt; Add Registry</code></p>
<p>This registers the registry at the Supervisor level — VKS clusters in that Supervisor can then reference it.</p>
<p><strong>Air-gapped package management:</strong><br />
Use <code>imgpkg</code> to copy package bundle images from internet-accessible registry to internal Harbor:</p>
<pre><code class="language-bash"># Copy package bundle to internal registry
imgpkg copy \
  -b projects.registry.vmware.com/tkg/packages/standard/repo:v2.2.0 \
  --to-repo harbor.vcf.lab/tkg/packages/standard/repo \
  --cosign-signatures

# For VKS bundles specifically (--cosign-signatures required)
imgpkg copy \
  -b &lt;vks-bundle-source&gt; \
  --to-repo &lt;internal-registry&gt;/vks \
  --cosign-signatures
</code></pre>
<p>Then update <code>AddonRepository</code> to point to the internal registry URL.</p>
<h3>Exam Decision Points</h3>
<ul>
<li><code>AddonRepositoryInstall</code> is <strong>global to the Supervisor</strong> (not per VKS cluster)</li>
<li>Never modify <code>package-offerings</code> annotation — system-managed</li>
<li><code>SecretExport</code> = cross-namespace secret sharing (from secret-gen-controller)</li>
<li>Private registry at Supervisor level = registered via <strong>vSphere Client &gt; Supervisor Management</strong></li>
<li>Air-gapped VKS bundles require <code>--cosign-signatures</code> flag with <code>imgpkg</code></li>
</ul>
</div>
</details>

---

<details>
<summary><strong>4.34 — Upgrade a Supervisor Service or VKS Cluster</strong></summary>
<div class="details-body">
<h3>Key Concepts</h3>
<p>There are three distinct upgrade types in this objective: Supervisor Services, VKS cluster Kubernetes version, and Supervisor itself.</p>
<p><strong>Critical distinction: Registration ≠ Upgrade</strong><br />
When you add a new version of a Supervisor Service to vCenter, that is a <strong>registration</strong> step — not an upgrade. The upgrade is a separate action applied to installed instances.</p>
<p><strong>Supervisor Service upgrade methods:</strong></p>
<table>
<thead>
<tr>
<th>Method</th>
<th>Description</th>
<th>Use For</th>
</tr>
</thead>
<tbody>
<tr>
<td>Synchronous</td>
<td>Upgrade happens immediately in the current operation</td>
<td>Simple, in-place upgrades</td>
</tr>
<tr>
<td>Asynchronous Public</td>
<td>Upgrade pulls images from public registry</td>
<td>Connected environments</td>
</tr>
<tr>
<td>Asynchronous Private</td>
<td>Upgrade pulls from internal registry</td>
<td>Air-gapped environments</td>
</tr>
</tbody>
</table>
<p><strong>VKS cluster upgrade:</strong><br />
Covered in 4.32 (rolling update). Upgrading the Kubernetes version = change <code>tkr.reference.name</code> or <code>topology.version</code> in the cluster manifest.</p>
<p><strong>Upgrade blocking conditions:</strong></p>
<table>
<thead>
<tr>
<th>Condition</th>
<th>Effect</th>
</tr>
</thead>
<tbody>
<tr>
<td>WARNING during upgrade check</td>
<td><strong>Non-blocking</strong> — upgrade proceeds</td>
</tr>
<tr>
<td>ERROR during upgrade check</td>
<td><strong>Blocking</strong> — upgrade will not proceed until resolved</td>
</tr>
</tbody>
</table>
<p><strong>Cannot downgrade:</strong> Once a Supervisor Service or VKS cluster is upgraded to a newer version, you cannot roll it back to an older version.</p>
<p><strong>VCF Operations for Logs — special case:</strong><br />
Unlike other VCF Operations components, <strong>VCF Operations for Logs cannot be upgraded in-place — it must be redeployed</strong>. Plan accordingly when scheduling fleet upgrades.</p>
<p><strong>NSX host components with ESX upgrade:</strong><br />
NSX host components (NSX VIBs on ESXi hosts) are bundled with the ESX upgrade lifecycle. They upgrade as part of the ESX host upgrade via SDDC Manager — not as a separate standalone NSX operation.</p>
<p><strong>imgpkg with --cosign-signatures for VKS bundles:</strong><br />
When copying VKS upgrade bundles to an air-gapped registry, the <code>--cosign-signatures</code> flag is required to copy the cosign signature files alongside the images (for verification integrity).</p>
<h3>Exam Decision Points</h3>
<ul>
<li>Registering a new service version ≠ upgrading installed instances (two separate steps)</li>
<li>WARNING during upgrade = <strong>non-blocking</strong>; ERROR = <strong>blocking</strong></li>
<li>No downgrade path for Supervisor Services or VKS clusters</li>
<li>VCF Operations for Logs = <strong>must redeploy</strong> (cannot upgrade in-place)</li>
<li>NSX host VIBs upgrade with <strong>ESX host upgrade</strong> via SDDC Manager LCM</li>
<li>Air-gapped VKS upgrade bundles: <code>imgpkg copy --cosign-signatures</code> required</li>
</ul>
</div>
</details>

---

*Previous: [VCF Automation — Objectives 4.19–4.29]({% post_url 2026-03-20-vcap-admin-vcf9-s3-automation %}) · Next: [VCF Operations & Fleet — Objectives 4.35–4.47]({% post_url 2026-03-20-vcap-admin-vcf9-s5-operations %})*
