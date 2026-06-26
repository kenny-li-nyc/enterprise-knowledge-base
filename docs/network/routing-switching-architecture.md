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

## 3. Inter-VLAN routing & L3 switching

### Technical Definition
Inter-VLAN routing is the process of forwarding traffic between distinct VLANs using a Layer 3 device, such as a router or a multilayer switch. L3 switching (or multilayer switching) performs this function at hardware speeds by utilizing Application-Specific Integrated Circuits (ASICs) to perform routing lookups, effectively bypassing the performance bottlenecks associated with traditional "router-on-a-stick" configurations. This architecture allows for high-density, wire-speed routing between subnets within the same physical switching fabric.

### Underlying Mechanism
The mechanism relies on Switch Virtual Interfaces (SVIs) or routed ports to act as the default gateway for each VLAN. When a packet arrives at the switch, the ASIC performs a lookup in the Forwarding Information Base (FIB), which is populated by the control plane's routing table. The FIB, stored in high-speed TCAM, contains the destination prefix, the next-hop MAC address, and the egress interface. The switch then performs a hardware-level rewrite of the Ethernet frame header—updating the source and destination MAC addresses and decrementing the TTL—before forwarding the frame to the destination VLAN. This process, often referred to as CEF (Cisco Express Forwarding) or equivalent ASIC-based forwarding, ensures that the routing decision is made in a single clock cycle, regardless of the number of routes in the table.

[DIAGRAM: Flowchart illustrating the packet rewrite process during L3 switching, showing the transition from ingress VLAN to egress VLAN]

### Why It Exists
Inter-VLAN routing exists to provide connectivity between segmented broadcast domains while maintaining the security and performance benefits of VLAN isolation. Traditional router-on-a-stick designs, where all inter-VLAN traffic must traverse a single physical link to an external router, create significant bandwidth bottlenecks and latency. L3 switching solves this by distributing the routing function across the switching fabric, allowing traffic to be routed at the core or distribution layer at wire speed, which is essential for modern, high-throughput datacenter environments.

### Enterprise / Banking Reality
In Tier-1 banking, L3 switching is the backbone of the datacenter and campus network. We utilize high-performance multilayer switches to handle massive volumes of inter-VLAN traffic, such as communication between application tiers and database clusters. We avoid router-on-a-stick designs entirely, preferring to terminate SVIs on core or distribution switches. Security is enforced through SVI-based Access Control Lists (ACLs) or, more commonly, by offloading traffic to a dedicated firewall cluster for deep packet inspection (DPI) when crossing security zones (e.g., from the web tier to the database tier). This ensures that we maintain high performance while adhering to strict regulatory requirements for traffic inspection and segmentation.

### Operational Considerations
Operationalizing L3 switching requires careful management of SVI configurations and MTU consistency across the fabric. Administrators must ensure that ACLs applied to SVIs are optimized for hardware processing to avoid "punting" traffic to the CPU, which can cause significant performance degradation. Monitoring tools should track ASIC utilization and TCAM usage to ensure that the switch has sufficient resources to handle the routing table and ACL requirements.
[CLI: Command to verify the SVI status and IP configuration]
[CLI: Command to inspect the hardware forwarding table (FIB) for a specific destination prefix]
[CLI: Command to verify ACL hit counts on an SVI to identify traffic patterns]

### Common Misconceptions
!!! warning
    A common misconception is that an L3 switch is "just a router." While they perform the same function, L3 switches are optimized for high-density, wire-speed switching and may lack the advanced features (e.g., complex NAT, deep packet inspection, or massive routing table support) found in dedicated edge routers. Another error is assuming that all ports on an L3 switch can perform L3 routing; in many platforms, only specific ports or SVIs are capable of L3 forwarding, and misconfiguration can lead to unexpected traffic flows.

### Interview Angle
1. Question: How do you decide between using an SVI and a routed port for inter-VLAN routing in a core switch?
   Answer: We use SVIs when we need to provide a gateway for multiple VLANs on a single switch or stack, which is typical for access and distribution layers. We use routed ports for point-to-point connections between core switches or to external routers, as they provide a cleaner, more deterministic L3 boundary and avoid the overhead of L2 encapsulation and STP processing.
2. Question: What are the performance implications of applying ACLs to an SVI in a high-throughput environment?
   Answer: If the ACL is not optimized for hardware processing, the switch may be forced to "punt" packets to the CPU for inspection, which can lead to high CPU utilization and packet drops. We ensure that all ACLs are compiled into TCAM and that we avoid features that require software-based processing, such as logging or complex regex matching, on high-traffic interfaces.
3. Question: How do you handle MTU mismatches in an L3 switching environment, and why does it matter?
   Answer: MTU mismatches can lead to fragmentation or dropped packets, which are notoriously difficult to troubleshoot. We enforce a consistent MTU (e.g., 9000 bytes for jumbo frames) across the entire L3 fabric. We also implement Path MTU Discovery (PMTUD) where possible, but we rely on consistent configuration as the primary defense against fragmentation-related performance issues.

### Related Concepts
- Section 4.1: L2 fundamentals
- Section 4.2: L3 routing protocols (OSPF, EIGRP, BGP)
- Section 4.4: First-hop redundancy (HSRP/VRRP/GLBP)

## 4. First-hop redundancy (HSRP/VRRP/GLBP)

### Technical Definition
First-hop redundancy protocols (FHRPs) are a class of network protocols designed to provide high availability for default gateways. They allow multiple physical routers or multilayer switches to present a single, virtual IP address and virtual MAC address to end hosts. If the primary device fails, the virtual gateway seamlessly transitions to a standby device, ensuring continuous connectivity for clients without requiring manual reconfiguration of default gateway settings on end-user devices.

### Underlying Mechanism
FHRPs operate by electing an "Active" or "Master" router to handle traffic for the virtual IP, while "Standby" or "Backup" routers monitor the health of the active device. This is achieved through periodic multicast "hello" packets. If the standby device stops receiving these hellos, it assumes the active device has failed and promotes itself to the active role. The transition involves the new active device sending a gratuitous ARP (Address Resolution Protocol) to update the CAM tables of connected switches, ensuring that traffic destined for the virtual MAC address is now forwarded to the new active device's physical port. HSRP (Hot Standby Router Protocol) is Cisco-proprietary, VRRP (Virtual Router Redundancy Protocol) is the open-standard equivalent, and GLBP (Gateway Load Balancing Protocol) extends this concept by providing load balancing across multiple routers by responding to ARP requests with different virtual MAC addresses.

[DIAGRAM: Flowchart illustrating the failover process between an Active and Standby router, including the gratuitous ARP update]

### Why It Exists
FHRPs exist to eliminate the single point of failure inherent in a static default gateway configuration. In a standard network, if the router serving as the default gateway fails, all hosts on that subnet lose connectivity to external networks. FHRPs provide the necessary redundancy to ensure that the default gateway remains available even in the event of a hardware or link failure, which is critical for maintaining uptime in enterprise environments.

### Enterprise / Banking Reality
In Tier-1 banking, FHRPs are a fundamental component of our high-availability architecture. We typically deploy HSRP or VRRP in our distribution and core layers to provide redundant gateways for our server and user VLANs. We heavily utilize "tracking objects" to monitor upstream connectivity; if the uplink to the core network fails, the FHRP priority is automatically decremented, forcing a failover to the standby device that still has a healthy uplink. This prevents "black-holing" traffic, where a router remains active but cannot reach the destination. We avoid GLBP in most core environments due to its complexity and the potential for asymmetric routing, preferring deterministic active/standby models.

### Operational Considerations
Operationalizing FHRPs requires careful tuning of hello and hold timers to balance fast failover with control plane stability. We also ensure that FHRP groups are aligned with the STP root bridge to prevent suboptimal traffic paths. Monitoring is critical; we alert on any state changes (e.g., active-to-standby transitions) as these often indicate underlying physical layer issues.
[CLI: Command to verify the HSRP/VRRP state and priority of a switch interface]
[CLI: Command to configure interface tracking to trigger a failover based on uplink status]
[CLI: Command to display the virtual MAC address and current active router for an FHRP group]

### Common Misconceptions
!!! warning
    A common misconception is that FHRPs provide load balancing. HSRP and VRRP are strictly active/standby protocols; they do not distribute traffic across multiple routers. Another error is assuming that FHRPs can solve all connectivity issues; they only protect the first hop. If the failure is further upstream in the network, the FHRP will remain active, and traffic will still be dropped.

### Interview Angle
1. Question: How do you ensure that the FHRP active router is also the STP root bridge for a given VLAN?
   Answer: We manually configure the priority values for both the FHRP and the STP instance to ensure that the same physical device is the primary for both. This ensures that traffic flows are symmetric and predictable, preventing unnecessary hair-pinning of traffic between switches.
2. Question: Explain the difference between HSRP, VRRP, and GLBP, and when you would choose one over the other.
   Answer: HSRP is Cisco-proprietary and widely used in Cisco-centric environments. VRRP is the open-standard equivalent, which is necessary for multi-vendor interoperability. GLBP provides load balancing by distributing traffic across multiple routers, but it introduces significant complexity and potential for asymmetric routing, so we generally avoid it in core banking environments where deterministic traffic paths are required.
3. Question: What is the impact of setting FHRP timers too aggressively?
   Answer: Setting timers too aggressively (e.g., sub-second hellos) can lead to "flapping" where the routers constantly switch roles due to transient network congestion or CPU spikes. This can cause significant instability in the network and should be avoided unless absolutely necessary. We prefer to use BFD (Bidirectional Forwarding Detection) for fast failure detection, as it is more efficient and stable than aggressive FHRP timers.

### Related Concepts
- Section 4.1: L2 fundamentals
- Section 4.3: Inter-VLAN routing & L3 switching

## 5. Route summarization & addressing hierarchy

### Technical Definition
Route summarization (or route aggregation) is the process of combining multiple specific network prefixes into a single, less-specific summary route. Addressing hierarchy refers to the structured allocation of IP address space (CIDR blocks) that mirrors the physical or logical topology of the network. This approach ensures that routing tables remain compact and stable by hiding the complexity of the underlying network structure from the core backbone.

### Underlying Mechanism
The mechanism relies on the Longest Prefix Match (LPM) algorithm used by routers to determine the best path. When a router receives a packet, it compares the destination IP against all entries in the Forwarding Information Base (FIB). If multiple matches exist, the router selects the most specific (longest) prefix. Summarization works by advertising only the aggregate prefix (e.g., 10.1.0.0/16) to the core, while the local distribution layer maintains the specific subnets (e.g., 10.1.1.0/24, 10.1.2.0/24). This reduces the size of the RIB and FIB, minimizing memory usage and CPU overhead during route recalculations. Hierarchical addressing is enforced through strict IPAM (IP Address Management) policies, where large blocks are allocated to regions or data centers, and then subdivided into smaller blocks for specific functions or VLANs, ensuring that summarization boundaries align with the physical topology.

[DIAGRAM: Flowchart illustrating the aggregation of multiple specific subnets into a single summary route at the distribution layer]

### Why It Exists
Route summarization and hierarchical addressing exist to solve the scalability limitations of routing protocols. Without summarization, every minor link flap or subnet change in an access layer would trigger a global update across the entire enterprise network, leading to "routing table churn" and potential instability. By summarizing at the distribution or core boundary, we isolate the impact of local topology changes, ensuring that the core backbone remains stable and efficient. Hierarchical addressing also simplifies troubleshooting and policy enforcement, as traffic patterns become predictable and aligned with the IP structure.

### Enterprise / Banking Reality
In Tier-1 banking, route summarization is a critical design requirement for global WAN and datacenter interconnects. We enforce a strict hierarchical IP addressing scheme, often utilizing a "top-down" allocation model where large blocks are assigned to specific business units or geographic regions. This allows us to summarize routes at the regional boundary, significantly reducing the size of the global routing table. We treat summarization as a mandatory control for stability; any network segment that cannot be summarized is flagged as a design exception and requires a formal risk assessment. This discipline is essential for maintaining the performance of our high-frequency trading and core banking platforms, where even minor routing instability can have significant financial impact.

### Operational Considerations
Operationalizing summarization requires robust IPAM tools to manage address allocation and prevent overlapping subnets. Administrators must ensure that summarization boundaries are correctly configured and that "discard routes" (null0 routes) are implemented to prevent routing loops when a summary route is advertised but the specific subnets are down. Monitoring tools should track the size of the routing table and alert on any unexpected growth or prefix instability.
[CLI: Command to configure a summary address on an OSPF or EIGRP process]
[CLI: Command to verify the routing table and confirm that specific subnets are hidden behind the summary route]
[CLI: Command to configure a static discard route to prevent loops for a summary prefix]

### Common Misconceptions
!!! warning
    A common misconception is that summarization is purely for reducing routing table size. While true, its primary benefit in large networks is stability; it prevents local instability from propagating globally. Another error is assuming that summarization is always possible; if the network is not designed hierarchically, summarization will lead to black-holing traffic, as the router will not know how to reach the specific subnets that were hidden by the summary.

### Interview Angle
1. Question: How do you design a hierarchical IP addressing scheme that supports both current requirements and future growth?
   Answer: We use a "variable-length subnet masking" (VLSM) approach, allocating large blocks (e.g., /16 or /18) to major sites or business units, and then subdividing these into smaller blocks (e.g., /24) for specific VLANs or services. We always leave "white space" (unallocated address space) within each block to accommodate future growth without needing to re-address the network.
2. Question: What are the risks of aggressive route summarization, and how do you mitigate them?
   Answer: The primary risk is "black-holing" traffic if the summary route is advertised but the specific subnets are unreachable. We mitigate this by implementing "discard routes" (pointing the summary to null0) on the summarizing router. This ensures that if a specific subnet is down, the router drops the traffic rather than forwarding it to a default gateway, which could create a routing loop.
3. Question: How does route summarization impact traffic engineering and path selection in a multi-homed environment?
   Answer: Summarization can hide the specific path information needed for granular traffic engineering. If we need to influence traffic to a specific subnet, we may need to advertise more specific routes alongside the summary. We use route maps and prefix lists to carefully control which specific routes are advertised, balancing the need for summarization with the requirement for precise path control.

### Related Concepts
- Section 4.2: L3 routing protocols (OSPF, EIGRP, BGP)
- Section 4.3: Inter-VLAN routing & L3 switching
