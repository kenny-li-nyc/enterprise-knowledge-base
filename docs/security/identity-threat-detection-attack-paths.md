# 3.2 Identity Threat Detection & Attack Paths

## 1. KRBTGT Compromise & Golden/Silver Ticket Attacks

### Technical Definition
KRBTGT compromise and the subsequent generation of Golden and Silver tickets represent the most severe form of identity subversion in an Active Directory environment. The KRBTGT account is the service account for the Key Distribution Center (KDC) in a domain; its password hash is used to encrypt and sign all Kerberos Ticket Granting Tickets (TGTs). A Golden Ticket attack occurs when an adversary gains access to the KRBTGT NTLM hash, allowing them to forge TGTs for any user, effectively granting them unlimited, persistent access to the domain. A Silver Ticket attack involves forging a Service Ticket (ST) for a specific service by compromising the NTLM hash of the service account associated with that service, allowing the attacker to impersonate any user to that specific resource without interacting with the KDC.

### Underlying Mechanism
The Kerberos protocol relies on the KDC to issue TGTs, which are encrypted using the KRBTGT account's secret key, as established in Section 1.7. In a Golden Ticket attack, the adversary extracts the KRBTGT NTLM hash from a Domain Controller's memory (typically via LSASS dumping or DCSync). With this key, the attacker can craft a TGT offline, injecting arbitrary PAC (Privilege Attribute Certificate) data that includes high-privilege group memberships (e.g., Domain Admins). Because the TGT is signed with the valid KRBTGT key, the KDC trusts it implicitly upon presentation. Silver Tickets operate similarly but target the service-specific key (e.g., the machine account password for a file server). By forging a service ticket for a specific SPN (Service Principal Name) using the service account's NTLM hash, the attacker bypasses the KDC entirely for that service, as the service itself validates the ticket using its own secret key.

[DIAGRAM: Sequence diagram illustrating the difference between Golden Ticket (KDC-based) and Silver Ticket (Service-based) forgery flows]

### Why It Exists
These attacks exist because the Kerberos protocol, while robust, relies on the absolute secrecy of the long-term keys used to sign and encrypt tickets. If the KRBTGT key is compromised, the entire trust model of the domain collapses, as the KDC can no longer distinguish between legitimate and forged tickets. Silver Tickets exist because services must be able to verify tickets independently of the KDC to maintain performance and availability, creating a distributed trust model where the compromise of a single service account's key grants access to that specific service.

### Enterprise / Banking Reality
In Tier-1 banking, the compromise of the KRBTGT key is considered a "domain-ending" event, necessitating an immediate, enterprise-wide incident response. Regulatory frameworks such as SWIFT CSP and FFIEC emphasize the protection of the identity control plane, and the detection of ticket forgery is a primary focus for security operations centers (SOCs). Banks must implement rigorous monitoring for anomalous Kerberos traffic, such as tickets with unusually long lifetimes or PAC data that does not match the user's actual group memberships. Furthermore, the periodic rotation of the KRBTGT password—a complex operational task—is a mandatory control to limit the window of opportunity for an attacker who may have silently harvested the key.

### Operational Considerations
Operationalizing the defense against ticket attacks requires a combination of proactive hardening and reactive monitoring. Administrators must ensure that Domain Controllers are physically and logically isolated, and that access to the KRBTGT account is restricted to the absolute minimum. Monitoring is critical; administrators must implement high-fidelity alerting for Kerberos events, such as Event ID 4768 (TGT request) and 4769 (Service Ticket request), looking for anomalies like abnormal ticket lifetimes or requests from unexpected sources. Furthermore, administrators must have a clear, tested procedure for rotating the KRBTGT password twice to invalidate all existing tickets, ensuring that this process is performed without causing widespread service disruption.

[CLI: PowerShell command to audit the KRBTGT account and check for recent password changes or anomalies]

### Common Misconceptions
!!! warning
    A common misconception is that Golden Ticket attacks can be detected by simply looking for "bad" tickets. In reality, a well-crafted Golden Ticket is indistinguishable from a legitimate one to the KDC. Another error is assuming that rotating the KRBTGT password once is sufficient; because the KDC maintains the current and previous password versions to allow for replication latency, the password must be rotated twice to fully invalidate all forged tickets.

### Interview Angle
1. Question: How do you detect a Golden Ticket attack in a large, distributed banking environment?
   Answer: We focus on detecting the indicators of the attack rather than the ticket itself. This includes monitoring for anomalous DCSync activity, unusual LSASS access on Domain Controllers, and Kerberos requests with abnormal lifetimes or PAC data that deviates from the user's known group memberships. We also correlate these events with other security signals, such as unusual login times or locations.
2. Question: What is the operational impact of rotating the KRBTGT password, and how do you mitigate it?
   Answer: The primary impact is the potential invalidation of all existing Kerberos tickets, which can cause widespread service disruption. We mitigate this by performing the rotation during a scheduled maintenance window, ensuring that all Domain Controllers have replicated the change, and performing the rotation twice with a sufficient interval to allow for replication latency, as recommended by Microsoft.
3. Question: How do you differentiate between a Golden Ticket and a Silver Ticket attack during an incident investigation?
   Answer: A Golden Ticket attack involves the KDC, meaning the attacker can access any resource in the domain. A Silver Ticket attack is service-specific, meaning the attacker can only access the service associated with the compromised account. We differentiate them by analyzing the logs: Golden Ticket attacks show anomalies in TGT requests (Event ID 4768), while Silver Ticket attacks show anomalies in Service Ticket requests (Event ID 4769) without a corresponding TGT request.

### Related Concepts
- Section 1.7: Authentication Protocols
- Section 3.2: Pass-the-hash / pass-the-ticket techniques
- Section 3.2: Detection signals & anomalous authentication indicators

## 2. Pass-the-Hash / Pass-the-Ticket Techniques

### Technical Definition
Pass-the-Hash (PtH) and Pass-the-Ticket (PtT) are credential reuse techniques that allow an adversary to authenticate to network resources without knowing the plaintext password of the target account. PtH involves using a stolen NTLM hash to authenticate via the NTLM protocol, while PtT involves using a stolen Kerberos ticket (TGT or ST) to authenticate via the Kerberos protocol. These techniques are highly effective for lateral movement, as they allow an attacker to impersonate a user or service account that has already authenticated to a compromised system, bypassing the need for password cracking or brute-force attacks.

### Underlying Mechanism
PtH exploits the NTLM authentication protocol's design, where the client proves its identity by sending a challenge-response based on the NTLM hash of the user's password. If an attacker can extract this hash from the LSASS (Local Security Authority Subsystem Service) process memory on a compromised host, they can inject it into their own session and authenticate to other systems as the victim. PtT exploits the portability of Kerberos tickets. Once a user authenticates, their TGT and STs are stored in memory. An attacker with administrative access to the host can extract these tickets and inject them into their own session, allowing them to access any resource for which the ticket is valid. This process does not require the KDC to issue new tickets, as the attacker is simply reusing existing, valid credentials.

[DIAGRAM: Sequence diagram showing the extraction of credentials from LSASS memory and their subsequent reuse in PtH and PtT attacks]

### Why It Exists
These techniques exist because of the inherent design of authentication protocols that prioritize performance and usability over absolute security. NTLM was designed in an era where network security was less of a concern, and its reliance on hashes rather than plaintext passwords was intended to prevent password exposure. Kerberos tickets were designed to be portable to allow for single sign-on (SSO) across distributed environments. Attackers have simply repurposed these features to facilitate lateral movement, turning the convenience of SSO and NTLM authentication into a significant security liability.

### Enterprise / Banking Reality
In Tier-1 banking, PtH and PtT are the primary methods used by adversaries to move laterally from a compromised workstation to high-value servers and domain controllers. Banks must implement robust defenses, such as Credential Guard (which isolates LSASS in a virtualized container), to prevent the extraction of credentials from memory. Furthermore, the use of tiered administration (as discussed in Section 3.1) is essential to ensure that high-privilege credentials are never exposed on lower-trust systems where they could be harvested. Audit logs must be configured to detect anomalous authentication patterns, such as a user logging into multiple systems from a single workstation in a short period, which is a classic indicator of lateral movement.

### Operational Considerations
Operationalizing the defense against PtH and PtT requires a multi-layered approach. Administrators must enforce the use of modern authentication protocols (e.g., Kerberos over NTLM) wherever possible and disable legacy protocols like NTLM and LLMNR/NetBIOS. Monitoring is critical; administrators must implement high-fidelity alerting for authentication events, such as Event ID 4624 (Logon) with Logon Type 3 (Network) or 9 (NewCredentials), which are often associated with PtH/PtT activity. Furthermore, administrators must ensure that they have a clear, tested procedure for responding to credential theft, including the immediate isolation of compromised hosts and the rotation of credentials for any accounts that may have been exposed.

[CLI: PowerShell command to audit the current authentication protocol usage and identify systems still relying on NTLM]

### Common Misconceptions
!!! warning
    A common misconception is that PtH and PtT are "password cracking" attacks. In reality, they are "credential reuse" attacks; the attacker never needs to know the plaintext password, making traditional password complexity policies ineffective against them. Another error is assuming that these attacks can be fully mitigated by simply changing passwords; while password rotation is important, it does not prevent the reuse of currently valid hashes or tickets.

### Interview Angle
1. Question: How do you prevent credential harvesting from LSASS memory in a large banking environment?
   Answer: We implement Windows Defender Credential Guard, which uses virtualization-based security to isolate the LSA process, making it inaccessible to unauthorized processes, even those running with administrative privileges. We also enforce strict tiered administration, ensuring that high-privilege accounts never log into systems where their credentials could be harvested.
2. Question: What are the key indicators of a Pass-the-Hash attack in your environment?
   Answer: We look for anomalous authentication patterns, such as a user account logging into multiple servers from a single workstation in a short timeframe, or authentication requests using NTLM when Kerberos is expected. We also monitor for the use of tools like Mimikatz or other credential dumping utilities on our endpoints.
3. Question: How do you mitigate the risk of Pass-the-Ticket attacks if you cannot fully disable NTLM?
   Answer: We focus on limiting the exposure of Kerberos tickets by enforcing short ticket lifetimes and using PAWs for all administrative tasks. We also implement strict monitoring for anomalous Kerberos traffic, such as tickets with unusually long lifetimes or requests from unexpected sources, and we ensure that our Domain Controllers are hardened against ticket forgery.

### Related Concepts
- Section 1.7: Authentication Protocols
- Section 3.1: Tiered Admin Model (Tier 0/1/2) & ESAE/Red Forest Concepts
- Section 3.2: KRBTGT Compromise & Golden/Silver Ticket Attacks
