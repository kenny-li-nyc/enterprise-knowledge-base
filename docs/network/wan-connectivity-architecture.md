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

## 3. Site-to-site VPN (IPsec tunnels, dynamic vs. static)

### Technical Definition
Site-to-site VPNs are secure, encrypted tunnels established between two or more network gateways (e.g., branch routers, firewalls) to provide private connectivity over untrusted public networks. Static VPNs involve manually configured, point-to-point tunnels with fixed peer IP addresses and security parameters. Dynamic VPNs, such as DMVPN (Dynamic Multipoint VPN) or GETVPN (Group Encrypted Transport VPN), utilize automated discovery and signaling protocols to establish tunnels on-demand, enabling scalable, full-mesh or hub-and-spoke topologies without the administrative burden of managing thousands of static peer configurations.

### Underlying Mechanism
The mechanism relies on the IPsec protocol suite, which provides authentication (AH or ESP), encryption (AES), and integrity (SHA). IKEv2 (Internet Key Exchange version 2) is the standard for tunnel negotiation, handling the exchange of security associations (SAs) and key management. Static tunnels use crypto-maps or Virtual Tunnel Interfaces (VTIs) to bind traffic to a specific peer. Dynamic VPNs introduce additional protocols: DMVPN uses NHRP (Next Hop Resolution Protocol) to map virtual tunnel IPs to physical public IPs, allowing spokes to dynamically discover each other and build direct tunnels. GETVPN uses a Key Server to distribute group keys, allowing all members of a group to decrypt traffic without the need for point-to-point tunnels, effectively treating the underlying network as a secure, encrypted fabric. Cryptographic validation is handled via PKI, as referenced in Section 3.3, ensuring that tunnel endpoints are trusted.

[DIAGRAM: Flowchart illustrating the difference between static point-to-point tunnels and dynamic DMVPN hub-and-spoke topology]

### Why It Exists
Site-to-site VPNs exist to provide secure, cost-effective connectivity between geographically dispersed sites. Static VPNs are suitable for small-scale deployments where the number of sites is limited and the topology is simple. However, as the number of sites grows, static configurations become unmanageable, leading to "configuration sprawl" and increased risk of human error. Dynamic VPNs exist to solve this scalability problem, providing a flexible, automated framework that can support thousands of sites while maintaining the security and performance required for enterprise workloads.

### Enterprise / Banking Reality
In Tier-1 banking, site-to-site VPNs are critical for connecting retail branches, ATMs, and remote offices to the central datacenter. We prioritize dynamic VPN architectures like DMVPN or GETVPN for branch connectivity, as they allow us to scale to thousands of sites with minimal manual intervention. We enforce strict security policies, requiring AES-256 encryption and IKEv2 with certificate-based authentication. We also implement "crypto-agility," ensuring that we can update our encryption algorithms and keys without disrupting connectivity. For high-security environments, we often layer VPNs over private circuits to provide "defense-in-depth," ensuring that even if the private circuit is compromised, the data remains encrypted.

### Operational Considerations
Operationalizing site-to-site VPNs requires rigorous monitoring of tunnel status, rekeying intervals, and MTU/MSS settings. MTU/MSS clamping is particularly important, as the overhead of IPsec encapsulation can lead to fragmentation and performance issues if not properly managed. Administrators must also ensure that PKI certificates are renewed before they expire to prevent catastrophic tunnel failures.
[CLI: Command to verify IPsec tunnel status and security associations]
[CLI: Command to inspect NHRP registration and tunnel mapping in a DMVPN environment]
[CLI: Command to configure MSS clamping on a tunnel interface to prevent fragmentation]

### Common Misconceptions
!!! warning
    A common misconception is that "VPN is just a tunnel." In reality, a VPN is a comprehensive security framework that requires careful management of keys, certificates, and policies. Another error is assuming that static VPNs are "simpler" than dynamic ones; while they are easier to understand initially, they become exponentially more complex and fragile as the network scales, making them a poor choice for large-scale enterprise deployments.

### Interview Angle
1. Question: How do you handle the MTU/MSS issues that often plague IPsec VPN deployments?
   Answer: We implement MSS clamping on all tunnel interfaces, setting the MSS value to a size that accounts for the IPsec overhead (typically 1360-1400 bytes). This ensures that TCP segments are sized correctly before they are encapsulated, preventing fragmentation and the associated performance degradation. We also perform path MTU discovery (PMTUD) where possible, but we rely on MSS clamping as the primary defense.
2. Question: Explain the trade-offs between DMVPN and GETVPN for a large-scale branch network.
   Answer: DMVPN is a hub-and-spoke or dynamic mesh architecture that is excellent for connecting branches over the internet, as it handles NAT traversal and dynamic addressing well. GETVPN is a group-based encryption architecture that is best suited for private, MPLS-based networks where you want to encrypt traffic without the overhead of point-to-point tunnels. We choose based on the underlying transport and the specific requirements for topology and scalability.
3. Question: How do you ensure crypto-agility in your VPN architecture?
   Answer: We use a centralized PKI and policy management system to push updated security policies and certificates to all VPN gateways. This allows us to rotate keys, update encryption algorithms, and revoke compromised certificates across the entire network in a coordinated and automated manner, ensuring that we can respond quickly to new threats or vulnerabilities.

### Related Concepts
- Section 3.3: Applied PKI & Cryptographic Chains (for tunnel authentication context)
- Section 4.2.2: SD-WAN architecture & overlay design
- Section 4.2.5: QoS & bandwidth management across WAN links

## 4. Circuit diversity & last-mile redundancy

### Technical Definition
Circuit diversity and last-mile redundancy refer to the architectural strategy of ensuring that a site's connectivity to the WAN is not dependent on a single physical path or a single service provider. This involves provisioning multiple circuits—often from different carriers—that enter the premises through physically separate points of entry (POEs) and traverse distinct physical paths (conduits) to the local exchange or central office. The goal is to eliminate single points of failure in the "last mile," which is statistically the most vulnerable segment of the WAN.

### Underlying Mechanism
The mechanism relies on physical infrastructure planning and logical edge configuration. Physically, this requires diverse fiber builds, where primary and secondary circuits are routed through separate conduits, manholes, and potentially different central offices (COs) or points of presence (POPs). Logically, the edge routers are dual-homed, with each circuit terminating on a separate physical interface or, ideally, a separate physical router (chassis diversity). The routing control plane (e.g., BGP or SD-WAN) is configured to detect link failures and automatically steer traffic to the redundant path. This often involves BFD (Bidirectional Forwarding Detection) for sub-second failure detection, ensuring that the transition between circuits is seamless and transparent to the application layer.

[DIAGRAM: Flowchart illustrating physically diverse fiber paths entering a building through separate POEs]

### Why It Exists
Circuit diversity exists to mitigate the risk of "backhoe fade"—the accidental severing of fiber cables during construction—and carrier-specific outages. In a standard single-homed configuration, a single physical failure in the last mile results in total site isolation. By implementing diversity, the network can survive the loss of a primary circuit, ensuring that critical business operations continue without interruption. This is a fundamental requirement for high-availability infrastructure, particularly in environments where downtime carries significant financial or regulatory penalties.

### Enterprise / Banking Reality
In Tier-1 banking, circuit diversity is not just a best practice; it is a regulatory mandate. Frameworks like DORA (Digital Operational Resilience Act) and FFIEC guidelines require banks to demonstrate resilience against physical infrastructure failures. We perform rigorous "physical path audits" to verify that our primary and secondary circuits are truly diverse and do not share common points of failure (e.g., the same conduit or manhole). For critical sites like data centers and major trading hubs, we often require "triple-homing" or even diverse entry points into the building to ensure that we can survive even catastrophic physical damage to the facility's entry infrastructure.

### Operational Considerations
Operationalizing circuit diversity requires ongoing validation and testing. Administrators must perform regular "failover tests" to ensure that the secondary circuit is correctly configured and capable of carrying the full production load. Monitoring tools should track the health of both circuits independently and alert on any degradation or failure, even if the site remains online via the redundant path.
[CLI: Command to verify the status and traffic statistics of the primary and secondary WAN interfaces]
[CLI: Command to simulate a circuit failure to test failover behavior]
[CLI: Command to check BFD session status for sub-second failure detection]

### Common Misconceptions
!!! warning
    A common misconception is that having two circuits from two different carriers automatically provides diversity. If both carriers use the same physical conduit or enter the building through the same POE, they are not diverse; a single construction accident can take down both. Another error is assuming that "active/standby" is the only way to use redundant circuits; with SD-WAN, we can actively utilize both circuits simultaneously, maximizing bandwidth and improving performance.

### Interview Angle
1. Question: How do you verify that your primary and secondary circuits are truly physically diverse?
   Answer: We require "as-built" documentation and physical path maps from our carriers. We also conduct periodic physical audits, where we inspect the building entry points and conduit paths. If we cannot verify diversity, we treat the circuits as non-diverse and plan for additional redundancy, such as satellite or LTE/5G backups.
2. Question: What is the difference between "carrier diversity" and "path diversity," and why do both matter?
   Answer: Carrier diversity protects against outages caused by the service provider's core network or operational errors. Path diversity protects against physical damage to the local loop (the last mile). Both are necessary; carrier diversity alone is insufficient if both circuits share the same fiber conduit, and path diversity alone is insufficient if both circuits are managed by the same carrier's core network.
3. Question: How do you handle the failover process to ensure that it doesn't cause application-level issues?
   Answer: We use BFD for sub-second failure detection, which allows the routing protocol to converge almost instantly. We also ensure that our QoS policies are consistent across both circuits, so that traffic prioritization is maintained during failover. Finally, we perform regular, scheduled failover testing during maintenance windows to validate that the application stack handles the transition gracefully.

### Related Concepts
- Section 4.2.1: MPLS vs. internet-based transport
- Section 4.2.2: SD-WAN architecture & overlay design
- Section 4.2.5: QoS & bandwidth management across WAN links
