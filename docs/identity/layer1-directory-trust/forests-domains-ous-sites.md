# Forests, Domains, OUs & Sites

## Forest

### Technical Definition
An Active Directory (AD) Forest is the highest-level container in the Active Directory logical structure. It represents the ultimate security boundary for an identity environment. A forest is defined by a single, shared schema (the set of all object classes and attributes), a single configuration partition (the physical and logical topology of the environment), and a set of transitive, two-way Kerberos trust relationships between all domains contained within it. 

From an architectural perspective, the forest is the boundary of administrative authority. While delegation can occur within the forest, the forest root domain holds the Enterprise Admins and Schema Admins groups, which possess the capability to modify the forest-wide configuration and schema, effectively granting them control over every object in every domain within that forest.

### Underlying Mechanism
The forest is implemented through three primary directory partitions that are replicated across Domain Controllers (DCs) within the forest:

1.  **Schema Partition:** Contains the definitions of all object classes (e.g., `user`, `computer`) and attributes (e.g., `sAMAccountName`, `memberOf`). This partition is replicated to every DC in the forest.
2.  **Configuration Partition:** Contains the physical and logical topology of the forest, including site definitions, subnets, services, and domain information. This is also replicated to every DC in the forest.
3.  **Domain Partition:** Contains the actual objects (users, computers, groups) for a specific domain. This is only replicated to DCs within that specific domain.

The forest also relies on the **Global Catalog (GC)**. The GC is a special role held by one or more DCs that maintains a partial, read-only copy of every object in every domain within the forest. This allows for forest-wide searches without requiring cross-domain queries for every request. Trust relationships between domains in a forest are automatically created as transitive, two-way Kerberos trusts, allowing for seamless authentication across the entire forest boundary.

