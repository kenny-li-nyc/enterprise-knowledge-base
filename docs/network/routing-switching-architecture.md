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
