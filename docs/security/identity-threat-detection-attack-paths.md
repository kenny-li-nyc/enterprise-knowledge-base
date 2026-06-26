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
