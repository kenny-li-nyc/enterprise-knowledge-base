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

## 2. NTLM Fallback Mechanics

### Technical Definition
NTLM (NT LAN Manager) is a legacy challenge-response authentication protocol that serves as the fallback mechanism when Kerberos authentication fails or is unavailable. It relies on a three-way handshake (Negotiate, Challenge, Response) to verify identity without transmitting the user's password, though it is significantly less secure than Kerberos due to its susceptibility to relay and brute-force attacks.

### Underlying Mechanism
The negotiation process is governed by SPNEGO (Simple and Protected GSSAPI Negotiation Mechanism). When a client attempts to connect to a service, it first attempts to negotiate Kerberos. If the client lacks a Service Principal Name (SPN) for the target, if the target is an IP address, or if the Kerberos handshake fails, the client falls back to NTLM. The NTLM exchange involves the server sending a challenge to the client, which the client encrypts with its password hash and returns as a response. The server then forwards this response to the domain controller for validation.

### Why It Exists
NTLM exists primarily for backward compatibility with legacy applications, non-domain-joined clients, and scenarios where Kerberos cannot be used (e.g., accessing resources via IP address or across untrusted network boundaries). It was the default authentication protocol in Windows NT 4.0 and remains embedded in the Windows ecosystem to ensure that legacy systems can still authenticate within a modern Active Directory environment.

### Enterprise / Banking Reality
In Tier-1 banking, NTLM is a significant security liability. It is the primary vector for NTLM Relay attacks, where an attacker intercepts the NTLM challenge-response and relays it to another service to gain unauthorized access. Banking security policies (e.g., PCI-DSS, SWIFT CSP) strongly mandate the elimination of NTLM in favor of Kerberos or modern authentication. This involves auditing NTLM usage, implementing SMB signing, and enforcing NTLM restrictions via Group Policy to minimize the attack surface.

### Operational Considerations
Identifying and eliminating NTLM usage is a continuous operational effort.
[CLI: Get-EventLog -LogName Security -InstanceId 4776]
[CLI: Set-SmbServerConfiguration -RequireMessageSigning $true]
Monitoring for NTLM authentication events is critical for identifying legacy applications that still rely on the protocol.

### Common Misconceptions
!!! warning
    A common misconception is that disabling NTLM will only break "old" applications. This is false. Many modern applications and services still rely on NTLM for specific authentication scenarios, such as accessing resources via IP address or in workgroup environments. Disabling NTLM without a thorough audit and testing plan can lead to widespread service outages.

### Interview Angle
1. **Scenario:** You are tasked with eliminating NTLM from your environment. What is your strategy?
   *Model Answer:* I would start by auditing NTLM usage using event logs to identify the applications and services that rely on it. Then, I would work with application owners to migrate these services to Kerberos or modern authentication. Finally, I would implement NTLM restrictions in phases, starting with the most sensitive environments, while monitoring for any impact.
2. **Scenario:** Why is NTLM more susceptible to relay attacks than Kerberos?
   *Model Answer:* NTLM does not provide the same level of mutual authentication and ticket-based security as Kerberos. Because the NTLM response is tied to the user's password hash and not a time-limited ticket, an attacker can capture the response and relay it to another service, effectively impersonating the user.
3. **Scenario:** What is the role of SPNEGO in the authentication negotiation?
   *Model Answer:* SPNEGO is the mechanism that allows the client and server to negotiate the authentication protocol. It enables the client to propose Kerberos first and, if that fails, fall back to NTLM, ensuring a seamless authentication experience even when Kerberos is not available.

### Related Concepts
*   [Kerberos Core Architecture (Section 1.7)] - For understanding the preferred authentication protocol.
*   [Trust Architecture (Section 1.2)] - For understanding how NTLM authentication behaves across trust boundaries.
