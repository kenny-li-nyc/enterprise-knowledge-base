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

## 2. Privileged Access Workstations (PAWs)

### Technical Definition
Privileged Access Workstations (PAWs) are dedicated, hardened computing devices used exclusively for administrative tasks. Unlike general-purpose workstations, PAWs are stripped of non-essential software, restricted from internet and email access, and configured with strict security policies to prevent the execution of unauthorized code. They serve as the "clean source" for administrative actions, ensuring that privileged credentials—such as domain admin passwords or cloud global admin tokens—are never exposed to the risks inherent in a standard user environment.

### Underlying Mechanism
The mechanism relies on hardware-backed security and strict software isolation. PAWs utilize TPM 2.0 for secure boot and device identity, ensuring that the OS has not been tampered with. Network isolation is enforced via host-based firewalls and network-level restrictions, preventing the PAW from communicating with the public internet or non-essential internal segments. Application control is enforced via WDAC or AppLocker, ensuring that only a minimal, pre-approved set of administrative tools can execute. As noted in Section 1.8, the PAW is configured to only interact with the specific administrative interfaces required for its tier, ensuring that structural authorization is maintained. Furthermore, as discussed in Section 2.5, the PAW relies on endpoint-side enforcement mechanisms to ensure that the device remains in a compliant state, preventing the introduction of malicious software or unauthorized configuration changes.

[DIAGRAM: Architecture diagram showing the PAW isolation model, with restricted network access and hardened software configuration]

### Why It Exists
PAWs exist to mitigate the risk of credential theft, which is the primary vector for most advanced persistent threats (APTs). In a standard workstation environment, an administrator's credentials are vulnerable to keyloggers, memory scraping, and other malware that can be easily introduced via email or web browsing. By using a PAW, the administrator performs their duties on a device that is inherently resistant to these threats, ensuring that their credentials remain secure even if the rest of the enterprise network is compromised.

### Enterprise / Banking Reality
In Tier-1 banking, PAWs are a critical security control for protecting the identity control plane and other high-value assets. Banks must implement PAWs for all Tier 0 and Tier 1 administrators, ensuring that these accounts are never used on general-purpose workstations. Architects must design these solutions to be highly secure, with strict physical or virtual isolation, and they must be integrated with the bank's broader security infrastructure, including EDR and SIEM, to ensure that any suspicious activity on a PAW is immediately detected and investigated.

### Operational Considerations
Operationalizing PAWs requires a disciplined, policy-driven approach. Administrators must maintain a strict inventory of all PAWs, ensuring that they are kept up-to-date and compliant with security baselines. Monitoring is critical; administrators must track all activity on PAWs, identify any unauthorized or suspicious behavior, and provide reporting on the compliance of the PAW fleet. Furthermore, administrators must ensure that they have a clear process for handling PAW-related support requests, providing support staff with the necessary tools and guidance to perform their duties without compromising the security boundary.

[CLI: PowerShell command to verify the security configuration and compliance status of a PAW]

### Common Misconceptions
!!! warning
    A common misconception is that a PAW is just "a laptop for admins." In reality, it is a highly specialized security boundary that must be strictly managed and isolated to be effective. Another error is assuming that PAWs are a "set and forget" solution; they require ongoing maintenance, as the security policies must be constantly updated to reflect changes in the threat landscape and the business.

### Interview Angle
1. Question: How do you handle the operational friction caused by the strict restrictions on PAWs?
   Answer: We minimize friction by providing clear, well-documented procedures for administrative tasks and by using automation to streamline the process of performing common administrative actions. We also provide support staff with the necessary training and guidance to work effectively within the PAW environment, ensuring that they understand the importance of the security controls.
2. Question: What are the key considerations when designing a PAW strategy for a Tier-1 bank?
   Answer: The key considerations are isolation, security, and manageability. The PAW must be designed to provide the highest level of isolation, with strict network and application controls. It must also be secure, with hardware-backed security and continuous monitoring, and it must be manageable, with automated deployment and configuration management.
3. Question: How do you handle the challenge of detecting and responding to suspicious activity on a PAW?
   Answer: We use centralized monitoring and logging to track all activity on PAWs, identifying any unauthorized or suspicious behavior. We also implement automated alerts that trigger an immediate investigation if a security event is detected on a PAW, and we have a clear, pre-defined incident response plan for these events.

### Related Concepts
- Section 1.8: Directory Authorization & Scoping
- Section 2.5: Endpoint Security & LAPS Client Mechanics
- Section 3.1: Tiered Admin Model (Tier 0/1/2) & ESAE/Red Forest Concepts
