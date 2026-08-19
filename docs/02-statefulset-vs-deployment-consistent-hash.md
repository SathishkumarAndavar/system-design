# StatefulSet vs Deployment & Consistent Hashing

Summary

- **Deployment:** For stateless, replaceable pods; replicas are identical and schedulable anywhere. Use for horizontally scalable microservices.
- **StatefulSet:** Stable network IDs, stable persistent volumes per pod, and ordered lifecycle (start/stop). Use for stateful services that require stable storage and identity (databases, Kafka brokers, ZooKeeper).

Consistent hashing strategies

- **Client-side consistent hashing (hash ring):** Clients compute key→node using a shared ring metadata. Works well with `Deployment` where pods are fungible and can be resharded.
- **Proxy/sidecar consistent-hash:** An edge proxy (Envoy, HAProxy) performs consistent-hash routing; clients communicate with the proxy which routes to the backend pod.
- **StatefulSet + ordinal mapping:** Map shards to pod ordinals (pod-0 → shard-0). Use when per-pod storage must be preserved and identity matters.

Mermaid diagram

```mermaid
flowchart LR
  subgraph ClientSide
    Client -->|hash(key)| ClientRouter
    ClientRouter --> PodA[Pod A (Deployment)]
    ClientRouter --> PodB[Pod B (Deployment)]
  end
  subgraph Stateful
    Client --> Proxy
    Proxy --> ss-0[StatefulPod-0]
    Proxy --> ss-1[StatefulPod-1]
  end
```

Guidance

- Use `Deployment` + client-side or proxy hashing for elastic, easily reshardable services.
- Use `StatefulSet` when pod identity and local disk durability are required.
- Use an external metadata service (service discovery, ring metadata) to coordinate topology changes and reduce client complexity.

---
