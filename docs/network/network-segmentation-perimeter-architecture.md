# 4.3 Network Segmentation & Perimeter Architecture

## 1. Firewall zone architecture (north-south vs. east-west)

### Technical Definition
Firewall zone architecture is the strategic partitioning of a network into distinct security domains, each with defined trust levels and access policies. North-south traffic refers to communication between the internal network and external entities (e.g., the internet, third-party partners), typically traversing the perimeter firewall. East-west traffic refers to lateral communication between workloads, servers, or segments within the internal network. This architecture dictates where and how security inspection is applied to mitigate unauthorized access and lateral movement.

### Underlying Mechanism
The mechanism relies on stateful inspection engines and policy lookup pipelines within firewalls or network virtual appliances (NVAs). For north-south traffic, the firewall acts as a gateway, inspecting packets against a set of rules that define allowed ingress and egress flows, often utilizing NAT (Network Address Translation) to hide internal topology. For east-west traffic, inspection is often performed by distributed firewalls or internal segmentation gateways. When a packet enters the inspection pipeline, the engine performs a 5-tuple lookup (source IP, destination IP, source port, destination port, protocol) against the state table. If the flow is new, it is evaluated against the security policy; if allowed, a state entry is created, and subsequent packets in the flow are processed at wire speed using hardware-accelerated lookups. This process is distinct from the basic L3 routing mechanics described in Section 4.1, as it adds a layer of deep packet inspection and stateful tracking to every flow.

[DIAGRAM: Flowchart illustrating the traffic flow paths for north-south vs. east-west traffic through perimeter and internal firewalls]

### Why It Exists
This architecture exists to enforce the principle of least privilege at the network layer. Historically, networks were flat, allowing any host to communicate with any other host, which facilitated rapid lateral movement for attackers. By segmenting the network into zones, we create "choke points" where security policies can be enforced. North-south segmentation protects the enterprise from external threats, while east-west segmentation contains potential breaches, preventing an attacker who has compromised a single web server from easily moving to the database or core banking systems.

### Enterprise / Banking Reality
In Tier-1 banking, zone architecture is a critical component of our defense-in-depth strategy. We implement a "hardened perimeter" for north-south traffic, utilizing high-throughput, next-generation firewalls (NGFWs) that perform full-stack inspection, including TLS decryption. For east-west traffic, we employ a "zero-trust" approach, where even internal traffic between application tiers is inspected. This is essential for PCI-DSS compliance, as we must demonstrate that cardholder data environments (CDE) are isolated from non-CDE segments. We treat every zone boundary as a potential security perimeter, ensuring that no traffic crosses these boundaries without explicit, audited policy approval.

### Operational Considerations
Operationalizing zone architecture requires rigorous policy management and lifecycle control. Administrators must ensure that firewall rules are regularly audited to remove stale or overly permissive rules. Monitoring tools should track traffic patterns across zones to identify anomalies that might indicate a breach or a misconfiguration.
[CLI: Command to verify firewall policy rules for a specific zone-to-zone flow]
[CLI: Command to inspect the state table for active connections between internal segments]
[CLI: Command to monitor firewall throughput and resource utilization]

### Common Misconceptions
!!! warning
    A common misconception is that a strong north-south perimeter is sufficient for modern security. In reality, most sophisticated attacks involve lateral movement, making east-west segmentation equally, if not more, important. Another error is assuming that "internal" traffic is inherently trusted; in a zero-trust architecture, all traffic must be treated as potentially malicious, regardless of its origin.

### Interview Angle
1. Question: How do you balance the need for granular east-west segmentation with the operational complexity of managing thousands of firewall rules?
   Answer: We utilize automated policy orchestration tools that integrate with our CI/CD pipelines. Instead of manual rule creation, we define security policies as code, which are then automatically pushed to the firewalls. We also use "group-based" policies rather than IP-based rules, which simplifies management and ensures that policies remain consistent even as workloads scale or move.
2. Question: Explain the architectural trade-offs between using a centralized firewall cluster versus distributed, host-based firewalls for east-west traffic.
   Answer: Centralized firewalls provide a single point of control and easier auditing, but they can become a performance bottleneck and create "hair-pinning" traffic patterns. Distributed, host-based firewalls provide better performance and scalability, as inspection happens at the source, but they are more complex to manage and audit. We use a hybrid approach: centralized firewalls for major zone boundaries and host-based firewalls for microsegmentation within application tiers.
3. Question: How do you ensure that your firewall policies are compliant with PCI-DSS requirements for CDE isolation?
   Answer: We implement a "deny-all" default policy for all traffic entering or leaving the CDE zone. We then explicitly permit only the necessary flows, which are documented and justified. We perform regular, automated audits of these rules to ensure that no unauthorized paths have been created, and we use network segmentation to physically or logically separate the CDE from the rest of the network, as referenced in Section 2.6 for application sandbox context.

### Related Concepts
- Section 4.1: L3 routing protocols & Inter-VLAN routing (for structural routing context)
- Section 2.6: App Control & WDAC Hardening (for application sandbox context)
- Section 4.3.2: DMZ design patterns
