# 4.1 Routing & Switching Architecture

## 1. L2 fundamentals (VLANs, trunking, spanning tree/RSTP/MSTP)

### Technical Definition
Layer 2 (L2) fundamentals encompass the protocols and mechanisms used to manage data link layer connectivity within a switched network. This includes Virtual Local Area Networks (VLANs) for logical segmentation of broadcast domains, 802.1Q trunking for carrying multiple VLANs across a single physical link, and Spanning Tree Protocol (STP) variants—specifically Rapid Spanning Tree Protocol (RSTP) and Multiple Spanning Tree Protocol (MSTP)—designed to prevent switching loops while maintaining redundant paths. These technologies collectively define the switching fabric's ability to isolate traffic and ensure loop-free topology convergence.

### Underlying Mechanism
At the hardware level, switches utilize Content Addressable Memory (CAM) tables to map MAC addresses to physical ports, enabling wire-speed frame forwarding. When a frame arrives, the switch inspects the destination MAC address; if found in the CAM table, the frame is forwarded to the appropriate egress port. If unknown, the switch floods the frame to all ports within the VLAN. TCAM (Ternary Content Addressable Memory) is employed for more complex lookups, such as Access Control Lists (ACLs) and Quality of Service (QoS) policies, allowing for simultaneous matching of multiple fields. Trunking operates by inserting a 4-byte 802.1Q tag into the Ethernet frame header, which identifies the VLAN ID, ensuring that frames remain isolated as they traverse inter-switch links. STP and its derivatives operate by exchanging Bridge Protocol Data Units (BPDUs) to elect a Root Bridge and determine the optimal path to that root. RSTP improves convergence by introducing explicit handshake mechanisms between switches, while MSTP maps multiple VLANs to a single spanning tree instance, significantly reducing the CPU overhead on switches compared to Per-VLAN Spanning Tree (PVST+) implementations.

[DIAGRAM: Flowchart illustrating the BPDU exchange process and the transition of switch ports through STP states]

### Why It Exists
L2 fundamentals exist primarily to manage broadcast domains and ensure network stability. Without VLANs, a single broadcast domain would encompass the entire network, leading to excessive broadcast traffic, security vulnerabilities, and performance degradation. Trunking allows for the efficient utilization of physical links by multiplexing traffic from multiple VLANs, which is essential for scalable network design. Spanning Tree protocols are a necessary evil in L2 networks; they exist to prevent the catastrophic broadcast storms that occur when redundant physical paths create logical loops, which would otherwise saturate the switching fabric and bring down the entire network segment.

### Enterprise / Banking Reality
In a Tier-1 banking environment, L2 design is governed by the need for strict segmentation and high availability. VLANs are the primary mechanism for enforcing PCI-DSS compliance, ensuring that Cardholder Data Environments (CDE) are logically isolated from general corporate traffic. We avoid large, flat L2 domains at all costs, preferring a "collapsed core" or "leaf-spine" architecture where L2 boundaries are kept small and localized to the access layer. MSTP is the standard for these environments, as it provides the necessary scalability to manage hundreds of VLANs without the resource exhaustion associated with per-VLAN protocols. We treat L2 as a transient transport layer, pushing L3 routing as close to the access edge as possible to minimize the blast radius of any potential L2 failure.

### Operational Considerations
Operationalizing L2 networks requires rigorous monitoring of CAM table utilization and STP topology changes. Administrators must ensure that root bridges are manually pinned to core switches to prevent suboptimal traffic paths. Monitoring tools should alert on any TCN (Topology Change Notification) events, as these indicate instability in the switching fabric.
[CLI: Command to display the current STP root bridge and port status]
[CLI: Command to verify the VLAN database and trunking configuration on a switch interface]

### Common Misconceptions
!!! warning
    A common misconception is that STP is obsolete in modern data centers. While it is true that technologies like VXLAN/EVPN or FabricPath reduce reliance on traditional STP, it remains a critical safety mechanism in the access layer and legacy environments. Another error is assuming that "more VLANs are better"; excessive VLAN sprawl increases management complexity and can lead to configuration errors that compromise security segmentation.

### Interview Angle
1. Question: How do you architect a high-availability L2 access layer in a Tier-1 banking environment while minimizing the risk of broadcast storms?
   Answer: We implement a design that pushes L3 routing to the distribution or access layer, effectively terminating L2 domains at the switch stack. We use MSTP to manage the remaining L2 segments, ensuring that root bridges are deterministic and that we have a clear, predictable topology. We also implement BPDU guard and root guard on all edge ports to prevent unauthorized switches from influencing the STP topology.
2. Question: Explain the trade-offs between RSTP and MSTP in a large-scale enterprise network.
   Answer: RSTP provides fast convergence but requires a separate spanning tree instance for every VLAN if using PVST+, which can overwhelm switch CPUs in environments with thousands of VLANs. MSTP allows us to group VLANs into instances, significantly reducing the number of active spanning tree processes and the associated control plane traffic, making it the superior choice for large-scale, multi-VLAN environments.
3. Question: How do you handle the security implications of VLAN hopping attacks in a multi-tenant banking network?
   Answer: We enforce strict port security, disable DTP (Dynamic Trunking Protocol) on all user-facing ports, and ensure that native VLANs are not used for any traffic. We also implement private VLANs (PVLANs) where necessary to isolate hosts within the same subnet, preventing lateral movement at the L2 layer.

### Related Concepts
- Section 1.1: Active Directory Sites & Subnets (for mapping logical network boundaries to physical site locations)
- Section 4.2: L3 routing protocols (OSPF, EIGRP, BGP)
- Section 4.3: Inter-VLAN routing & L3 switching

## 2. L3 routing protocols (OSPF, EIGRP, BGP) & route redistribution

### Technical Definition
Layer 3 (L3) routing protocols are the control plane mechanisms that enable routers to exchange reachability information, calculate optimal paths, and maintain a consistent view of the network topology. This includes Interior Gateway Protocols (IGPs) like OSPF (Open Shortest Path First) and EIGRP (Enhanced Interior Gateway Routing Protocol) for intra-domain routing, and the Exterior Gateway Protocol (EGP) BGP (Border Gateway Protocol) for inter-domain routing. Route redistribution is the process of injecting routing information from one protocol or autonomous system into another, allowing disparate routing domains to communicate.

### Underlying Mechanism
Routing protocols operate by maintaining a Routing Information Base (RIB) and a Forwarding Information Base (FIB). The RIB is the control plane database containing all learned routes, while the FIB is the hardware-optimized table (often stored in TCAM) used by the ASIC for wire-speed packet forwarding. OSPF uses a link-state algorithm (Dijkstra) to build a complete map of the area, flooding Link State Advertisements (LSAs) to ensure all routers have identical topology databases. EIGRP uses the DUAL (Diffusing Update Algorithm) to maintain a feasible successor, providing near-instant convergence by pre-calculating loop-free backup paths. BGP is a path-vector protocol that uses attributes (AS-Path, Local Preference, MED) to make routing decisions based on policy rather than just metric. Redistribution involves importing routes from one process into another, which requires careful manipulation of administrative distance and route tagging to prevent routing loops and suboptimal path selection.

[DIAGRAM: Flowchart illustrating the RIB-to-FIB programming process and the interaction between routing protocols and the forwarding plane]

### Why It Exists
L3 routing protocols exist to provide scalability, redundancy, and path control in large, complex networks. Unlike static routing, which is manually configured and brittle, dynamic routing protocols automatically adapt to topology changes, such as link failures or the addition of new subnets. They provide the intelligence required to route traffic across diverse physical infrastructures, ensuring that data reaches its destination efficiently. Route redistribution is necessary in heterogeneous environments where different departments, acquisitions, or service providers utilize different routing protocols, allowing for seamless connectivity across the enterprise.

### Enterprise / Banking Reality
In Tier-1 banking, routing architecture is defined by extreme stability, deterministic convergence, and strict policy enforcement. We typically utilize OSPF or EIGRP for the internal datacenter fabric, favoring OSPF for its vendor neutrality and EIGRP for its rapid convergence in Cisco-centric environments. BGP is reserved for the core backbone and external connectivity, where granular policy control is required. Redistribution is treated as a high-risk operation; we implement strict route filtering, prefix lists, and route tagging to prevent route feedback loops. We also prioritize "summarization at the boundary" to keep the global routing table lean and stable, ensuring that local flapping does not impact the entire enterprise core.

### Operational Considerations
Operationalizing L3 routing requires proactive monitoring of neighbor adjacencies, prefix counts, and CPU utilization on core routers. Administrators must ensure that routing protocols are secured with authentication (e.g., MD5 or SHA) to prevent unauthorized route injection.
[CLI: Command to verify OSPF neighbor adjacencies and database synchronization]
[CLI: Command to inspect the BGP table and verify path attributes for a specific prefix]
[CLI: Command to display the current routing table and verify the source of a specific route]

### Common Misconceptions
!!! warning
    A common misconception is that redistribution is a "set and forget" configuration. In reality, redistribution is the most common cause of routing loops and suboptimal routing in enterprise networks. Another error is assuming that "faster convergence is always better"; aggressive timers can lead to instability and CPU exhaustion during transient link flaps, which can be more damaging than a slightly slower convergence time.

### Interview Angle
1. Question: How do you mitigate the risk of routing loops when performing mutual redistribution between OSPF and EIGRP in a core banking network?
   Answer: We use route tagging to identify the source of redistributed routes. When redistributing from OSPF into EIGRP, we tag the routes; we then configure the OSPF process to deny any routes that carry that specific tag. This ensures that routes originating in OSPF are not re-injected back into OSPF from EIGRP, effectively preventing loops.
2. Question: Explain the architectural trade-offs between OSPF and EIGRP for a large-scale datacenter fabric.
   Answer: OSPF is a link-state protocol that provides a global view of the topology, which is excellent for troubleshooting but can be CPU-intensive in very large areas. EIGRP is a distance-vector protocol with link-state characteristics; it provides faster convergence through DUAL and is generally less resource-intensive, but it is historically more proprietary. In a multi-vendor environment, OSPF is the standard, while EIGRP is often preferred in pure Cisco environments for its operational simplicity and rapid convergence.
3. Question: How do you design a BGP architecture to ensure high availability for multi-homed internet connectivity?
   Answer: We use BGP attributes like Local Preference to influence outbound traffic and AS-Path Prepending or MED to influence inbound traffic. We also implement BGP communities to signal our traffic engineering policies to our upstream providers. This allows us to maintain granular control over our traffic flows and ensure that we can failover between providers without manual intervention.

### Related Concepts
- Section 1.1: Active Directory Sites & Subnets (for mapping logical network boundaries to physical site locations)
- Section 4.1: L2 fundamentals
- Section 4.3: Inter-VLAN routing & L3 switching
