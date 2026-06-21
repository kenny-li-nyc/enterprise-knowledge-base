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

