# Forest Services

## Global Catalog

### Technical Definition
The Global Catalog (GC) is a specialized domain controller role that maintains a forest-wide, read-only index of all objects within an Active Directory forest. It serves as the central directory service for forest-wide searches and authentication, allowing users and applications to locate objects across domain boundaries without requiring queries to every individual domain controller in the forest. By providing a unified view of the entire identity environment, the GC acts as the primary directory interface for cross-domain operations.

### Underlying Mechanism
The Global Catalog stores a full, read-write copy of all objects in its own domain and a partial, read-only copy of all objects in every other domain in the forest. This partial copy is defined by the Partial Attribute Set (PAS), which contains a subset of attributes for every object in the forest. As covered in the Replication Architecture page, the replication of the PAS is a forest-wide event that ensures consistency across all GC servers. When a client performs a forest-wide search, the GC server provides the results from its local index, eliminating the need for cross-domain referrals. During the logon process, the GC is queried to resolve Universal Group memberships, which is critical for determining the user's effective permissions in a multi-domain environment.

[DIAGRAM: A sequence diagram showing a client querying a Global Catalog server for a user object located in a different domain]

### Why It Exists
The Global Catalog exists to solve the fundamental problem of search and authentication efficiency in a multi-domain forest. Without a GC, a search for a user or resource in a different domain would require a cross-domain query to that domain's specific domain controllers, which is inefficient and introduces significant latency. The GC provides a single,