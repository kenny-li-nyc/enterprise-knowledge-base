# 2.2 Endpoint Identity & Trust

## Topic 1: Device Identity in AD vs. Entra ID

### Technical Definition
Device identity serves as the foundational anchor for Zero Trust architecture, distinguishing between three primary states of endpoint registration: Microsoft Entra joined, Hybrid Microsoft Entra joined, and Microsoft Entra registered. An Entra joined device is cloud-native, where the device identity exists solely within the Entra ID tenant, and the user authenticates directly against the cloud directory. A Hybrid Microsoft Entra joined device maintains a dual identity: it is joined to an on-premises Active Directory (AD) domain and simultaneously registered in Entra ID, allowing for legacy Kerberos-based authentication on-premises while leveraging cloud-based Conditional Access policies. A Microsoft Entra registered device, often referred to as Workplace Joined, is typically a personal or mobile device where the user adds their work account to the OS, establishing a lightweight identity relationship that allows for device-level compliance checks without full management or domain membership.

### Underlying Mechanism
The registration process for these identities relies on the Device Registration Service (DRS) and the issuance of a Primary Refresh Token (PRT). For Entra joined devices, the Windows Out-of-Box Experience (OOBE) or Autopilot process triggers a handshake with the Entra ID service, resulting in the creation of a device object and the issuance of a device-bound certificate stored in the Trusted Platform Module (TPM). Hybrid join leverages the on-premises AD synchronization engine to push the device object to the cloud, where the Entra Connect service completes the registration. As noted in Section 1.9, the federation engine facilitates the trust relationship required to synchronize these objects while maintaining the integrity of the on-premises directory. The PRT acts as the primary artifact for single sign-on (SSO) across cloud resources, effectively binding the user's session to the specific device identity.

[DIAGRAM: Flowchart illustrating the registration handshake between client, on-premises AD, and Entra ID for hybrid vs. cloud-native join types]

### Why It Exists
The evolution of device identity was driven by the shift from perimeter-based security to identity-centric access control. Historically, domain membership was the sole mechanism for enforcing group policy and managing trust, but this model failed to address the needs of a mobile, remote workforce accessing SaaS applications. The hybrid model was introduced as a bridge, allowing enterprises to maintain legacy application compatibility—which often relies on NTLM or Kerberos—while simultaneously enforcing modern security postures like Multi-Factor Authentication (MFA) and device compliance for cloud resources. Entra-joined devices represent the architectural destination, removing the dependency on line-of-sight to domain controllers and enabling a pure cloud-native management plane.

### Enterprise / Banking Reality
In Tier-1 banking environments, the choice of device identity is rarely about convenience and almost exclusively about risk mitigation and regulatory compliance. Hybrid join is the standard for legacy-heavy environments where internal applications require Kerberos authentication, but it introduces complexity in the form of synchronization latency and potential "stale" device objects. Principal architects must balance the need for FIPS 140-2/3 compliant hardware-backed identity with the operational reality of managing thousands of endpoints. Compliance mandates often require that only compliant, managed devices can access sensitive transactional systems, making the device identity the primary gatekeeper for Conditional Access policies.

### Operational Considerations
Managing device identities requires rigorous lifecycle management, particularly regarding the cleanup of stale objects that can accumulate in both on-premises AD and Entra ID. Monitoring the health of the Entra Connect sync cycle is critical, as a failure here prevents new devices from receiving the necessary identity artifacts. Administrators must also account for the rotation of device secrets and the monitoring of PRT issuance failures, which are often the first indicators of a broader identity synchronization issue. As discussed in Section 1.7, the underlying authentication protocols remain the bedrock of these interactions, but the operational focus shifts to the cloud-based device management plane.

[CLI: PowerShell command to verify the AzureADJoin status and PRT acquisition state of a local machine]

### Common Misconceptions
!!! warning
    A common misconception is that Hybrid Microsoft Entra joined devices provide the same level of security as Entra joined devices. In reality, Hybrid join inherits the security vulnerabilities of the on-premises Active Directory, including potential exposure to credential harvesting attacks that target the domain controller. Another frequent error is assuming that "Registered" devices are fully managed; they are not, and they lack the ability to enforce device-level configuration profiles or disk encryption mandates required for high-security banking endpoints.

### Interview Angle
1. Question: How would you architect a migration from Hybrid-joined to Entra-joined for a legacy banking environment with significant on-premises dependencies?
   Answer: The strategy involves a phased approach: first, decoupling cloud-native applications from on-premises Kerberos dependencies, then implementing Cloud Kerberos Trust or Windows Hello for Business to handle legacy auth requirements, and finally transitioning endpoints to Entra-joined while maintaining a minimal on-premises footprint for legacy infrastructure.
2. Question: What are the specific risks associated with stale device objects in a hybrid environment, and how do you mitigate them?
   Answer: Stale objects can lead to Conditional Access bypasses or incorrect compliance reporting. Mitigation involves implementing automated cleanup scripts for on-premises AD and configuring Entra ID lifecycle management policies to prune inactive device objects after a defined period of inactivity.
3. Question: Why is the TPM-backed PRT considered the "holy grail" of endpoint identity, and what happens if it is compromised?
   Answer: The PRT is the primary artifact for SSO; if compromised, an attacker can impersonate the device and the user, bypassing MFA. TPM-backing ensures the PRT cannot be exported, mitigating the risk of theft even if the OS is compromised.

### Related Concepts
- Section 1.7: Kerberos and NTLM Authentication Mechanics
- Section 1.9: Identity Federation and Hybrid Synchronization
- Section 2.3: Device Compliance and Conditional Access Policies
