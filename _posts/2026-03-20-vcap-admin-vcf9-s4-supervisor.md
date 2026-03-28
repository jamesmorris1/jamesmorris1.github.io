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

<details markdown="1">
<summary><strong>4.30 — Deploy and Configure a Supervisor Cluster</strong></summary>

### Key Concepts

The Supervisor is the control plane for Kubernetes workloads in VCF. It enables namespaces, VKS cluster deployment, and platform services on top of vSphere/NSX.

**Prerequisites:**
- VCF Workload Domain with NSX deployed
- NSX Tier-0 and Tier-1 gateways configured
- NSX Edge Cluster available
- Content Library with Supervisor release images (VCF 9: CDN subscribed library required)
- vSphere Distributed Switch (vDS) for Supervisor networking

**Deployment path:**
`vSphere Client > Workload Management > Supervisor Clusters > Enable`

Wizard steps:
1. Select vSphere cluster to enable as Supervisor
2. Select vDS and configure Supervisor networking (Supervisor Control Plane IP, Ingress CIDR, Egress CIDR)
3. Select NSX configuration (T0 gateway, Edge Cluster)
4. Select Content Library (must contain Supervisor release images)
5. Configure API server VIP (Supervisor Control Plane endpoint)
6. Set storage policies for etcd and Supervisor control plane VMs

**Supervisor networking in VCF 9:**
In VCF 9, the Supervisor uses NSX for networking. The Supervisor Control Plane VMs (3 VMs: one per host in the management cluster by default) get IPs from the Supervisor management network.

**Supervisor Control Plane VIP:**
The Supervisor API server is accessed via a VIP (e.g., 10.10.0.40 in lab). This is the endpoint used for `kubectl` commands against the Supervisor:
```bash
kubectl vsphere login --server=10.10.0.40 --insecure-skip-tls-verify -u administrator@vsphere.local
```

**Verifying Supervisor health:**
`vSphere Client > Workload Management > Supervisors > [Supervisor] > Summary`
All control plane nodes should be Running. If any show ConfigError or Degraded:
- Check `vmware-trustmanagement` service on SDDC Manager
- Verify spherelet VIBs are installed on ESXi hosts
- Check NSX Tier-0/Tier-1 connectivity

**Supervisor services status:**
```bash
# On SDDC Manager:
systemctl status vmware-trustmanagement
# On ESXi hosts:
esxcli software vib list | grep spherelet
```

### Exam Decision Points
- VCF 9: Supervisor images come from **CDN subscribed content library** — not bundled with vCenter
- Supervisor Control Plane = **3 VMs** deployed across management cluster hosts
- Supervisor VIP = **API server endpoint** for kubectl
- `vmware-trustmanagement` failure on SDDC Manager can block Supervisor enablement
- Spherelet VIBs must be on ESXi hosts before Supervisor can provision namespaces

</details>

---

<details markdown="1">
<summary><strong>4.31 — Install, Uninstall, and Manage Supervisor Add-on Services (e.g., Harbor, external-dns)</strong></summary>

### Key Concepts

There are **two distinct layers** of add-on services in VCF:

| Layer | What | Where managed | Examples |
|---|---|---|---|
| Supervisor Services | Cluster-wide services on the Supervisor | vCenter UI or VCF Automation Provider Portal | Harbor (Supervisor Service), Contour, external-dns |
| VKS Add-ons (Packages) | Packages on individual VKS clusters | `vcf package` CLI or `kubectl` | Harbor, cert-manager, Velero, Prometheus, Fluent-Bit, Istio |

**Supervisor Services — Install path (vCenter):**
`Hosts and Clusters > Supervisor > Configure > Supervisor Services > Add Service`

Service files: each Supervisor Service comes with:
- A `.yml` service definition file (metadata)
- A data values/configuration `.yml` file (customise before applying)

**Supervisor Services — Install path (VCF Automation Provider Portal):**
`Provider Portal > Services > Upload service file > Install instance > Publish to organisations`

**Harbor as Supervisor Service — prerequisites:**
- Harbor requires a **load balancer or Ingress controller** (Contour recommended)
- **Install Contour first**, then Harbor
- After Harbor installs, add a **DNS A record** mapping Harbor FQDN to the Envoy ingress IP
- Harbor as Supervisor Service = **not suitable for multi-tenant** environments (no tenant isolation)
- Multi-tenant Harbor: deploy as a **VKS add-on** on a dedicated shared-services VKS cluster

**external-dns (Supervisor Service):**
- Publishes DNS records for services automatically
- If used with Harbor + Contour: include `source=contour-httpproxy` in external-dns config values

**VCF Automation service lifecycle:**
VCF services (Broadcom vendor services like Data Services Manager, Secret Store) are self-healing — if you accidentally delete a rights bundle or service element, VCF Automation detects and recreates it within minutes.

**VKS Add-ons — `vcf package` CLI:**

```bash
# List available packages
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
```

**Install order:** `cert-manager` is a prerequisite for most packages (Contour, ExternalDNS, Prometheus, Harbor). Install cert-manager first.

**Automatically included packages (no install needed):** pinniped, Antrea, kapp-controller, gateway-api, secret-gen-controller, vSphere-pv-csi, metrics-server, vSphere-cpi.

**Air-gapped add-ons:**
Use `imgpkg` to copy packages to internal Harbor, then update `AddonRepository` YAML to point to private registry.

**Checking reconciliation status:**
```bash
kubectl get app cert-manager -n package-installed -o yaml
kubectl get addonrepositoryinstall -n vmware-system-vks-public
```

### Exam Decision Points
- Harbor as Supervisor Service = **single-tenant only** (no multi-tenant isolation)
- Multi-tenant Harbor = **VKS add-on** on dedicated shared-services VKS cluster
- `cert-manager` must be installed **before** most other packages
- **Contour must be installed before Harbor** (as Supervisor Service)
- Harbor post-install: **DNS A record** required for FQDN → Envoy IP
- VCF service deleted accidentally = **auto-recreated** within minutes

</details>

---

<details markdown="1">
<summary><strong>4.32 — Perform Rolling Updates and Configuration Changes on VKS Clusters</strong></summary>

### Key Concepts

VKS cluster updates use a rolling replacement model: new nodes are provisioned, old nodes are drained and deleted one at a time.

**Rolling update stages (in order):**
1. Add-ons update
2. Control plane nodes update (all control plane nodes first)
3. Worker nodes update (one node at a time, Zone-A first for multi-nodepool)

**What triggers a rolling update:**

| Trigger | Type |
|---|---|
| Change VKr (Kubernetes release) version | Manual |
| Change VM class | Manual |
| Change storage class | Manual |
| Supervisor/vSphere Namespace update | System-initiated |
| VKS Controller update (new manifest values) | System-initiated |
| VirtualMachineImage renamed/replaced | System-initiated |
| New image added to Content Library | ❌ Does NOT trigger rolling update |

**Mechanics:**
- New node is provisioned → comes online with target version → old node is marked for deletion → all pods drained → old node deleted
- **PodDisruptionBudgets (PDBs)** are respected — node is cordoned but not deleted until PDB allows eviction
- **Standalone pods** (not in a Deployment/ReplicaSet) are **deleted** during node drain — use Deployments for all workloads

**Rolling update procedure (v1alpha3 / TanzuKubernetesCluster API):**

```bash
# Check current state and available updates
kubectl get tanzukubernetescluster
kubectl get cluster CLUSTER-NAME -o json | jq '.status.conditions[] | select(.type=="UpdatesAvailable") | .message'

# Edit the cluster manifest (change tkr.reference.name in both controlPlane and nodePools)
kubectl edit tanzukubernetescluster/CLUSTER-NAME

# Monitor progress
kubectl get tanzukubernetescluster -w
kubectl get cluster,kubeadmcontrolplane,machinedeployment
```

**v1beta2 (ClusterClass API):** Edit `topology.classRef.version` or `topology.version` field.

**API version summary:**

| API | Resource | Edit Command |
|---|---|---|
| v1alpha3 | tanzukubernetescluster | `kubectl edit tanzukubernetescluster/NAME` |
| v1beta1/v1beta2 | cluster (ClusterClass) | `kubectl edit cluster/NAME` |

**Multi-nodepool update order:** Control plane → Zone-A nodepool first → other nodepools. One worker node rolled at a time — creating extra identical nodepools does NOT speed up upgrades.

**Check cluster readiness before updating:**
```bash
kubectl get cluster CLUSTER-NAME -o json | jq '.status.conditions[] | select(.type=="Ready") | .status'
# Must return "True"
```

### Exam Decision Points
- Control plane nodes updated **before** workers
- PDBs are **respected** — node is cordoned but not deleted if PDB blocks eviction
- Standalone pods = **deleted** during node drain (not restarted)
- Adding Content Library images = **does NOT trigger** rolling update
- Creating extra nodepools to speed upgrades = **does not work**; use PDBs for protection
- Zone-A nodepool updates **first** in multi-nodepool clusters
- Cluster shows `READY: False` during update = **expected**

</details>

---

<details markdown="1">
<summary><strong>4.33 — Manage Package Repositories, Standard Packages, Registry Secrets, and Private Registries</strong></summary>

### Key Concepts

**AddonRepository (VKS 3.5+):**
Replaces the older method of package management. An `AddonRepositoryInstall` custom resource manages the package repository for the entire Supervisor (not per-cluster).

```bash
# Check addon repository status
kubectl get addonrepositoryinstall -n vmware-system-vks-public

# List all available add-ons
kubectl get addons -n vmware-system-vks-public

# List add-on releases (versions)
kubectl get addonreleases -n vmware-system-vks-public
```

> ⚠️ **Never modify the `package-offerings` annotation** on an AddonRepositoryInstall — this is managed by the system and will be overwritten.

**Registry Secrets (SecretExport for cross-namespace):**
When a VKS cluster needs to pull images from a private registry, you create a Kubernetes Secret with the registry credentials. For secrets to be usable across namespaces, use the `SecretExport` resource (from the secret-gen-controller package, which is auto-installed).

```yaml
# Create registry secret in source namespace
kubectl create secret docker-registry my-registry-secret \
  --docker-server=harbor.vcf.lab \
  --docker-username=robot$myapp \
  --docker-password=<token> \
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
```

**Registering a private container registry (vSphere Client):**
`vSphere Client > Supervisor Management > Container Registries > Add Registry`

This registers the registry at the Supervisor level — VKS clusters in that Supervisor can then reference it.

**Air-gapped package management:**
Use `imgpkg` to copy package bundle images from internet-accessible registry to internal Harbor:

```bash
# Copy package bundle to internal registry
imgpkg copy \
  -b projects.registry.vmware.com/tkg/packages/standard/repo:v2.2.0 \
  --to-repo harbor.vcf.lab/tkg/packages/standard/repo \
  --cosign-signatures

# For VKS bundles specifically (--cosign-signatures required)
imgpkg copy \
  -b <vks-bundle-source> \
  --to-repo <internal-registry>/vks \
  --cosign-signatures
```

Then update `AddonRepository` to point to the internal registry URL.

### Exam Decision Points
- `AddonRepositoryInstall` is **global to the Supervisor** (not per VKS cluster)
- Never modify `package-offerings` annotation — system-managed
- `SecretExport` = cross-namespace secret sharing (from secret-gen-controller)
- Private registry at Supervisor level = registered via **vSphere Client > Supervisor Management**
- Air-gapped VKS bundles require `--cosign-signatures` flag with `imgpkg`

</details>

---

<details markdown="1">
<summary><strong>4.34 — Upgrade a Supervisor Service or VKS Cluster</strong></summary>

### Key Concepts

There are three distinct upgrade types in this objective: Supervisor Services, VKS cluster Kubernetes version, and Supervisor itself.

**Critical distinction: Registration ≠ Upgrade**
When you add a new version of a Supervisor Service to vCenter, that is a **registration** step — not an upgrade. The upgrade is a separate action applied to installed instances.

**Supervisor Service upgrade methods:**

| Method | Description | Use For |
|---|---|---|
| Synchronous | Upgrade happens immediately in the current operation | Simple, in-place upgrades |
| Asynchronous Public | Upgrade pulls images from public registry | Connected environments |
| Asynchronous Private | Upgrade pulls from internal registry | Air-gapped environments |

**VKS cluster upgrade:**
Covered in 4.32 (rolling update). Upgrading the Kubernetes version = change `tkr.reference.name` or `topology.version` in the cluster manifest.

**Upgrade blocking conditions:**

| Condition | Effect |
|---|---|
| WARNING during upgrade check | **Non-blocking** — upgrade proceeds |
| ERROR during upgrade check | **Blocking** — upgrade will not proceed until resolved |

**Cannot downgrade:** Once a Supervisor Service or VKS cluster is upgraded to a newer version, you cannot roll it back to an older version.

**VCF Operations for Logs — special case:**
Unlike other VCF Operations components, **VCF Operations for Logs cannot be upgraded in-place — it must be redeployed**. Plan accordingly when scheduling fleet upgrades.

**NSX host components with ESX upgrade:**
NSX host components (NSX VIBs on ESXi hosts) are bundled with the ESX upgrade lifecycle. They upgrade as part of the ESX host upgrade via SDDC Manager — not as a separate standalone NSX operation.

**imgpkg with --cosign-signatures for VKS bundles:**
When copying VKS upgrade bundles to an air-gapped registry, the `--cosign-signatures` flag is required to copy the cosign signature files alongside the images (for verification integrity).

### Exam Decision Points
- Registering a new service version ≠ upgrading installed instances (two separate steps)
- WARNING during upgrade = **non-blocking**; ERROR = **blocking**
- No downgrade path for Supervisor Services or VKS clusters
- VCF Operations for Logs = **must redeploy** (cannot upgrade in-place)
- NSX host VIBs upgrade with **ESX host upgrade** via SDDC Manager LCM
- Air-gapped VKS upgrade bundles: `imgpkg copy --cosign-signatures` required

</details>

---

*Previous: [VCF Automation — Objectives 4.19–4.29]({% post_url 2026-03-20-vcap-admin-vcf9-s3-automation %}) · Next: [VCF Operations & Fleet — Objectives 4.35–4.47]({% post_url 2026-03-20-vcap-admin-vcf9-s5-operations %})*
