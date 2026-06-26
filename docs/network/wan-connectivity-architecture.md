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

## 2. SD-WAN architecture & overlay design

### Technical Definition
SD-WAN (Software-Defined Wide Area Network) is an architectural framework that decouples the network control plane from the underlying physical transport, creating a virtualized overlay network. This overlay abstracts the physical connectivity, allowing for centralized policy management, application-aware routing, and dynamic path selection across heterogeneous transport links, including MPLS, broadband, and LTE/5G. It transforms the WAN from a collection of static, manually configured routers into a unified, software-defined fabric.

### Underlying Mechanism
The SD-WAN architecture consists of three primary planes: the management plane (orchestrator), the control plane (controller), and the data plane (edge devices). The orchestrator provides a centralized interface for policy definition and zero-touch provisioning. The controller maintains the global topology and distributes routing information to edge devices. The data plane utilizes tunnels (e.g., IPsec, VXLAN) to build the overlay, encapsulating traffic and steering it based on real-time path metrics. Edge devices perform deep packet inspection (DPI) to identify applications and apply policies based on latency, jitter, and packet loss thresholds. The forwarding plane uses these metrics to steer traffic dynamically, often utilizing BFD (Bidirectional Forwarding Detection) for sub-second failure detection and path switching.

[DIAGRAM: Flowchart illustrating the SD-WAN control plane orchestration and data plane overlay tunnel establishment]

### Why It Exists
SD-WAN exists to address the operational complexity and rigidity of traditional WAN architectures. In a traditional model, every router must be configured individually, and traffic steering is often limited to static routing or basic policy-based routing. SD-WAN simplifies this by centralizing management, automating provisioning, and enabling intelligent, application-aware traffic steering. It allows organizations to leverage lower-cost internet circuits while maintaining the performance and security required for enterprise applications, effectively turning the WAN into a flexible, software-defined asset.

### Enterprise / Banking Reality
In Tier-1 banking, SD-WAN is the primary driver for branch transformation and cloud-first strategies. We use SD-WAN to manage thousands of retail branches, enabling zero-touch provisioning (ZTP) that allows non-technical staff to deploy new sites in minutes. Centralized security policy enforcement is critical; we integrate SD-WAN with cloud-delivered security stacks (SASE) to ensure that direct internet access (DIA) from branches is as secure as traffic routed through the central datacenter. We also use SD-WAN to optimize the performance of SaaS applications (e.g., Office 365, Salesforce) by steering traffic directly to the nearest cloud entry point, bypassing the latency of backhauling traffic to the corporate core.

### Operational Considerations
Operationalizing SD-WAN requires a shift from box-by-box configuration to centralized policy management. Administrators must define global policies that are pushed to all edge devices, ensuring consistency and compliance. Monitoring is focused on the health of the overlay tunnels and the performance of the underlying transport circuits.
[CLI: Command to verify the status of SD-WAN overlay tunnels and controller connectivity]
[CLI: Command to inspect the application-aware routing policy and path selection logic]
[CLI: Command to trigger a zero-touch provisioning process for a new branch edge device]

### Common Misconceptions
!!! warning
    A common misconception is that SD-WAN is a "magic bullet" for poor-quality circuits. While SD-WAN can optimize traffic steering and mitigate the impact of minor jitter or loss, it cannot fix a fundamentally broken or congested physical circuit. Another error is assuming that SD-WAN replaces the need for robust security; while it simplifies security policy enforcement, it must be integrated with a comprehensive security architecture (e.g., SASE, firewalls) to be effective.

### Interview Angle
1. Question: How do you ensure the scalability and reliability of the SD-WAN control plane in a global banking network?
   Answer: We deploy the SD-WAN controllers in a highly available, geographically distributed cluster. We use anycast addressing for controller connectivity, ensuring that edge devices always connect to the nearest available controller. We also implement strict rate-limiting and authentication for all control plane traffic to prevent unauthorized access or DDoS attacks.
2. Question: How do you integrate SD-WAN with existing legacy MPLS networks during a migration?
   Answer: We use a phased migration approach. We initially deploy SD-WAN as an overlay on top of the existing MPLS network, allowing us to gain visibility and control without disrupting existing traffic. We then gradually introduce internet circuits and migrate traffic to the SD-WAN fabric, eventually decommissioning the MPLS circuits once the SD-WAN overlay is proven to meet our performance and reliability requirements.
3. Question: What is your strategy for securing direct internet access (DIA) from branch sites using SD-WAN?
   Answer: We enforce a "security-first" policy. All DIA traffic is routed through a cloud-delivered security stack (SASE) that includes firewalling, URL filtering, and malware inspection. We also implement local breakout policies that only allow trusted, business-critical traffic to bypass the central datacenter, ensuring that we maintain visibility and control over all internet-bound traffic.

### Related Concepts
- Section 4.2.1: MPLS vs. internet-based transport
- Section 4.2.3: Site-to-site VPN (IPsec tunnels, dynamic vs. static)
- Section 4.2.5: QoS & bandwidth management across WAN links
