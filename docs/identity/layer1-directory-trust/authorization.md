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
