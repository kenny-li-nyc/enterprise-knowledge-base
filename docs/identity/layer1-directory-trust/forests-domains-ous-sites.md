# Forests, Domains, OUs & Sites

## Forest

### Technical Definition
An Active Directory (AD) Forest is the highest-level container in the Active Directory logical structure. It represents a single security boundary, a single schema, and a single configuration partition. All domains within a forest share a common schema (the definition of all object classes and attributes) and a common configuration (the topology of the forest, including sites, subnets, and services). The forest is the ultimate boundary for administrative authority; a user with Domain Admin rights in one domain cannot inherently manage objects in another domain within the same forest, but a user with Enterprise Admin rights has authority across the entire forest.

### Underlying Mechanism
The forest is implemented through three primary partitions replicated across all Domain Controllers (DCs) in the forest:

1.  **Schema Partition:** Defines the rules for the directory (what objects exist, what attributes they have). This is replicated to every DC in the forest.
2.  **Configuration Partition:** Contains the physical and logical topology of the forest (sites, subnets, services, and domain information). This is also replicated to every DC in the forest.
3.  **Domain Partition:** Contains the actual objects (users, computers, groups) for a specific domain. This is only replicated to DCs within that specific domain.

The forest also relies on the **Global Catalog (GC)**