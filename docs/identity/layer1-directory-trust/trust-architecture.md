# Trust Architecture

## Trust Fundamentals

### Technical Definition
A Trust is a logical relationship established between two Active Directory domains or forests that allows security principals (users, computers, groups) in one domain to be authenticated by the other. At its core, a trust is a mechanism that extends the authentication boundary of a domain or forest, enabling users to access resources (files, applications, services) located in a different security context. It is the fundamental building block of cross-domain and cross-forest resource sharing in Windows environments.

### Underlying Mechanism
Trusts are implemented through the creation of `trustedDomain` objects within the Active Directory database. When a trust is established, each side of the relationship stores information about the other, including the trust type, direction, and transitivity. 

The authentication process relies on the Kerberos protocol. When a user in a trusted domain attempts to access a resource in a trusting domain, the user's domain controller issues a referral ticket. The client then presents this referral ticket to the trusting domain's domain controller, which validates the ticket and grants access to the requested resource. This process involves the exchange of inter-realm TGTs (Ticket Granting Tickets) and relies on the secure channel established between the domain controllers of the two domains. The `LSASS` (Local Security Authority Subsystem Service) process on the domain controllers handles the cryptographic verification of these tickets, ensuring that the trust relationship is secure and that authentication requests are legitimate.

[DIAGRAM: A visual representation of the Kerberos referral process between two trusted domains]

### Why It Exists
Trusts exist to facilitate resource sharing and collaboration in distributed environments. Without trusts, every domain would be an isolated island, and users would need separate accounts for every domain they needed to access. Trusts allow organizations to maintain a single identity for each user while providing access to resources across the entire enterprise. They are essential for mergers and acquisitions, where two previously independent organizations need to integrate their IT environments, and for large enterprises that need to delegate administrative control while maintaining a unified identity infrastructure.

### Enterprise / Banking Reality
In a Tier-1 banking environment, trust architecture is a critical component of the security posture. The design philosophy is to minimize the number of trusts to the absolute minimum required for business operations. Every trust is a potential vector for lateral movement and must be carefully audited and monitored. 

From a compliance perspective (SOX, PCI-DSS, FFIEC), trusts are a major audit point. Auditors will look for evidence that trusts are necessary, that they are properly configured (e.g., using SID filtering), and that they are regularly reviewed. In a modern banking architecture, you will rarely see complex, multi-hop trust chains. Instead, you will see a "hub-and-spoke" model where a central identity forest trusts (or is trusted by) specific resource forests, with strict controls on what can be accessed across those boundaries. Entity segregation is paramount; trusts are often restricted to specific services or applications rather than being broad, forest-wide relationships.

| Trust Attribute | Description |
| :--- | :--- |
| **Direction** | One-way (inbound/outbound) or Two-way |
| **Transitivity** | Transitive (extends to child domains) or Non-transitive |
| **Scope** | Domain-wide or Forest-wide |
| **Security** | SID Filtering, Selective Authentication |

### Operational Considerations
Day-to-day operations involve monitoring the health of trust relationships. If a trust breaks, authentication across the boundary will fail, leading to service outages. You must monitor for Event IDs related to trust failures (e.g., 5806, 5807, 5810). 

Regular audits of trust relationships are essential. You should use PowerShell to list all trusts and verify their status. If a trust is no longer needed, it should be removed immediately to reduce the attack surface.

[CLI: Command to list all trust relationships in the current domain]

### Common Misconceptions
!!! warning "Common Misconceptions"
    *   **"Trusts are inherently secure."** Trusts are a mechanism for authentication, not a security feature. They must be configured with appropriate security controls (like SID filtering) to be secure.
    *   **"Two-way trusts are always better."** Two-way trusts are more convenient, but they also increase the attack surface. If you only need one-way access, use a one-way trust.
    *   **"Trusts are transitive by default."** Only trusts within a forest are transitive by default. Cross-forest trusts are non-transitive unless explicitly configured otherwise (e.g., using forest trusts).

### Interview Angle
1.  **"How do you secure a cross-forest trust in a high-security environment?"**
    *   *Model Answer:* "I would implement SID filtering to prevent SID history injection, use selective authentication to restrict which users can authenticate to specific resources, and ensure that the trust is only as broad as necessary. I would also implement rigorous monitoring and alerting for any changes to the trust configuration."
2.  **"What is the difference between transitive and non-transitive trusts?"**
    *   *Model Answer:* "Transitive trusts extend the trust relationship to all child domains in the forest, while non-transitive trusts are limited to the two domains involved in the trust relationship. Transitive trusts are the default within a forest, while cross-forest trusts are non-transitive by default."
3.  **"Why would you choose selective authentication over domain-wide authentication?"**
    *   *Model Answer:* "Selective authentication provides a higher level of security by allowing you to specify exactly which users or groups can authenticate to specific resources in the trusting domain. It is a critical control for preventing lateral movement in high-security environments."

### Related Concepts
*   [Forests, Domains, OUs & Sites](forests-domains-ous-sites.md)
*   [Trust Types](trust-architecture.md#trust-types)
*   [Trust Security Controls](trust-architecture.md#trust-security-controls)

## Trust Types

### Technical Definition
Trust types define the scope and behavior of the relationship between domains or forests. Active Directory supports several distinct trust types, each designed for specific architectural scenarios: Parent-Child, Tree-Root, External, Forest, and Realm trusts. These types dictate whether the trust is transitive, the direction of authentication, and the scope of the trust (domain-level vs. forest-level).

### Underlying Mechanism
The trust type is stored as a property of the `trustedDomain` object in Active Directory. When a trust is created, the `trustType` attribute (e.g., `DOWNLEVEL`, `UPLEVEL`, `MIT`, `DCE`) and `trustAttributes` (e.g., `FOREST_TRANSITIVE`, `NON_TRANSITIVE`) are set to define how the domain controllers interact. For example, a Forest Trust uses the `FOREST_TRANSITIVE` attribute, which allows the trust to extend to all domains within the forest, whereas an External Trust is non-transitive and limited to the two specific domains involved. The Kerberos Key Distribution Center (KDC) uses these attributes to determine whether to issue a referral ticket or to deny the authentication request.

[DIAGRAM: A comparison matrix or flowchart showing the different trust types and their characteristics]

### Why It Exists
Different trust types exist to accommodate the evolution of Active Directory and the diverse needs of enterprise environments. Parent-Child and Tree-Root trusts are the "native" trusts that form the structure of a forest. External trusts were the original mechanism for connecting disparate domains before the concept of a "Forest" was fully realized. Forest trusts were introduced to allow for the secure, transitive connection of entire forests, which is essential for large-scale mergers and acquisitions. Realm trusts exist to allow Active Directory to interoperate with non-Windows Kerberos environments (e.g., Unix/Linux).

### Enterprise / Banking Reality
In a Tier-1 bank, the use of trust types is strictly governed. You will almost exclusively see Parent-Child trusts (if multiple domains exist) and Forest trusts (for connecting to partner or subsidiary forests). External trusts are generally discouraged because they are non-transitive and harder to manage at scale. When connecting to a non-Windows environment, Realm trusts are used, but they are treated with extreme caution due to the potential for security mismatches between the two environments. Audit requirements often mandate that any cross-forest trust must be documented, justified, and reviewed annually.

| Trust Type | Transitive | Scope | Typical Use Case |
| :--- | :--- | :--- | :--- |
| **Parent-Child** | Yes | Forest | Building the forest hierarchy |
| **Tree-Root** | Yes | Forest | Adding a new tree to a forest |
| **External** | No | Domain | Connecting to a legacy domain |
| **Forest** | Yes | Forest | Connecting two separate forests |
| **Realm** | No | Domain | Connecting to non-Windows Kerberos |

### Operational Considerations
Managing trust types requires understanding the implications of each type. For instance, creating an External trust when a Forest trust is needed can lead to significant administrative overhead, as you would need to create multiple external trusts to achieve the same connectivity. Monitoring involves checking the `trustAttributes` and `trustType` properties to ensure they align with the intended design. If a trust is not behaving as expected (e.g., authentication is failing across a forest boundary), the first step is to verify that the trust type is correctly configured.

[CLI: Command to inspect the properties of a specific trust]

### Common Misconceptions
!!! warning "Common Misconceptions"
    *   **"All trusts are the same."** This is false. Trust types have fundamentally different behaviors regarding transitivity and scope.
    *   **"I can use an External trust to connect two forests."** While technically possible, it is a poor design choice. Forest trusts are designed for this purpose and provide better manageability and security.
    *   **"Realm trusts are just like External trusts."** Realm trusts are specifically for non-Windows Kerberos environments and have different configuration requirements and limitations.

### Interview Angle
1.  **"When would you choose a Forest trust over an External trust?"**
    *   *Model Answer:* "I would choose a Forest trust when I need to connect two entire forests and require transitive authentication across all domains in both forests. External trusts are non-transitive and limited to the two specific domains, making them unsuitable for large-scale forest-to-forest connectivity."
2.  **"What are the risks of using Realm trusts?"**
    *   *Model Answer:* "Realm trusts introduce the risk of security mismatches between Active Directory and the non-Windows Kerberos environment. You must ensure that the encryption types, ticket lifetimes, and other Kerberos settings are compatible, or authentication will fail or be insecure."
3.  **"How do you determine the correct trust type for a new business requirement?"**
    *   *Model Answer:* "I would first assess the scope of the requirement: is it domain-to-domain or forest-to-forest? Do I need transitivity? Is it a Windows-to-Windows connection or a cross-platform connection? Based on these answers, I would select the trust type that provides the necessary connectivity with the least amount of administrative overhead and the highest level of security."

### Related Concepts
*   [Trust Fundamentals](trust-architecture.md#trust-fundamentals)
*   [Trust Security Controls](trust-architecture.md#trust-security-controls)
*   [Kerberos](kerberos.md)

## Trust Security Controls

### Technical Definition
Trust Security Controls are the mechanisms used to restrict and validate authentication traffic across trust boundaries. These controls include SID Filtering, Selective Authentication, and Quarantine. They are designed to prevent unauthorized access, lateral movement, and privilege escalation by ensuring that only legitimate authentication requests are processed and that security principals cannot impersonate identities from other domains or forests.

### Underlying Mechanism
These controls operate by inspecting the authentication tokens (specifically the Privilege Attribute Certificate, or PAC) and the Kerberos referral process. 
- **SID Filtering:** When a trust is established, Active Directory can be configured to filter out any SIDs in the user's token that do not belong to the trusted domain. This prevents "SID History" attacks, where an attacker injects a high-privilege SID into their own account's SID history to gain elevated access in the trusting domain.
- **Selective Authentication:** This control restricts which users can authenticate to specific resources in the trusting domain. When enabled, the trusting domain controller checks the "Allowed to Authenticate" permission on the target computer object before granting access.
- **Quarantine:** This is a state applied to a trust that restricts the scope of authentication until the trust is fully validated and configured.

[DIAGRAM: A visual representation of how SID filtering and Selective Authentication intercept and validate authentication requests]

### Why It Exists
These controls exist because trusts are inherently risky. By default, a trust allows any user in the trusted domain to authenticate to any resource in the trusting domain. In a large enterprise, this is unacceptable. These controls provide the necessary granularity to enforce the principle of least privilege and to contain the impact of a potential compromise in one domain or forest.

### Enterprise / Banking Reality
In a Tier-1 bank, these controls are mandatory for all cross-forest trusts. SID filtering is enabled by default for forest trusts and must never be disabled. Selective authentication is the standard for high-value assets (e.g., Tier 0 systems, core banking applications). The design pattern is to use "Allowed to Authenticate" permissions on computer objects to strictly control access. Audit teams will verify that these controls are in place and that they are correctly configured. Any deviation from this standard is considered a significant security risk and will be flagged during audits.

### Operational Considerations
Implementing these controls requires careful planning. Selective authentication, in particular, can be complex to manage, as it requires explicitly granting "Allowed to Authenticate" permissions on every computer object that needs to be accessed by users from the trusted domain. If this is not done correctly, authentication will fail, and users will be unable to access resources. Monitoring involves checking for Event IDs related to authentication failures (e.g., 4624, 4625) and auditing the "Allowed to Authenticate" permissions.

[CLI: Command to configure Selective Authentication on a computer object]

### Common Misconceptions
!!! warning "Common Misconceptions"
    *   **"SID filtering breaks everything."** SID filtering only breaks authentication if the user's token contains invalid or malicious SIDs. If the trust is configured correctly, it should have no impact on legitimate authentication.
    *   **"Selective authentication is only for users."** Selective authentication applies to both users and groups. You must grant the "Allowed to Authenticate" permission to the specific security principal that needs access.
    *   **"I can just enable these controls after the trust is established."** While you can enable them later, it is best practice to plan and configure them during the trust creation process to avoid service disruptions.

### Interview Angle
1.  **"What is the purpose of SID filtering in a cross-forest trust?"**
    *   *Model Answer:* "SID filtering prevents SID history injection attacks by ensuring that only SIDs belonging to the trusted domain are allowed in the user's token. It is a critical security control for preventing privilege escalation across trust boundaries."
2.  **"How does Selective Authentication differ from standard authentication?"**
    *   *Model Answer:* "Standard authentication allows any user in the trusted domain to authenticate to any resource in the trusting domain. Selective authentication restricts this by requiring an explicit 'Allowed to Authenticate' permission on the target computer object, providing a much higher level of control."
3.  **"What are the operational challenges of implementing Selective Authentication?"**
    *   *Model Answer:* "The main challenge is the administrative overhead of managing the 'Allowed to Authenticate' permissions on every computer object that needs to be accessed. It requires a well-defined process for granting and revoking these permissions to avoid service disruptions and security gaps."

### Related Concepts
*   [Trust Fundamentals](trust-architecture.md#trust-fundamentals)
*   [Trust Types](trust-architecture.md#trust-types)
*   [Kerberos](kerberos.md)
