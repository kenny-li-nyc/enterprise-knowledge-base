# Authorization

## 1. ACLs/DACLs

### Technical Definition
Access Control Lists (ACLs) are the fundamental mechanism for securing objects within Active Directory. An ACL is an ordered list of Access Control Entries (ACEs) attached to an object, which defines the permissions granted or denied to specific security principals (users, groups, computers). The Discretionary Access Control List (DACL) is the specific component of the security descriptor that dictates who can perform which actions on an object, while the System Access Control List (SACL) governs auditing.

### Underlying Mechanism
Every object in Active Directory has a security descriptor, which contains the DACL and SACL. When a security principal attempts to access an object, the Local Security Authority (LSA) on the domain controller evaluates the DACL. The evaluation process is sequential: the system checks the ACEs in order, starting from the top. If an explicit "Deny" ACE is encountered that matches the principal, access is immediately denied. If an "Allow" ACE is found, access is granted, provided no subsequent "Deny" ACE overrides it. If the end of the DACL is reached without an explicit "Allow," access is implicitly denied. As discussed in Section 1.1, these permissions are subject to inheritance, where child objects automatically inherit the DACL from their parent Organizational Unit (OU) unless inheritance is explicitly blocked.

### Why It Exists
ACLs and DACLs exist to enforce the principle of least privilege within the directory. By providing a granular, object-level security model, Active Directory allows administrators to delegate specific administrative tasks—such as password resets or group membership management—without granting full domain-wide administrative rights. This architectural design is essential for maintaining the security and integrity of the directory in large, multi-admin environments.

### Enterprise / Banking Reality
In Tier-1 banking, DACL management is a critical compliance control. Regulatory frameworks like SOX and GLBA require strict enforcement of least privilege, meaning that access to sensitive objects (e.g., financial application service accounts, high-value security groups) must be tightly controlled and audited. Banking environments often implement "AdminSDHolder" protection to ensure that critical administrative accounts have a hardened, non-inheritable DACL, preventing unauthorized modification. Audit teams frequently review DACLs to identify "over-privileged" accounts, making the ability to programmatically audit and remediate ACLs a core operational requirement.

### Operational Considerations
Managing ACLs requires a disciplined approach to avoid security gaps or accidental lockouts.
[CLI: Get-Acl -Path "AD:\CN=Users,DC=example,DC=com"]
[CLI: Set-Acl -Path "AD:\CN=Users,DC=example,DC=com" -AclObject $acl]
Monitoring for changes to DACLs on sensitive objects is vital for detecting unauthorized privilege escalation.

### Common Misconceptions
!!! warning
    A common misconception is that "Deny" ACEs should be used frequently to restrict access. This is false. "Deny" ACEs should be used sparingly, as they can lead to complex, unpredictable access patterns and are difficult to troubleshoot. The preferred approach is to use "Allow" ACEs on specific groups, adhering to the principle of least privilege.

### Interview Angle
1. **Scenario:** You are auditing a domain and find that a standard user group has "Full Control" over a sensitive OU. How do you remediate this?
   *Model Answer:* I would first identify the business requirement for this access. If it is not required, I would remove the "Full Control" ACE and replace it with the minimum necessary permissions (e.g., "Read" or "List Contents"). I would then document the change in our Change Management system and verify that no critical applications are broken by the reduction in permissions.
2. **Scenario:** What is the difference between a DACL and a SACL?
   *Model Answer:* A DACL defines who has access to an object and what they can do (e.g., Read, Write, Delete). A SACL defines what actions should be audited (e.g., successful or failed attempts to access the object). DACLs are for authorization; SACLs are for accountability.
3. **Scenario:** How does inheritance affect DACL management?
   *Model Answer:* Inheritance allows child objects to automatically receive the DACL of their parent OU. This simplifies management but can lead to "permission creep" if not carefully controlled. I would use inheritance for standard permissions and only block inheritance on objects that require unique, non-standard security configurations.

### Related Concepts
*   [Directory Structure (Section 1.1)] - For context on OU-based administrative inheritance boundaries.
*   [Trust Architecture (Section 1.2)] - For understanding how cross-forest SIDs are validated within an authentication token.

## 2. Group-based access models

### Technical Definition
Group-based access models are the primary mechanism for aggregating security principals (users, computers, and other groups) to simplify the assignment of permissions. Instead of assigning access rights to individual users, administrators assign rights to groups, and users are then added to those groups. This abstraction layer decouples the identity of the user from the permissions they hold, enabling scalable, role-based access control (RBAC).

### Underlying Mechanism
When a user authenticates, the KDC compiles a list of all SIDs associated with that user—including their own SID and the SIDs of all groups they belong to—into the Privilege Attribute Certificate (PAC) within their Kerberos ticket. When the user accesses a resource, the resource server evaluates the object's DACL against the SIDs present in the user's PAC. This process is highly efficient because the resource server does not need to query the domain controller for group membership at the time of access; the group membership is "baked into" the user's security token.

### Why It Exists
Group-based access exists to solve the management complexity of individual permission assignment. In an enterprise with thousands of users, managing permissions on a per-user basis is operationally impossible and prone to error. Groups provide a logical structure that mirrors business roles, allowing administrators to grant or revoke access to entire departments or project teams with a single change to group membership.

### Enterprise / Banking Reality
In Tier-1 banking, group-based access is the foundation of RBAC. Banking regulations require that access be granted based on job function, and groups are the primary tool for implementing this. However, "group bloat" and "nested group complexity" are significant risks. Banking environments must implement strict group lifecycle management, including regular attestation (reviewing group membership) and automated provisioning/deprovisioning. Audit teams focus heavily on group membership changes, particularly for high-privilege groups, to detect unauthorized access.

### Operational Considerations
Effective group management requires a clear strategy and consistent tooling.
[CLI: Add-ADGroupMember -Identity "Finance_Users" -Members "jdoe"]
[CLI: Get-ADGroupMember -Identity "Finance_Users"]
Monitoring for changes to sensitive groups is critical for security.
[DIAGRAM: A diagram showing the relationship between users, groups, and resources, illustrating how group membership simplifies permission assignment.]

### Common Misconceptions
!!! warning
    A common misconception is that the "Everyone" or "Authenticated Users" groups are safe to use for general access. This is false. These groups include all users in the domain (or forest), and granting them permissions can lead to massive security vulnerabilities. Always use specific, purpose-built groups for access control.

### Interview Angle
1. **Scenario:** You are designing a group strategy for a large, multi-domain forest. What is your approach to group nesting?
   *Model Answer:* I would adopt a standard AGDLP (Account, Global, Domain Local, Permission) strategy. This ensures that group nesting is predictable and manageable across domains. I would use Global groups to organize users by role, and Domain Local groups to assign permissions to resources, nesting the Global groups into the Domain Local groups.
2. **Scenario:** How do you mitigate the risk of "token bloat" in a large environment?
   *Model Answer:* Token bloat occurs when a user is a member of too many groups, causing their Kerberos ticket to exceed the maximum size. I would mitigate this by implementing a strict group nesting policy, using Universal groups sparingly, and regularly auditing group memberships to remove unnecessary assignments.
3. **Scenario:** Why is group attestation important in a banking environment?
   *Model Answer:* Group attestation is the process of periodically reviewing group memberships to ensure that users still require the access they have been granted. It is a critical control for maintaining least privilege and meeting regulatory requirements for access governance.

### Related Concepts
*   [Directory Structure (Section 1.1)] - For context on OU-based administrative inheritance boundaries.
*   [Trust Architecture (Section 1.2)] - For understanding how cross-forest SIDs are validated within an authentication token.
