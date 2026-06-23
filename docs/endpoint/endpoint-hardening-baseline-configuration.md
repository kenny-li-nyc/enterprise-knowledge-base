# 2.5 Endpoint Hardening & Baseline Configuration

## 1. CIS/STIG/DISA Baseline Benchmarks

### Technical Definition
Baseline benchmarks, such as those provided by the Center for Internet Security (CIS), the Defense Information Systems Agency (DISA) Security Technical Implementation Guides (STIGs), and other regulatory bodies, represent the industry-standard "hardened" configuration state for operating systems and applications. These benchmarks define a comprehensive set of security settings—ranging from password complexity and account lockout policies to advanced audit logging and service restrictions—designed to minimize the attack surface of an endpoint. In a Tier-1 banking context, these benchmarks serve as the foundational security posture, providing a verifiable, repeatable, and defensible configuration that aligns with global regulatory requirements.

### Underlying Mechanism
The implementation of these benchmarks involves mapping the recommended settings to the native configuration interfaces of the operating system, such as the Windows Registry, local security policies, or specific application configuration files. As noted in Section 2.3, the actual enforcement of these settings is handled by the organization's policy delivery vehicles, such as GPOs or MDM profiles, which translate the benchmark requirements into actionable configuration state. The benchmarks themselves are typically delivered as structured documentation or automated configuration templates (e.g., GPO backups or Intune configuration profiles) that administrators import into their management systems to ensure that the endpoint configuration aligns with the desired security baseline.

[DIAGRAM: Flowchart illustrating the mapping of benchmark requirements to policy delivery vehicles and endpoint enforcement]

### Why It Exists
Baseline benchmarks exist to solve the problem of "configuration entropy," where endpoints drift from a secure state over time due to manual changes, software installations, or lack of standardized management. Historically, administrators relied on ad-hoc security configurations, which were often incomplete, inconsistent, and difficult to audit. Benchmarks provide a standardized, peer-reviewed, and widely accepted security baseline that organizations can adopt to ensure a consistent level of protection across their entire fleet, regardless of the specific hardware or software environment.

### Enterprise / Banking Reality
For Tier-1 banks, adopting a recognized baseline like CIS or DISA STIG is not optional; it is a fundamental requirement for compliance with frameworks such as FFIEC, SWIFT CSP, and NIST SP 800-53. These benchmarks provide the "gold standard" against which the bank's security posture is measured during internal and external audits. Architects must tailor these benchmarks to the bank's specific operational needs, ensuring that the hardening measures do not disrupt critical banking applications while maintaining the highest possible level of security. This often involves a "baseline-plus" approach, where the standard benchmark is augmented with additional, bank-specific security controls.

### Operational Considerations
Operationalizing baseline benchmarks requires a continuous, lifecycle-based approach. Administrators must regularly review and update their configuration baselines to align with the latest versions of the benchmarks and the evolving threat landscape. This involves testing the impact of new baseline settings in a staging environment before deploying them to production, ensuring that they do not cause operational issues. Furthermore, administrators must monitor for "baseline drift," where endpoints deviate from the established configuration, and implement automated remediation processes to bring them back into compliance.

[CLI: PowerShell command to audit the current system configuration against a defined security baseline]

### Common Misconceptions
!!! warning
    A common misconception is that applying a baseline benchmark makes a system "unhackable." In reality, benchmarks are only one layer of a defense-in-depth strategy; they reduce the attack surface but do not eliminate all vulnerabilities. Another error is assuming that benchmarks are "set and forget"; they require ongoing maintenance and adjustment to remain effective as the OS and application landscape evolves.

### Interview Angle
1. Question: How do you balance the strict security requirements of a DISA STIG with the operational needs of a high-performance trading environment?
   Answer: The balance is achieved through a risk-based approach. We implement the core security controls of the STIG that are essential for protection, while selectively relaxing or compensating for controls that would negatively impact performance or application stability. This is documented and approved through the bank's formal risk acceptance process.
2. Question: What is the difference between a "baseline" and a "policy" in the context of endpoint hardening?
   Answer: A baseline is the desired, hardened state of the system, while a policy is the mechanism used to enforce that state. The baseline defines *what* the configuration should be, and the policy defines *how* that configuration is delivered and maintained on the endpoint.
3. Question: How do you manage the conflict between different baseline standards (e.g., CIS vs. STIG) in a global banking environment?
   Answer: We adopt a "superset" approach, where we identify the most stringent requirements from each standard and create a unified, internal baseline that meets or exceeds all applicable regulatory requirements. This ensures consistency and simplifies the audit process across different jurisdictions.

### Related Concepts
- Section 1.4: Active Directory Schema and Attribute Management
- Section 2.3: Configuration Management (GPO/MDM)
- Section 2.5: Baseline drift auditing & remediation

## 2. Attack Surface Reduction Rules

### Technical Definition
Attack Surface Reduction (ASR) rules are a set of granular security controls within Microsoft Defender for Endpoint that restrict specific behaviors often exploited by malware and adversarial actors. These rules target common attack vectors, such as the execution of malicious scripts, the launching of child processes from Office applications, and the theft of credentials from the Local Security Authority Subsystem Service (LSASS). By enforcing these restrictions, ASR rules effectively shrink the "attack surface" of an endpoint, preventing malicious code from executing or spreading even if it manages to bypass initial perimeter defenses.

### Underlying Mechanism
ASR rules operate by hooking into the operating system kernel and application processes to monitor for specific, high-risk behaviors. When an application or script attempts to perform a restricted action—such as an Office application attempting to spawn a PowerShell process—the ASR rule intercepts the call and evaluates it against the configured policy. As noted in Section 2.3, these rules are delivered via the organization's policy delivery vehicles (MDM or GPO), which configure the ASR engine on the endpoint. If the action violates the policy, the ASR rule blocks it and logs the event, providing visibility into potential malicious activity without requiring a full security scan.

[DIAGRAM: Sequence diagram showing the ASR rule interception of a malicious process execution attempt]

### Why It Exists
ASR rules exist to mitigate the risk of "living off the land" (LotL) attacks, where adversaries use legitimate, built-in system tools (like PowerShell, WMI, or Office macros) to execute malicious code. Traditional antivirus solutions often struggle to detect these attacks because the tools themselves are trusted. ASR rules shift the focus from detecting *malware* to restricting *malicious behavior*, providing a powerful defense against sophisticated threats that rely on abusing legitimate system functionality.

### Enterprise / Banking Reality
In Tier-1 banking, ASR rules are a critical component of the endpoint security strategy, particularly for preventing ransomware and credential theft. Banks must implement these rules carefully, as they can potentially disrupt legitimate business processes that rely on complex Office macros or administrative scripts. The standard practice is to deploy ASR rules in "Audit" mode first, allowing security teams to analyze the impact on business applications before switching to "Block" mode. This ensures that the bank's security posture is hardened without causing operational downtime, which is essential for maintaining the availability of critical banking services.

### Operational Considerations
Operationalizing ASR rules requires a disciplined, iterative approach. Administrators must use the audit logs to identify legitimate business processes that might be blocked by ASR rules and create exclusions where necessary. This requires close collaboration between the security team and the application owners to ensure that the rules are tuned correctly. Furthermore, administrators must monitor the ASR event logs for blocked actions, which can serve as an early warning system for potential security incidents. The ability to quickly adjust ASR policies in response to new threat intelligence is a key operational capability.

[CLI: PowerShell command to query the current status and configuration of ASR rules on a Windows device]

### Common Misconceptions
!!! warning
    A common misconception is that ASR rules are a replacement for EDR (Endpoint Detection and Response) or antivirus solutions. In reality, ASR rules are a complementary defense-in-depth control; they reduce the attack surface, but they do not provide the comprehensive threat detection and response capabilities of an EDR platform. Another error is assuming that ASR rules can be deployed in "Block" mode without prior testing; this is a recipe for operational disruption and should be avoided at all costs.

### Interview Angle
1. Question: How do you manage the risk of ASR rules blocking legitimate business applications in a banking environment?
   Answer: We use a phased deployment strategy. ASR rules are first deployed in "Audit" mode, and we use the resulting telemetry to identify and whitelist legitimate business processes. Only after we have verified that the rules will not disrupt critical applications do we transition them to "Block" mode.
2. Question: What is the difference between ASR rules and traditional application whitelisting (e.g., AppLocker or WDAC)?
   Answer: Application whitelisting restricts *what* can run (i.e., only signed/approved binaries), while ASR rules restrict *how* applications behave (i.e., preventing specific, high-risk actions). They are complementary controls; whitelisting prevents unauthorized software, while ASR rules prevent authorized software from being abused.
3. Question: How do you handle a situation where an ASR rule blocks a critical business process during an emergency?
   Answer: We have a pre-defined "break-glass" procedure that allows authorized administrators to temporarily disable or modify the ASR policy to restore service. This action is logged, audited, and must be followed by a post-incident review to determine if the policy needs to be permanently adjusted or if the business process needs to be re-engineered.

### Related Concepts
- Section 2.3: Configuration Management (GPO/MDM)
- Section 3.2: EDR and Threat Detection
- Section 2.5: CIS/STIG/DISA Baseline Benchmarks
