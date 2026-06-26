# 3.1 Privileged Access Management & Tiered Administration

## 1. Tiered Admin Model (Tier 0/1/2) & ESAE/Red Forest Concepts

### Technical Definition
The tiered administrative model is a security architecture designed to isolate privileged identities and control surfaces into distinct, non-overlapping trust boundaries. By categorizing assets and accounts into Tiers (Tier 0 for the identity control plane, Tier 1 for enterprise servers/applications, and Tier 2 for workstations), organizations can prevent lateral movement and privilege escalation. The Enhanced Security Administrative Environment (ESAE), often referred to as the "Red Forest," is a specific implementation of this model that utilizes a separate, hardened Active Directory forest to manage the Tier 0 control plane, ensuring that administrative credentials for the most sensitive assets are never exposed to lower-trust environments.

### Underlying Mechanism
The mechanism relies on the strict enforcement of trust boundaries and the segregation of administrative credentials. Tier 0 contains the most sensitive assets—Domain Controllers, identity providers, and the administrative accounts that manage them. Tier 1 contains enterprise servers and applications, while Tier 2 contains user workstations. The core principle is that an account in a lower tier (e.g., a Tier 2 workstation admin) must never have administrative rights in a higher tier (e.g., Tier 0). This is enforced through strict ACLs and security descriptors on directory objects, as outlined in Section 1.8, which prevent lower-tier accounts from modifying or interacting with higher-tier objects. In an ESAE/Red Forest model, the Tier 0 control plane is physically and logically separated into a dedicated forest, with one-way trusts ensuring that the production forest can be managed by the Red Forest, but the Red Forest remains isolated from any compromise in the production environment.

[DIAGRAM: Architecture diagram showing the three-tier model with isolated administrative boundaries and the Red Forest trust topology]

### Why It Exists
This model exists to break the "flat" administrative structure that is common in legacy environments, where a single compromised workstation admin account could potentially lead to a full domain compromise. By creating these tiers, organizations can contain the blast radius of a credential theft event. If a Tier 2 workstation is compromised, the attacker is trapped within the Tier 2 boundary and cannot move laterally into Tier 1 or Tier 0. The ESAE/Red Forest concept takes this a step further by ensuring that even if the entire production forest is compromised, the identity control plane (Tier 0) remains secure and recoverable.

### Enterprise / Banking Reality
In Tier-1 banking, the tiered administrative model is a non-negotiable requirement for protecting the core banking systems and the identity infrastructure. Banks must demonstrate strict separation of duties and access control to comply with regulations like SWIFT CSP and SOX. Architects must design these tiers to be rigid, with automated monitoring to detect any "tier-crossing" activity, such as a Tier 1 admin logging into a Tier 2 workstation. The ESAE/Red Forest model is often the gold standard for these environments, providing the highest level of isolation for the identity control plane, though it requires significant operational overhead to maintain.

### Operational Considerations
Operationalizing the tiered model requires a disciplined, policy-driven approach. Administrators must maintain a strict inventory of all administrative accounts and their assigned tiers, ensuring that no account has permissions across multiple tiers. Monitoring is critical; administrators must track all administrative logins, identify any tier-crossing activity, and provide reporting on the compliance of the tiered model. Furthermore, administrators must ensure that they have a clear process for handling administrative tasks, providing support staff with the necessary tools and guidance to perform their duties within their assigned tier without compromising the security boundary.

[CLI: PowerShell command to audit the membership of administrative groups and identify accounts with cross-tier permissions]

### Common Misconceptions
!!! warning
    A common misconception is that the tiered model is a "set and forget" configuration. In reality, it requires ongoing maintenance, as the administrative landscape is dynamic and new assets are constantly being added. Another error is assuming that the tiered model is a replacement for other security controls; it is a foundational architecture that must be combined with other measures, such as PAWs, JIT access, and session monitoring, to be effective.

### Interview Angle
1. Question: How do you handle the operational friction caused by a strict tiered administrative model?
   Answer: We minimize friction by providing clear, well-documented procedures for administrative tasks and by using automation to streamline the process of requesting and granting temporary access. We also use JIT access to ensure that administrators only have the necessary privileges for the duration of their task, reducing the need for permanent, high-privilege accounts.
2. Question: What are the key considerations when designing a tiered administrative model for a Tier-1 bank?
   Answer: The key considerations are isolation, auditability, and scalability. The model must be designed to provide the highest level of isolation for the identity control plane, with clear, auditable boundaries between tiers. It must also be scalable to support the bank's entire infrastructure, and it must be integrated with the bank's broader security and identity management systems.
3. Question: How do you handle the challenge of detecting and responding to "tier-crossing" activity?
   Answer: We use centralized monitoring and logging to track all administrative logins, identifying any activity that violates the tiered model. We also implement automated alerts that trigger an immediate investigation if a tier-crossing event is detected, and we have a clear, pre-defined incident response plan for these events.

### Related Concepts
- Section 1.8: Directory Authorization & Scoping
- Section 2.5: Endpoint Security & LAPS Client Mechanics
- Section 3.1: Privileged Access Workstations (PAWs)
