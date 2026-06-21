# Forests, Domains, OUs & Sites

## Forest

### Technical Definition
An Active Directory (AD) Forest is the highest-level container in the Active Directory logical structure. It represents a single security boundary, a single schema, and a single configuration partition. All domains within a forest share a common schema (the definition of all object classes and attributes) and a common configuration (the topology of the forest, including sites, subnets, and services). The forest is the ultimate boundary for administrative authority; a user with Domain Admin rights in one domain cannot inherently manage objects in another domain within the same forest, but a user with Enterprise Admin rights has authority across the entire forest.

### Underlying Mechanism
The forest is implemented through three primary partitions replicated across all Domain Controllers (DCs) in the forest:

1.  **Schema Partition:** Defines the rules for the directory (what objects exist, what attributes they have). This is replicated to every DC in the forest.
2.  **Configuration Partition:** Contains the physical and logical topology of the forest (sites, subnets, services, and domain information). This is also replicated to every DC in the forest.
3.  **Domain Partition:** Contains the actual objects (users, computers, groups) for a specific domain. This is only replicated to DCs within that specific domain.

The forest also relies on the **Global Catalog (GC)**. The GC is a special role held by one or more DCs that maintains a partial, read-only copy of every object in every domain within the forest. This allows for forest-wide searches without requiring cross-domain queries for every request. Trust relationships between domains in a forest are automatically created as transitive, two-way Kerberos trusts, allowing for seamless authentication across the entire forest boundary.

### Why It Exists
The forest exists to provide a unified namespace and a shared security context while allowing for the delegation of administrative control. Historically, it was designed to allow organizations to merge disparate IT environments into a single, manageable directory service. It provides a mechanism to enforce a consistent security policy (via Group Policy Objects) and a unified identity store, ensuring that a user's identity is consistent regardless of which domain they authenticate against.

### Enterprise / Banking Reality
In a Tier-1 banking environment, the Forest is treated as the "Tier 0" security boundary. The design philosophy is almost exclusively "Single Forest, Single Domain" (or a very limited number of forests for specific, isolated use cases like M&A or extreme regulatory isolation). 

From an audit and compliance perspective (e.g., SOX, PCI-DSS), the Forest is the scope of the "Identity Perimeter." If an attacker compromises the Forest (e.g., gains Enterprise Admin or Domain Admin in a forest-root domain), they have effectively compromised the entire identity infrastructure. Therefore, banking architectures implement strict "Tiered Administration" models where Forest-level administrative accounts are restricted to highly secure, hardened workstations (PAWs - Privileged Access Workstations) and are never used to log into lower-tier systems.

### Operational Considerations
Day-to-day operations at the forest level are rare but high-impact. 
*   **Schema Updates:** Modifying the schema is a forest-wide operation. It requires the Schema Admin role and must be carefully tested, as it cannot be easily undone.
*   **Disaster Recovery:** Forest Recovery is the "nuclear option." If the entire forest is compromised or corrupted, you must perform a forest-wide recovery, which involves restoring the first DC of the forest from backup and rebuilding the rest of the environment.
*   **Monitoring:** You must monitor the health of the Schema and Configuration partitions. If replication of these partitions fails, the forest becomes inconsistent, leading to authentication failures and service outages.

To verify the current state of your forest configuration, you can use the following PowerShell commands:

Get-ADForest | Select-Object SchemaMaster

### Common Misconceptions
!!! warning "Misconception: Forests are just for grouping domains"
    A common error is believing that forests are merely a way to organize domains. In reality, the forest is the **security boundary**. You cannot "trust" your way out of a forest security boundary; if you need to isolate two environments so that an admin in one cannot affect the other, you *must* use separate forests. Do not create multiple domains in a single forest thinking it provides security isolation—it does not.

### Interview Angle
1.  **"What is the difference between a Domain and a Forest as a security boundary?"**
    *   *Model Answer:* A domain is a boundary for administrative delegation and replication. A forest is the ultimate security boundary. An Enterprise Admin in the forest can override any security setting in any domain. If you need to prevent an admin from one environment from having control over another, you must use separate forests.
2.  **"When would you recommend a multi-forest design?"**
    *   *Model Answer:* Only when there is a hard requirement for security isolation (e.g., a separate forest for a high-security lab or a separate forest for a subsidiary that cannot be trusted) or when there are conflicting schema requirements that cannot be reconciled.
3.  **"Explain the role of the Global Catalog in a multi-domain forest."**
    *   *Model Answer:* The GC provides a forest-wide index of objects. It allows users to search for objects (like finding a user's email address) across the entire forest without needing to query every individual domain controller, which would be inefficient and slow.

### Related Concepts
- [Domains](#domain)
- [Trees](#tree)
- [Trusts](#trusts)
- [Schema](#schema)

## Tree

### Technical Definition
An Active Directory Tree is a hierarchical collection of one or more domains that share a contiguous namespace. For example, if the root domain is `corp.bank.com`, a child domain named `emea.corp.bank.com` belongs to the same tree. The tree structure is defined by the DNS hierarchy. All domains within a tree share a common schema and configuration partition, as they are part of the same forest.

### Underlying Mechanism
The tree structure is enforced by the DNS namespace and the trust relationships between domains. When a new child domain is created, a parent-child trust is automatically established. This is a two-way, transitive Kerberos trust. The "Tree Root" domain is the domain at the top of this contiguous namespace. The mechanism relies on the `msDS-AllowedToDelegateTo` and other trust attributes stored in the `System` container of the domain partition to facilitate cross-domain authentication.

### Why It Exists
The tree structure was designed to mirror organizational structures and provide a logical way to organize domains within a forest. It allows for a contiguous DNS namespace, which simplifies administration and makes it easier for users to understand the relationship between different parts of the organization.

### Enterprise / Banking Reality
In modern banking architectures, the "Tree" concept is largely legacy. While you may still see multi-tree forests in older environments that grew through organic acquisition or departmental silos, the current standard is a single-domain forest. If you see a multi-tree forest in a bank today, it is often a sign of technical debt or a complex M&A history that has not yet been consolidated.

### Operational Considerations
Managing multiple trees within a forest adds complexity to DNS management and trust troubleshooting. You must ensure that DNS zones are correctly delegated and that trust paths are optimized.
*   **DNS Delegation:** Ensure that the parent domain correctly delegates the child domain's DNS zone.
*   **Trust Troubleshooting:** Use `nltest /dsgetdc:<domain>` and `dcdiag /test:outboundtrusts` to verify connectivity between domains in the tree.

### Common Misconceptions
!!! warning "Misconception: Trees provide security boundaries"
    A common misconception is that domains in different trees within the same forest are isolated from each other. They are not. Because they are in the same forest, they share the same schema and configuration, and an Enterprise Admin can manage both. Trees are purely a logical and namespace organization, not a security boundary.

### Interview Angle
1.  **"What is the difference between a Tree and a Forest?"**
    *   *Model Answer:* A forest is the security boundary and contains the schema and configuration. A tree is a collection of domains with a contiguous namespace within that forest. A forest can contain multiple trees.
2.  **"Why would you choose to create a new tree instead of a new child domain?"**
    *   *Model Answer:* You would create a new tree if you needed a non-contiguous namespace (e.g., `bank.com` and `subsidiary.net` in the same forest). This is rarely done today; usually, you would just use a separate forest or a child domain.
3.  **"How does the namespace affect trust?"**
    *   *Model Answer:* The namespace defines the hierarchy of the trust. Parent-child trusts are automatic and transitive. If you have multiple trees, you need a "Forest Trust" or "Shortcut Trust" to bridge the gap if they aren't part of the same contiguous namespace, though within a single forest, the transitive trust is handled by the forest root.

### Related Concepts
- [Forest](#forest)
- [Domains](#domain)
- [DNS](#dns)

## Domain

## OU

## Site

## Why Multiple Domains Existed Historically

## Modern Single-Domain Design Philosophy
