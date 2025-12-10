---
layout: default
title: "Understanding VCF Tenant Network Creation: Provider vs Organization Responsibilities"
date: 2025-12-10
categories: vcf networking nsx
---

# Understanding VCF Tenant Network Creation: Provider vs Organization Responsibilities

When deploying VMware Cloud Foundation (VCF) in a multi-tenant environment, understanding the clear separation between provider and organization responsibilities is crucial. This article breaks down the tenant network creation process, explaining how each configuration step maps to underlying NSX-T constructs.

## The Two-Tier Model

VCF's networking model operates on a clear separation of concerns:

- **Provider Network Configuration**: Infrastructure-level networking managed by the service provider
- **Organization Network Configuration**: Tenant-level networking managed by individual organizations

This separation enables true multi-tenancy while maintaining security boundaries and operational independence.

## Provider Network Configuration

### IP Spaces and IP Blocks

The foundation of VCF networking starts with IP address management through IP Spaces and IP Blocks.

**IP Spaces** act as containers for organizing IP address allocations. They define the scope and reachability characteristics of networks within VCF.

**IP Blocks** are carved from IP Spaces and represent actual allocatable IP ranges. In our example:
- IP Block: `10.10.10.32/28`
- External Reachability: `0.0.0.0/0` (publicly routable)

These IP Blocks are assigned to provider gateways and made available to organizations for external connectivity.

### Provider Gateway

The Provider Gateway is the critical north-south routing component, mapping directly to an **NSX-T Tier-0 gateway**.

**Key mappings:**
- **VCF Construct**: Provider Gateway (`sm-01-provider-gateway-01`)
- **NSX-T Construct**: Tier-0 Gateway
- **Edge Cluster**: `sa-wld01-cl01`
- **Tier-0/VRF Gateway Configuration**: Tier 0

During creation, you specify:
1. **Region**: Defines where this gateway operates
2. **Edge Cluster**: The physical edge infrastructure (`sa-wld01-cl01`) where the Tier-0 runs
3. **IP Block Assignment**: External IP range for SNAT and external connectivity

The Provider Gateway establishes BGP peering with the physical gateway, enabling routing between the VCF environment and external networks.

### Regional Network Configuration

Once configured, the provider gateway synchronizes with the edge cluster. The VCF interface displays:
- Health status
- Average CPU usage
- Average memory usage
- Organization assignments
- Associated VPCs

This synchronization ensures the edge cluster is ready to handle tenant traffic.

## Organization Network Configuration

### Organization (Org) Setup

Each organization receives an allocation from the provider's IP Space. In our example:
- **Org Name**: `sm-01-org`
- **External IP Block**: `10.10.10.32` (allocated from provider)

The organization operates within the boundaries set by the provider but has autonomy over its internal networking.

### Transit Gateway (TGW)

The Transit Gateway acts as the central routing hub for the organization, connecting VPCs and managing external connectivity.

**Key components:**
- **Name**: `default@asia-east`
- **External IP Block**: `10.10.10.32`
- **Default SNAT IP**: `10.10.10.32`
- **Private-TGW IP Block**: `192.183.237.0/24`

The TGW manages:
- SNAT for outbound traffic from private VPCs
- Routing between multiple VPCs (if applicable)
- Connection to the provider gateway via the external IP block

### Private VPC and Subnets

The Private VPC represents the organization's isolated network space, mapping to **NSX-T's VPC construct**.

**Configuration:**
- **VPC Name**: `asia-east-Default-VPC`
- **Private Subnet CIDR**: `192.173.237.0/24`
- **Region**: `asia-east`
- **Scope**: Private - VPC

Within the VPC:
- **VPC Gateway**: Connects the private subnet to the Transit Gateway
- **Subnets**: Carved from the VPC CIDR (`192.173.237.0/24`)
- **Connectivity Profiles**: Define how VMs connect (not applicable initially)

### Default VPC and Transit Gateway Relationship

The diagram shows two key IP blocks for the organization:
1. **Private-VPC Subnet**: `192.173.237.0/24` - Internal VM networking
2. **Private-TGW IP Block**: `192.183.237.0/24` - Transit Gateway infrastructure

The VPC Gateway (`192.173.237.1` typically) connects to the Transit Gateway, which then uses SNAT (`10.10.10.32`) for external connectivity.

## The Complete Network Flow

Let's trace a packet from a VM in the organization to the internet:

1. **VM sends traffic** from Private VPC (`192.173.237.0/24`)
2. **VPC Gateway** routes to Transit Gateway
3. **Transit Gateway** performs SNAT using external IP (`10.10.10.32`)
4. **Provider Gateway** (Tier-0) receives SNATed traffic
5. **BGP peering** routes traffic to physical gateway
6. **Physical Gateway** forwards to internet

Return traffic follows the inverse path, with the provider gateway routing based on the SNAT IP back to the correct organization.

## Key Takeaways

**Provider Responsibilities:**
- Configure IP Spaces and allocate IP Blocks
- Deploy and maintain Provider Gateways (Tier-0)
- Manage edge cluster infrastructure
- Establish BGP peering with physical network

**Organization Responsibilities:**
- Create and manage Transit Gateways
- Define VPC topology and subnets
- Configure SNAT policies
- Deploy workloads within private VPCs

**NSX-T Construct Mapping:**
- Provider Gateway → Tier-0 Gateway
- Transit Gateway → Central routing construct for org
- Private VPC → NSX-T VPC with isolated networking
- VPC Gateway → Connectivity between VPC and TGW

Understanding this separation allows both providers and organizations to operate independently while maintaining secure, scalable multi-tenant networking in VCF.

## Conclusion

VCF's network architecture provides a clean separation between infrastructure provider and tenant concerns. By understanding how the VCF wizard inputs map to NSX-T constructs, you can better troubleshoot issues, plan capacity, and design robust multi-tenant environments.

In future articles, I'll dive deeper into specific topics like BGP configuration, SNAT policies, and VPC networking patterns.

---

*Have questions or corrections? Connect with me on [LinkedIn](https://www.linkedin.com/in/jamemorr/).*

*Last updated: December 10, 2025*
