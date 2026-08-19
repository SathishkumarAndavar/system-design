# Network Differences: AWS vs GCP vs Azure

Short summary

- **AWS**
  - VPC per region with subnets in AZs; ENI network model; Security Groups (stateful) and NACLs (stateless); features: Transit Gateway, VPC Peering, PrivateLink.
  - Kubernetes: AWS VPC CNI assigns pod IPs from VPC (ENI/secondary IPs) or alternate CNIs available. `LoadBalancer` → ELB/ALB/NLB.
- **GCP**
  - Global VPC (spans regions) with regional subnets; firewall rules applied at network level; Shared VPC; Cloud NAT for egress.
  - Kubernetes: GKE supports VPC-native alias IPs; Google Cloud Load Balancer provides global L7 and cross-region balancing.
- **Azure**
  - VNet per region (peering for cross-region); NSGs for ACLs; Azure Firewall and Virtual WAN for centralization.
  - Kubernetes: AKS supports Azure CNI or kubenet; `LoadBalancer` → Azure LB or Application Gateway.

Operational differences

- IP model & scope
  - GCP's global VPC simplifies cross-region routing.
  - AWS uses regional VPCs with peering/transit gateways for cross-VPC communication.
- Load-balancing & global routing
  - GCP: first-class global L7 that can front multi-region backends.
  - AWS: global routing via Route53 + regional LBs; PrivateLink for private endpoints.
  - Azure: Application Gateway and Azure Front Door for global routing.
- Security primitives
  - AWS: Security Groups (resource-scoped, stateful) + NACLs.
  - GCP: priority-based firewall rules, targeted by tags/service accounts.
  - Azure: NSGs at subnet/NIC and Azure Firewall for centralized rules.

Decision notes

- Pick GCP for global VPC and simplified cross-region networking.
- Pick AWS for mature private connectivity features and broad ecosystem.
- Pick Azure for Microsoft ecosystem integration and Azure-specific gateway features.

---
