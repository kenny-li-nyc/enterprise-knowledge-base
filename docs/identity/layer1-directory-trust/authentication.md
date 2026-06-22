# Authentication

## 1. Kerberos Core Architecture

### Technical Definition
Kerberos is the primary authentication protocol for Active Directory, functioning as a ticket-based, symmetric-key authentication system. It enables a client to prove its identity to a server (and vice versa) across an insecure network without ever sending the user's password over the wire. The architecture relies on a trusted third party—the Key Distribution Center (KDC)—to issue time-limited, cryptographically signed tickets that grant access to specific services.

### Underlying Mechanism
The Kerberos exchange begins with the AS-REQ/AS-REP (Authentication Service Request/Reply) phase, where the client requests a Ticket Granting Ticket (TGT) from the KDC. The KDC validates the user's credentials against the Active Directory database and returns a TGT encrypted with the KRBTGT account's secret key. Subsequently, the client uses this TGT in the TGS-REQ/TGS-REP (Ticket Granting Service Request/Reply) phase to request a Service Ticket for a specific target resource. This Service Ticket contains the Privilege Attribute Certificate (PAC), which includes the user's SID and group memberships, allowing the target server to authorize the user without querying the domain controller. As noted in Section 1.2, inter-realm Kerberos referrals are handled via cross-realm TGT exchanges, where the KDC of one domain issues a referral ticket to the KDC of the trusted domain.

### Why It Exists
Kerberos was adopted by Microsoft to replace the legacy NTLM protocol, providing a more secure, scalable, and efficient authentication mechanism. By using tickets instead of repeated password exchanges, Kerberos significantly reduces the risk of credential interception and replay attacks. Its architectural design, which separates authentication (TGT) from authorization (Service Ticket), allows for granular access control and delegation, which are essential for modern, distributed enterprise environments.

### Enterprise / Banking Reality
In Tier-1 banking, Kerberos is the bedrock of identity security. The KRBTGT account is the most sensitive object in the forest; if compromised, an attacker can forge TGTs (Golden Ticket attack) and impersonate any user in the domain. Consequently, banking security standards mandate strict rotation of the KRBTGT password and rigorous monitoring of Kerberos traffic for anomalies. Furthermore, the PAC is a critical component for authorization; ensuring its integrity is vital for preventing privilege escalation. Audit requirements often focus on the validation of Kerberos tickets and the monitoring of TGS requests to detect lateral movement.

### Operational Considerations
Monitoring Kerberos health is essential for maintaining authentication availability.
[CLI: klist]
[CLI: Get-EventLog -LogName Security -InstanceId 4768, 4769]
[CLI: Set-DomainObject -Identity "KRBTGT" -ClearPassword]
Regularly auditing the KRBTGT account and monitoring for Kerberos-related errors (e.g., clock skew, ticket expiration) is a standard operational requirement.

### Common Misconceptions
!!! warning
    A common misconception is that Kerberos is "passwordless." This is false. Kerberos still relies on the user's password (or a smart card/certificate) to perform the initial authentication and obtain the TGT. It is a ticket-based protocol, not a passwordless one, although it can be integrated with modern authentication factors to achieve a passwordless experience.

### Interview Angle
1. **Scenario:** You are investigating a potential Golden Ticket attack. What is your first step?
   *Model Answer:* I would immediately rotate the KRBTGT account password twice to invalidate all existing TGTs. This is the only way to ensure that any forged tickets are rendered useless, effectively "resetting" the trust of the Kerberos infrastructure.
2. **Scenario:** How does Kerberos prevent replay attacks?
   *Model Answer:* Kerberos uses timestamps and authenticators within the ticket exchange. If a ticket is intercepted and replayed, the KDC or the target server will reject it because the timestamp is outside the acceptable window or the authenticator has already been used.
3. **Scenario:** Why is clock synchronization critical for Kerberos?
   *Model Answer:* Kerberos relies on timestamps to validate tickets and prevent replay attacks. If the clock on the client, the KDC, or the target server is out of sync (typically by more than 5 minutes), the authentication request will be rejected, leading to widespread authentication failures.

### Related Concepts
*   [Trust Architecture (Section 1.2)] - For details on cross-forest authentication and referrals.
*   [PDC Emulator (Section 1.5)] - For details on password validation and lockout processing.
