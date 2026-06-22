# Device Lifecycle & Provisioning

## 1. Procurement-to-decommission lifecycle stages

### Technical Definition
The procurement-to-decommission lifecycle encompasses the entire operational lifespan of an endpoint device, from the initial acquisition and supply-chain integration to its eventual secure retirement. This lifecycle is a structured sequence of phases—procurement, staging, deployment, operational management, and decommissioning—designed to ensure that every device is accounted for, secured, and compliant with organizational policies from the moment it leaves the manufacturer until it is physically or logically destroyed.

### Underlying Mechanism
Internally, this lifecycle is managed through a combination of asset management databases (CMDB), Mobile Device Management (MDM) platforms, and directory services. When a device is procured, its unique hardware identifier (e.g., serial number, hardware hash) is registered in the vendor's portal (like Apple Business Manager or Windows Autopilot) and subsequently linked to the organization's tenant. As the device moves through the lifecycle, its state is tracked via status flags in the MDM, which trigger automated workflows—such as domain joining or certificate enrollment—based on the device's current phase. This state-machine approach ensures that a device cannot bypass security controls, as the MDM enforces compliance policies at every transition point.

### Why It Exists
The lifecycle model exists to mitigate the risks associated with unmanaged hardware, which is a primary vector for data exfiltration and unauthorized network access. By formalizing the stages of a device's life, organizations can enforce consistent security baselines, maintain an accurate inventory for audit purposes, and ensure that sensitive data is properly sanitized before a device is repurposed or retired. It transforms the endpoint from a "black box" into a managed asset with a verifiable chain of custody.

### Enterprise / Banking Reality
In Tier-1 banking, the device lifecycle is not merely an IT operational task but a regulatory requirement under frameworks like BCBS 239 and SOX, which demand precise asset tracking and data protection. Banking environments utilize "immutable supply-chain security," where devices are shipped directly from the manufacturer to the end-user, with the hardware hash pre-registered in the tenant to prevent interception. The decommissioning phase is equally critical; banking standards require strict adherence to NIST SP 800-88 R1 guidelines for media sanitization, ensuring that no residual data can be recovered from retired assets, often requiring a certificate of destruction for audit compliance.

### Operational Considerations
Effective lifecycle management requires tight integration between procurement, IT operations, and security teams.
[CLI: Get-AutopilotDevice -SerialNumber <SerialNumber>]
[CLI: Invoke-DeviceWipe -DeviceId <DeviceId>]
Monitoring the lifecycle requires real-time dashboards that track device status, compliance, and age, allowing for proactive replacement cycles and identifying "orphaned" devices that have fallen out of the management loop.

### Common Misconceptions
!!! warning
    A common misconception is that "decommissioning" simply means deleting the device object from Active Directory or the MDM. This is false. Decommissioning is a multi-faceted process that includes revoking certificates, wiping the device's storage according to regulatory standards, and updating the asset inventory. Simply deleting the object leaves the physical hardware and any residual data in an unmanaged, potentially compromised state.

### Interview Angle
1. **Scenario:** You are tasked with implementing a zero-touch procurement process for a global bank. How do you ensure the security of the supply chain?
   *Model Answer:* I would implement a "direct-to-user" shipping model where devices are shipped directly from the manufacturer to the employee. I would require the manufacturer to provide the hardware hashes for pre-registration in our tenant, ensuring that the device is automatically enrolled and secured the moment it connects to the internet, without ever touching our internal staging environment.
2. **Scenario:** An auditor asks how you prove that retired devices do not contain sensitive banking data. What is your response?
   *Model Answer:* I would provide the audit team with our documented decommissioning policy, which mandates adherence to NIST SP 800-88 R1 for media sanitization. I would also present the certificates of destruction provided by our certified IT asset disposition (ITAD) partner, which verify that the storage media was physically or logically destroyed.
3. **Scenario:** How do you handle the "orphaned device" problem in a large enterprise?
   *Model Answer:* I would implement an automated "heartbeat" monitoring system that flags devices that have not checked in with the MDM for a defined period (e.g., 30 days). These devices are then automatically moved to a "quarantine" OU, as discussed in Section 1.1, and their access is revoked until they are accounted for or decommissioned.

### Related Concepts
*   [Directory Structure (Section 1.1)] - For context on OU-based administrative inheritance boundaries for staged devices.
*   [Directory Authorization & Delegation (Section 1.8)] - For the scoped permission sets required for enrollment agents to register devices.

## 2. Imaging vs. zero-touch provisioning (Autopilot/DEP/Android Enterprise)

### Technical Definition
Imaging is the legacy practice of deploying a monolithic, pre-configured OS image to a device, often requiring physical access or PXE boot infrastructure. Zero-touch provisioning (ZTP) is the modern, cloud-native paradigm where devices are automatically configured upon first boot via cloud services like Windows Autopilot, Apple Device Enrollment Program (DEP), or Android Enterprise, bypassing the need for custom images.

### Underlying Mechanism
Imaging relies on capturing and deploying a static OS state, which is then "pushed" to the device. ZTP, conversely, leverages hardware-bound identifiers (e.g., hardware hashes, serial numbers) registered in a cloud portal. Upon the device's first network connection, it queries the portal, receives an enrollment profile, and automatically pulls its configuration, policies, and applications from the MDM. This process ensures the device starts from a clean, vendor-provided OS state, which is then transformed into the corporate state via policy application, rather than image application.

### Why It Exists
Imaging was a necessary response to the limitations of early network bandwidth and the lack of cloud-based management tools. ZTP exists to eliminate the "IT staging" bottleneck, enabling remote, secure, and scalable deployments that ensure every device starts from a clean, vendor-provided OS state, which is inherently more secure and easier to maintain than a custom image that requires constant patching and testing.

### Enterprise / Banking Reality
In Tier-1 banking, imaging is increasingly viewed as technical debt and a security risk due to the difficulty of maintaining "golden images" and the potential for image-based malware. ZTP is the standard for modern banking endpoints, as it provides an immutable, audit-ready deployment path that aligns with regulatory requirements for secure, verifiable device configuration. Banking environments prioritize ZTP because it allows for rapid, secure deployment to a global workforce without the need for centralized staging facilities.

### Operational Considerations
Transitioning to ZTP requires a shift from image management to policy management, where the focus is on defining the desired state rather than the initial state.
[CLI: Get-AutopilotProfile -Name "BankingStandard"]
[CLI: Set-DeviceEnrollmentProfile -ProfileName "BankingStandard" -Assignment "AllDevices"]
Monitoring the success of ZTP deployments is critical, as is ensuring that the cloud portal is correctly configured and that the MDM is ready to receive the device.

### Common Misconceptions
!!! warning
    A common misconception is that ZTP is simply "imaging in the cloud." This is false. ZTP is a fundamental shift from managing static images to managing dynamic, policy-driven configurations, which requires a different mindset and operational model. Imaging is about "what the device looks like," while ZTP is about "what the device is allowed to do."

### Interview Angle
1. **Scenario:** You are leading a migration from legacy imaging to Windows Autopilot. What is the biggest architectural challenge?
   *Model Answer:* The biggest challenge is the shift from image-based deployment to policy-based configuration. We must ensure that all applications, security policies, and configurations are correctly defined in the MDM before we can retire the imaging infrastructure. It requires a thorough audit of our current image and a mapping of those settings to modern MDM policies.
2. **Scenario:** Why is ZTP considered more secure than imaging?
   *Model Answer:* ZTP starts from a clean, vendor-provided OS state, which is inherently more secure than a custom image that may contain outdated software, misconfigurations, or even embedded malware. ZTP also ensures that the device is enrolled in the MDM from the very first boot, providing immediate visibility and control.
3. **Scenario:** How do you handle applications that are not yet compatible with ZTP?
   *Model Answer:* I would use a phased approach, where ZTP is used for the OS and core security policies, and legacy applications are delivered via a secondary mechanism, such as application virtualization or a just-in-time delivery service. This allows us to move to ZTP while still supporting the necessary legacy applications.

### Related Concepts
*   [Directory Structure (Section 1.1)] - For context on OU-based administrative inheritance boundaries for staged devices.
*   [Directory Authorization & Delegation (Section 1.8)] - For the scoped permission sets required for enrollment agents to register devices.

## 3. Device enrollment models (corporate, BYOD, shared/kiosk)

### Technical Definition
Device enrollment models define the relationship between the organization and the endpoint, dictating the level of management, security control, and user privacy. Corporate-owned models provide full management and control; Bring Your Own Device (BYOD) models focus on containerization and data separation; and Shared/Kiosk models are designed for multi-user, restricted-access scenarios.

### Underlying Mechanism
Corporate enrollment typically uses device-level management (e.g., DEP, Autopilot) to enforce full control over the OS, including security policies, application deployment, and remote wipe capabilities. BYOD enrollment uses Mobile Application Management (MAM) or "Work Profiles" to create a secure, encrypted container for corporate data, leaving the personal data untouched. Shared/Kiosk models utilize specialized enrollment profiles that restrict the device to a single application or a limited set of applications, often with a "guest" or "multi-user" profile that resets the device state after each session.

### Why It Exists
These models exist to accommodate the diverse needs of the modern workforce, balancing the requirement for security and compliance with the need for flexibility and user privacy. By offering different enrollment models, organizations can tailor their management strategy to the specific risk profile of the device and the user, ensuring that corporate data is protected without unnecessarily infringing on personal privacy or hindering productivity.

### Enterprise / Banking Reality
In Tier-1 banking, the enrollment model is a critical security decision. Corporate-owned devices are the standard for high-risk roles (e.g., traders, developers, executives) and are subject to full management and monitoring. BYOD is often restricted to low-risk roles (e.g., contractors, general staff) and is limited to email and collaboration tools via MAM. Shared/Kiosk devices are used for branch operations, teller machines, and customer-facing kiosks, where security is enforced through strict application whitelisting and session-based resets. Compliance frameworks require that each model be documented, with clear policies on what data can be accessed and what security controls are enforced.

### Operational Considerations
Managing multiple enrollment models requires a robust MDM/UEM platform that can enforce different policies based on the enrollment type.
[CLI: Set-DeviceEnrollmentRestriction -Platform "iOS" -AllowPersonalDevices $false]
[CLI: Get-DeviceEnrollmentProfile -Type "Kiosk"]
Monitoring compliance across different enrollment models is essential, as is ensuring that the appropriate security policies are applied to each device type.

### Common Misconceptions
!!! warning
    A common misconception is that BYOD means "unmanaged." This is false. BYOD is a management model, not a lack of management. Even in BYOD, the organization must enforce security policies on the corporate data container, such as requiring MFA, preventing data copy-paste, and ensuring that the device is not jailbroken or rooted.

### Interview Angle
1. **Scenario:** You are designing an enrollment strategy for a bank. How do you decide which model to use for a specific user group?
   *Model Answer:* I would perform a risk assessment based on the user's role, the data they access, and the device they use. High-risk roles require corporate-owned devices with full management. Low-risk roles can be supported via BYOD with MAM policies. Kiosk devices are reserved for specific, restricted-use cases.
2. **Scenario:** How do you ensure that personal data is protected in a BYOD model?
   *Model Answer:* I would use MAM policies to create a secure, encrypted container for corporate data. This container is logically separated from the personal data, and the MDM has no visibility into or control over the personal apps or data on the device.
3. **Scenario:** What are the security risks of shared/kiosk devices?
   *Model Answer:* Shared/kiosk devices are vulnerable to physical tampering, unauthorized application installation, and session-based data leakage. I would mitigate these risks by using a restricted OS profile, disabling unnecessary hardware ports, and ensuring that the device state is reset after each session.

### Related Concepts
*   [Directory Structure (Section 1.1)] - For context on OU-based administrative inheritance boundaries for staged devices.
*   [Directory Authorization & Delegation (Section 1.8)] - For the scoped permission sets required for enrollment agents to register devices.
