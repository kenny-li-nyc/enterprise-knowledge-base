# 4.2 WAN & Connectivity Architecture

## 1. MPLS vs. internet-based transport

### Technical Definition
MPLS (Multiprotocol Label Switching) is a private, carrier-managed transport technology that uses label-based forwarding to provide deterministic, private connectivity between sites, often with strict Service Level Agreements (SLAs) for latency, jitter, and packet loss. Internet-based transport, conversely, utilizes the public internet as the underlying medium, relying on encryption protocols like IPsec to secure traffic. While MPLS provides a private, isolated backbone, internet-based transport leverages the ubiquity and cost-efficiency of public broadband, shifting the complexity of path reliability and security to the edge devices.

### Underlying Mechanism
MPLS operates by assigning labels to packets at the ingress Provider Edge (PE) router, allowing the core Provider (P) routers to switch traffic based on these labels rather than performing expensive IP routing lookups. This creates a "virtual circuit" that is logically isolated from other customers. Internet-based transport relies on standard IP routing across the public internet, where packets are subject to best-effort delivery. To secure this, edge gateways encapsulate traffic within IPsec tunnels, which provide confidentiality and integrity. These gateways utilize crypto-accelerators to handle the overhead of encryption/decryption at wire speed. Path health is managed via active probing (e.g., BFD or proprietary SD-WAN probes) that continuously measures latency, jitter, and loss across available internet circuits, allowing the edge device to dynamically steer traffic based on real-time performance metrics.

[DIAGRAM: Comparison of MPLS label-switched path vs. encrypted IPsec tunnel over public internet]

### Why It Exists
MPLS was developed to provide a reliable, private alternative to the unpredictable nature of the early internet, offering the performance characteristics of a leased line with the flexibility of a routed network. It remains the gold standard for critical, latency-sensitive applications. Internet-based transport exists to address the limitations of MPLS: high cost, long provisioning lead times, and the difficulty of scaling bandwidth in remote locations. By leveraging commodity internet, organizations can achieve massive bandwidth increases at a fraction of the cost, provided they implement robust overlay technologies to manage the inherent unreliability of the public internet.

### Enterprise / Banking Reality
In Tier-1 banking, we rarely choose between MPLS and internet; we utilize a hybrid WAN architecture. MPLS is reserved for high-value, latency-sensitive traffic—such as core banking transactions, real-time market data feeds, and inter-datacenter replication—where deterministic performance is non-negotiable. Internet-based transport is used for general corporate traffic, guest Wi-Fi, and cloud-based SaaS applications. We strictly enforce PCI-DSS compliance by ensuring that all cardholder data traversing the internet is encrypted via IPsec tunnels, as referenced in Section 3.3 for cryptographic validation. We also maintain strict carrier diversity requirements to satisfy DORA and FFIEC resilience mandates, ensuring that a single carrier outage cannot isolate a branch or datacenter.

### Operational Considerations
Operationalizing a hybrid WAN requires sophisticated monitoring to manage the disparate performance characteristics of MPLS and internet circuits. Administrators must configure policy-based routing or SD-WAN policies to steer traffic based on application requirements.
[CLI: Command to verify MPLS label distribution and path status]
[CLI: Command to monitor IPsec tunnel health and encryption statistics]
[CLI: Command to configure path-probing intervals for internet circuit monitoring]

### Common Misconceptions
!!! warning
    A common misconception is that internet-based transport is "just as good" as MPLS for all traffic types. While internet bandwidth is cheaper and more abundant, it lacks the deterministic performance guarantees of MPLS. Another error is assuming that IPsec encryption is a substitute for the physical isolation provided by MPLS; while IPsec secures the data, it does not provide the same level of protection against DDoS attacks or traffic analysis that a private MPLS circuit offers.

### Interview Angle
1. Question: How do you justify the continued use of MPLS in an era of high-speed, low-cost internet?
   Answer: MPLS provides deterministic performance and strict SLAs that the public internet cannot guarantee. For critical banking applications like real-time trading or core ledger synchronization, the cost of MPLS is an insurance policy against the unpredictability of the public internet. We use MPLS for the "must-work" traffic and internet for the "nice-to-have" traffic.
2. Question: How do you manage the security risks of using the public internet for branch connectivity?
   Answer: We treat the internet as an untrusted transport. All traffic is encapsulated in IPsec tunnels, and we implement a "zero-trust" approach at the edge, where every packet is inspected by a next-generation firewall. We also leverage SASE/ZTNA agents for remote users, as discussed in Section 2.7, to ensure that identity-based access controls are enforced regardless of the underlying transport.
3. Question: What is your strategy for ensuring carrier diversity in a hybrid WAN design?
   Answer: We mandate that primary and secondary circuits must be provided by different physical carriers and enter the building through different points of entry (POEs). We also ensure that the underlying physical infrastructure (e.g., fiber paths) is geographically diverse to prevent a single construction accident from taking down both circuits.

### Related Concepts
- Section 2.7: Remote Access & ZTNA (for client-side connectivity context)
- Section 3.3: Applied PKI & Cryptographic Chains (for tunnel authentication context)
- Section 4.2.2: SD-WAN architecture & overlay design
