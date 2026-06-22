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
