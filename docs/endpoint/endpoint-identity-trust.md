#  Endpoint Identity & Trust

## 1. Device Identity in AD vs. Entra ID

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

## 2. Device Certificates & TPM-backed Keys

### Technical Definition
Device certificates and TPM-backed keys represent the cryptographic foundation of endpoint trust, ensuring that an identity claim is cryptographically bound to a specific piece of hardware. A TPM (Trusted Platform Module) is a secure microcontroller designed to provide hardware-based, security-related functions, most notably the generation and storage of cryptographic keys. When a device is joined to an identity provider, it generates an asymmetric key pair within the TPM. The private key never leaves the hardware, while the public key is signed by the identity provider to create a device certificate. This certificate serves as a non-repudiable proof of device identity, ensuring that authentication requests originate from the authorized hardware rather than a software-based emulation.

### Underlying Mechanism
The mechanism relies on the interaction between the OS-level Key Storage Provider (KSP) and the TPM firmware. During the device registration process, the OS requests the TPM to generate a key pair. The TPM creates these keys internally, ensuring the private portion is marked as non-exportable. The public key is then sent to the identity provider (such as Entra ID or an on-premises PKI), which issues a certificate. When the device authenticates, the identity provider issues a challenge that must be signed by the private key stored in the TPM. Because the private key is physically bound to the silicon, the authentication process is resistant to credential theft, even if the operating system kernel is compromised. This hardware-backed binding is the core requirement for Windows Hello for Business and modern device-based conditional access.

[DIAGRAM: Sequence diagram showing the TPM key generation, certificate signing request, and the challenge-response authentication flow]

### Why It Exists
The shift toward TPM-backed keys was necessitated by the inadequacy of software-based credentials in the face of sophisticated persistent threats. Software-stored keys, such as those in a standard certificate store, are vulnerable to memory scraping and file-system extraction. By moving the root of trust to hardware, organizations can ensure that even if an attacker gains administrative access to the OS, they cannot extract the private keys required to impersonate the device. This architectural shift is critical for meeting regulatory requirements that mandate hardware-backed security for accessing sensitive financial data and transactional systems.

### Enterprise / Banking Reality
In high-security banking environments, TPM 2.0 is a non-negotiable requirement for all endpoints. Architects must ensure that the entire fleet supports TPM 2.0 and that the BIOS/UEFI configurations are locked down to prevent tampering. The audit and compliance angle is significant: regulators often require proof that cryptographic keys are stored in FIPS 140-2/3 validated hardware. If a device lacks a TPM or has a disabled TPM, it is effectively untrusted and should be blocked from accessing sensitive resources via Conditional Access policies. This creates an operational dependency on hardware procurement standards and firmware lifecycle management.

### Operational Considerations
Operationalizing TPM-backed keys requires robust monitoring of the TPM state across the fleet. Administrators must use tools to verify that the TPM is initialized, owned, and functioning correctly. If a TPM fails or is cleared, the device identity is effectively destroyed, requiring a re-enrollment process. Monitoring for "TPM lockout" events is also critical, as these can indicate brute-force attempts against the hardware. Furthermore, administrators must manage the lifecycle of the certificates associated with these keys, ensuring they are renewed before expiration to prevent service disruption.

[CLI: PowerShell command to query the TPM status, manufacturer, and version information on a Windows endpoint]

### Common Misconceptions
!!! warning
    A common misconception is that BitLocker encryption alone provides sufficient hardware-backed identity. While BitLocker uses the TPM to protect the disk encryption key, it is a separate function from the device identity certificate. Another frequent error is assuming that virtual TPMs (vTPMs) in virtualized environments provide the same level of security as physical TPMs; while they offer similar functionality, they are only as secure as the hypervisor hosting them, which may not meet the strict hardware-isolation requirements of certain banking compliance frameworks.

### Interview Angle
1. Question: How do you handle a scenario where a fleet of legacy devices lacks TPM 2.0 but must access sensitive banking applications?
   Answer: The architectural response is to enforce a "Zero Trust" posture where these devices are treated as untrusted. They should be isolated to a restricted network segment or denied access to sensitive applications entirely, with a clear roadmap for hardware replacement.

## 3. Conditional Access Device Compliance Signals

### Technical Definition
Conditional Access (CA) device compliance signals are dynamic attributes evaluated at the time of authentication to determine if an endpoint meets the organization's security baseline. These signals are derived from the Unified Endpoint Management (UEM) platform—typically Microsoft Intune—which continuously assesses the device against defined policies such as OS version, disk encryption status, antivirus health, and jailbreak/root detection. When a user attempts to access a protected resource, Entra ID queries the device's compliance state; if the device is marked as "compliant," the access request proceeds, otherwise, it is blocked or redirected for remediation.

### Underlying Mechanism
The mechanism relies on a continuous feedback loop between the endpoint, the UEM, and the Identity Provider (IdP). The UEM agent on the device periodically reports telemetry to the cloud service. If the device falls out of compliance (e.g., a user disables the firewall), the UEM updates the device's compliance status in the Entra ID directory. During the authentication handshake, Entra ID evaluates the "isCompliant" claim. If the policy requires a compliant device, the IdP checks this claim in real-time. This is not a static check; it is a dynamic evaluation that can trigger re-authentication if the device state changes during an active session.

[DIAGRAM: Sequence diagram showing the UEM reporting loop, the Entra ID compliance check, and the resulting access decision]

### Why It Exists
This capability exists to enforce a "Zero Trust" posture where access is never granted based solely on user credentials. In a modern enterprise, the device is the primary attack vector. By requiring compliance signals, organizations ensure that even if a user's credentials are stolen, the attacker cannot access sensitive data from an unmanaged, compromised, or non-compliant device. It shifts the security burden from the network perimeter to the endpoint itself, ensuring that the "what" (the device) is as trusted as the "who" (the user).

### Enterprise / Banking Reality
For Tier-1 banks, compliance signals are the primary enforcement mechanism for regulatory frameworks like PCI-DSS or internal risk policies. A common pattern is to require "Compliant" status for access to core banking systems, while allowing "Hybrid Joined" (but potentially non-compliant) status for less sensitive productivity apps. Architects must design these policies to be granular, avoiding "all-or-nothing" approaches that disrupt business operations while maintaining a strict security boundary. This requires careful coordination between the security operations center (SOC) and the endpoint management teams to ensure that compliance policies are both enforceable and realistic.

### Operational Considerations
Operationalizing compliance signals requires careful management of "grace periods." If a device falls out of compliance, users need a window to remediate (e.g., update the OS) before being locked out. Monitoring the "Compliance Policy" dashboard is essential to identify widespread issues, such as a buggy OS update that marks the entire fleet as non-compliant. Administrators must also ensure that the UEM agent is healthy and communicating; if the agent stops reporting, the device may default to a "non-compliant" state, leading to unintended access denials.

[CLI: PowerShell command to check the current compliance status of a device via the Intune management extension]

### Common Misconceptions
!!! warning
    A common misconception is that a "Compliant" device is inherently secure. Compliance only means the device meets the *defined* policy settings; it does not guarantee the device is free from zero-day exploits or advanced persistent threats. Another error is assuming that compliance signals are instantaneous; there is often a latency between a device falling out of compliance and the IdP receiving that signal, which can create a window of vulnerability.

### Interview Angle
1. Question: How do you design a Conditional Access policy that balances security with user productivity?
   Answer: The design should be risk-based. Apply strict compliance requirements to high-value assets (e.g., core banking systems) while allowing more flexibility for general productivity tools, using "Report-only" mode to test policies before enforcement to minimize business disruption.
2. Question: What are the risks of relying solely on compliance signals for access control?
   Answer: The primary risk is "compliance drift" or false positives, where a device is technically compliant but compromised. Compliance signals should be one layer of a defense-in-depth strategy, combined with user behavior analytics (UBA) and threat protection signals.
3. Question: How do you handle "emergency access" scenarios where a device is non-compliant but a user needs immediate access to a critical system?
   Answer: Avoid hard-coding exceptions. Instead, implement a "break-glass" process that involves temporary policy exclusions, which are which are logged, audited, and automatically revoked after a short duration to maintain the integrity of the security posture.

### Related Concepts
- Section 2.3: Device Compliance and Conditional Access Policies

## 4. Device-bound vs. User-bound Credentials

### Technical Definition
Device-bound credentials are cryptographic keys that are physically tied to a specific piece of hardware, such as a Trusted Platform Module (TPM) or a Secure Enclave, ensuring that the credential cannot be exported or used on any other device. In contrast, user-bound credentials—often implemented via FIDO2 roaming authenticators or smart cards—are tied to the user's identity and can be used across multiple devices, provided the user possesses the physical authenticator. This distinction is critical in Zero Trust architectures, as it separates the assurance of the device's integrity from the assurance of the user's identity.

### Underlying Mechanism
Device-bound credentials rely on asymmetric key pairs generated within the hardware security module (HSM) or TPM. The private key is marked as non-exportable, meaning the authentication handshake must occur on the device itself. User-bound credentials, such as those used in FIDO2/WebAuthn, utilize a portable authenticator (e.g., a YubiKey or a mobile device) that holds the private key. When the user authenticates, the authenticator performs a local user verification (e.g., biometric or PIN) and then signs the challenge. The identity provider verifies the signature, confirming both the user's presence and the validity of the credential, regardless of the device being used to access the resource.

[DIAGRAM: Comparison matrix and flow diagram showing the difference between TPM-bound key usage and portable FIDO2 authenticator usage]

### Why It Exists
The separation of device-bound and user-bound credentials exists to provide flexibility in access control without compromising security. Device-bound credentials ensure that sensitive corporate data is only accessed from managed, trusted hardware, mitigating the risk of credential theft from compromised machines. User-bound credentials provide a secure, phishing-resistant method for users to access resources from various locations or devices, supporting modern work-from-anywhere requirements while maintaining high assurance levels.

### Enterprise / Banking Reality
In Tier-1 banking, the strategy is often a hybrid approach. Device-bound credentials (e.g., Windows Hello for Business) are mandated for standard workstations to ensure that only corporate-managed assets can access the internal network. Simultaneously, user-bound credentials (e.g., FIDO2 security keys) are deployed for high-privilege administrative access or for users who require access from multiple devices. This dual-layer approach ensures that even if a user's password is stolen, the attacker cannot gain access without the physical device or the specific hardware-bound credential.

### Operational Considerations
Operationalizing these credentials requires distinct lifecycle management strategies. Device-bound credentials must be managed as part of the device lifecycle (e.g., provisioning during Autopilot, revocation upon device retirement). User-bound credentials require a robust registration and recovery process, as the loss of a physical authenticator can lock a user out of their account. Administrators must implement self-service portals for users to register new authenticators and ensure that the revocation of lost or stolen authenticators is immediate and synchronized across all identity providers.

[CLI: PowerShell command to list registered FIDO2 keys and TPM-backed credentials for a specific user account]

### Common Misconceptions
!!! warning
    A common misconception is that user-bound credentials are less secure than device-bound credentials because they are portable. In reality, modern FIDO2-based user-bound credentials are highly resistant to phishing and man-in-the-middle attacks, often providing a higher level of assurance than traditional passwords. Another error is assuming that device-bound credentials alone are sufficient for all scenarios; they do not protect against an attacker who has physical access to the device and can bypass the OS-level authentication.

### Interview Angle
1. Question: How do you determine when to enforce device-bound vs. user-bound credentials for a specific user group?
   Answer: The decision should be based on the risk profile of the resource and the user's mobility requirements. High-risk, static workstations should enforce device-bound credentials, while mobile users or those requiring access to multiple systems should utilize user-bound, phishing-resistant credentials.
2. Question: What are the primary challenges in managing a large-scale deployment of user-bound FIDO2 keys?
   Answer: The primary challenges are the physical logistics of distribution, the need for a secure self-service registration process, and the requirement for a robust "break-glass" recovery procedure for when a user loses their physical key.
3. Question: How do you ensure that a user-bound credential has not been compromised if the physical authenticator is lost?
   Answer: The identity provider must support immediate revocation of the specific credential ID. This should be coupled with a policy that requires re-verification of the user's identity (e.g., through an alternative MFA method or manager approval) before a new credential can be registered.

### Related Concepts
- Section 1.7: Kerberos and NTLM Authentication Mechanics
- Section 2.1: FIDO2 and Passwordless Authentication
- Section 2.3: Device Compliance and Conditional Access Policies

## 5. Device Attestation & Health Certificates

### Technical Definition
Device attestation is the process by which a device proves its integrity and configuration state to a relying party (such as an identity provider or a management server) using hardware-backed cryptographic evidence. This process typically involves the TPM generating an "attestation quote"—a signed statement containing the current state of the device's boot configuration, firmware, and OS kernel. Health certificates are the resulting artifacts issued by an attestation service (like Microsoft's Device Health Attestation or a private PKI) that certify the device has passed these integrity checks, effectively acting as a "clean bill of health" for the endpoint.

### Underlying Mechanism
The mechanism relies on the TPM's ability to perform "Measured Boot." As the device boots, each component (firmware, bootloader, kernel, drivers) is measured (hashed) and stored in the TPM's Platform Configuration Registers (PCRs). When an attestation request is made, the TPM signs the current PCR values with an Attestation Identity Key (AIK), which is bound to the TPM's Endorsement Key (EK). This signed quote is sent to an attestation service, which verifies the measurements against a known-good baseline. If the measurements match, the service issues a health certificate or a signed claim that the device is in a trusted state, which can then be used to authorize access to sensitive resources.

[DIAGRAM: Sequence diagram showing the Measured Boot process, the TPM signing the PCRs, and the attestation service issuing a health certificate]

### Why It Exists
Device attestation exists to solve the "rootkit" and "firmware-level compromise" problem. Traditional software-based security agents can be disabled or bypassed by malware that gains kernel-level access. By moving the verification of the device's integrity to the hardware (TPM) and the boot process, organizations can ensure that the device has not been tampered with before the OS even loads. This provides a level of assurance that is impossible to achieve with software-only solutions, making it a critical component for high-security environments where the integrity of the endpoint is paramount.

### Enterprise / Banking Reality
In Tier-1 banking, device attestation is often a requirement for accessing the most sensitive transactional systems. Architects must ensure that the attestation service is highly available and that the baseline measurements are kept up-to-date with the latest firmware and OS patches. A common pattern is to integrate attestation results directly into Conditional Access policies: if a device fails attestation (e.g., due to an unauthorized firmware modification), it is automatically blocked from accessing the banking network, regardless of the user's credentials or compliance status.

### Operational Considerations
Operationalizing device attestation requires careful management of the "known-good" baselines. If a firmware update changes the boot measurements, the attestation service must be updated to recognize the new baseline, or the entire fleet will fail attestation. This requires tight integration between the patch management process and the attestation service. Administrators must also monitor for "attestation failures" as a potential indicator of a compromised device or a failed update, and ensure that there is a clear remediation path for devices that fail to attest.

[CLI: PowerShell command to query the current TPM PCR values and verify the device's attestation status]

### Common Misconceptions
!!! warning
    A common misconception is that attestation is a "set and forget" configuration. In reality, it requires continuous maintenance of the baseline measurements. Another error is assuming that attestation protects against all forms of compromise; it only verifies the integrity of the boot process and firmware, not the security of the applications running on top of the OS.

### Interview Angle
1. Question: How do you handle a situation where a legitimate firmware update causes a fleet of devices to fail attestation?
   Answer: The process must include a "pre-flight" phase where new firmware baselines are validated in a test environment and then pushed to the attestation service before the firmware is deployed to the production fleet.
2. Question: What is the difference between device compliance (from Section 3) and device attestation?
   Answer: Compliance is about the *configuration* and *state* of the OS and applications (e.g., is the firewall on?), while attestation is about the *integrity* of the hardware and boot process (e.g., has the firmware been tampered with?).
3. Question: Why is attestation considered a "higher" form of trust than standard compliance?
   Answer: Attestation is rooted in hardware and the boot process, making it much harder to spoof than software-based compliance signals, which can be manipulated by a compromised kernel.

### Related Concepts
- Section 2.2: Device Certificates & TPM-backed Keys
- Section 2.3: Conditional Access Device Compliance Signals
- Section 3.1: Secure Boot and Measured Boot Architecture

## 6. Cross-domain/cross-tenant device trust (multi-org environments)

### Technical Definition
Cross-domain and cross-tenant device trust refers to the architectural capability of allowing an endpoint managed in one identity boundary (e.g., Tenant A) to securely access resources in another identity boundary (e.g., Tenant B) while maintaining a consistent security posture. This is achieved through B2B collaboration settings, cross-tenant access policies, and the propagation of device compliance claims across organizational boundaries. It enables seamless collaboration in scenarios such as mergers, acquisitions, joint ventures, or supply chain integration, where users from one organization must access applications in another without requiring a full identity migration.

### Underlying Mechanism
The mechanism relies on the configuration of "Cross-tenant access settings" within the Entra ID portal. When a user from Tenant A accesses a resource in Tenant B, the IdP in Tenant B evaluates the trust settings configured for Tenant A. If "Trust device compliance claims" is enabled, Tenant B will accept the device compliance status issued by Tenant A's UEM/Intune environment. This requires a pre-established trust relationship where both tenants agree to honor the device identity and compliance signals of the other. The authentication handshake involves the exchange of claims, where the home tenant asserts the device's identity and compliance state, and the resource tenant validates these assertions against its own Conditional Access policies.

[DIAGRAM: Sequence diagram showing the cross-tenant authentication flow, claim exchange, and trust validation between two Entra ID tenants]

### Why It Exists
This capability exists to support the modern, interconnected enterprise. Organizations frequently collaborate with partners, subsidiaries, or acquired entities that maintain their own independent IT infrastructure. Without cross-tenant trust, users would be forced to maintain multiple identities or devices, leading to significant friction and security gaps. By enabling cross-tenant trust, organizations can extend their Zero Trust perimeter to include external identities and devices, ensuring that security policies are enforced consistently regardless of the user's home organization.

### Enterprise / Banking Reality
In Tier-1 banking, cross-tenant trust is a high-risk, high-reward architectural decision. It is essential for M&A activities where the bank needs to integrate an acquired entity's workforce quickly. However, it introduces the risk of "trust transitivity," where a compromise in the partner tenant could potentially impact the bank's own environment. Banking architects must implement strict "least privilege" access controls, ensuring that cross-tenant trust is limited to specific applications and that device compliance requirements are strictly enforced for all external users accessing sensitive data.

### Operational Considerations
Operationalizing cross-tenant trust requires rigorous governance. Administrators must regularly audit cross-tenant access settings to ensure that trust relationships are still valid and that access is not overly permissive. Monitoring is critical; the SOC must be able to distinguish between internal and external access attempts and alert on anomalous behavior originating from trusted partner tenants. Furthermore, there must be a clear process for revoking trust relationships if a partner organization's security posture degrades or if the business relationship ends.

[CLI: PowerShell command to list and audit cross-tenant access settings and trust configurations]

### Common Misconceptions
!!! warning
    A common misconception is that cross-tenant trust is "all or nothing." In reality, it can be highly granular, allowing organizations to trust device compliance for some users while requiring full MFA for others. Another error is assuming that trusting a partner's device compliance signal is equivalent to managing that device; it is not, and the resource tenant remains responsible for defining the policies that govern access to its own resources.

### Interview Angle
1. Question: How do you mitigate the risk of "trust transitivity" when establishing a cross-tenant trust relationship with a partner organization?
   Answer: Mitigation involves implementing granular Conditional Access policies that restrict access to specific applications, enforcing MFA for all cross-tenant access, and continuously monitoring the partner's security posture through threat intelligence and regular audits.
2. Question: What are the key differences between B2B guest access and cross-tenant synchronization?
   Answer: B2B guest access is a lightweight, on-demand model for collaboration, while cross-tenant synchronization is a more robust, automated model for long-term integration, providing a more seamless experience but requiring more complex configuration and governance.
3. Question: How do you handle a scenario where a partner organization's security standards do not meet your bank's requirements?
   Answer: You should not establish a trust relationship. Instead, require the partner to use your organization's managed devices or provide access through a secure, isolated environment (e.g., a VDI solution) that does not rely on cross-tenant trust.

### Related Concepts
- Section 1.9: Identity Federation and Hybrid Synchronization
- Section 2.3: Device Compliance and Conditional Access Policies
- Section 4.1: B2B Collaboration and External Identities
