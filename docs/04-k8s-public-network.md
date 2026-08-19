# How Public Network Traffic Reaches Kubernetes Pods

Typical path (cloud-managed cluster)

1. DNS resolves hostname to a cloud Load Balancer IP.
2. Internet → Cloud Load Balancer (L4/L7) → Ingress Controller or `Service type=LoadBalancer`.
3. Load Balancer forwards to node(s) or directly to pod IPs (provider dependent).
4. `kube-proxy` (iptables/ipvs) or CNI routes traffic to the target pod IP.
5. Pod receives traffic on its network interface (`eth0`/veth pair).

Key service types

- `Service type=LoadBalancer`:
  - Cloud provisioner creates an external LB that forwards to nodes' NodePort(s) or directly to pod IPs.
- `Ingress`:
  - L7 routing (host/path) and TLS termination. Implemented by IngressController (NGINX, Traefik, ALB Ingress) behind the cloud LB.
- `externalTrafficPolicy`:
  - `Cluster` (default) — LB forwards to any node; kube-proxy NATs to pod IPs.
  - `Local` — preserves source IPs; only nodes with local endpoints receive external traffic.
- `HostNetwork`/`HostPort`:
  - Bypass kube-proxy, bind directly on node interface.

Mermaid diagram

```mermaid
flowchart LR
  Internet --> LB[Cloud Load Balancer]
  LB --> Ingress[Ingress Controller]
  Ingress --> Service[Service ClusterIP]
  Service --> Node[kube-proxy / ipvs]
  Node --> Pod[Pod (container)]
```

Security & performance notes

- Place WAF / CDN in front of LB for filtering and caching.
- Use mTLS, NetworkPolicies, and least-privilege service accounts for in-cluster security.
- Monitor `externalTrafficPolicy` and node endpoint distribution to avoid traffic blackholing and to preserve client IPs when required.

---
