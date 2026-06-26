# 3.8 Governance, Risk & Compliance (GRC)

## 1. Security policy framework & lifecycle

### Technical Definition
The security policy framework is the top-level governance structure that defines the organization's security posture, objectives, and mandatory requirements. It serves as the authoritative source of truth for security expectations, encompassing high-level policies, detailed standards, and actionable procedures. The policy lifecycle refers to the systematic process of drafting, reviewing, approving, publishing, and periodically updating these documents to ensure they remain aligned with the evolving threat landscape, business objectives, and regulatory mandates.

### Underlying Mechanism
The mechanism relies on a centralized GRC platform that acts as the system of record for the entire policy lifecycle. This platform integrates with document management systems to track versioning, approval workflows, and policy distribution. Policy engines within the GRC platform map high-level policy requirements to specific technical controls, which are then validated through automated evidence pipelines that pull data from the SIEM and other monitoring systems, as referenced in Section 3.6 for telemetry gathering. The lifecycle is managed through automated workflows that trigger periodic reviews, ensuring that policies are not just created but actively maintained. This framework provides the governance abstraction layer that dictates how data classification policies, as defined in Section 3.7, must be applied across the enterprise, ensuring that the technical implementation of data protection is always backed by formal policy.

[DIAGRAM: Flowchart illustrating the policy lifecycle: Drafting, stakeholder review, executive approval, publication, and periodic re-certification]

### Why It Exists
Security policies exist to establish accountability and define the operational boundaries for all employees, contractors, and third parties. Without a formal framework, security becomes an ad-hoc, best-effort activity, leading to inconsistent controls, operational liability, and significant regulatory risk. The policy lifecycle ensures that these requirements are not static; it provides the necessary governance to adapt to new threats, business changes, and regulatory updates, ensuring that the organization's security posture remains resilient and compliant.

### Enterprise / Banking Reality
In Tier-1 banking, the security policy framework is a critical component of the bank's risk management and regulatory compliance strategy. Banks must align their policies with global multi-jurisdictional mandates, such as the Federal Reserve's SR 11-7 for model risk, SOX ITGC for financial reporting, and the DORA Digital Resilience framework. The reality is that policies must be highly structured, auditable, and integrated with the bank's broader risk management platform. Furthermore, the policy framework must be designed to support the bank's complex, decentralized structure, ensuring that policies are consistently applied across global business units while allowing for necessary local variations.

### Operational Considerations
Operationalizing the policy framework requires a disciplined approach to policy development and maintenance. Administrators must ensure that policies are clearly written, well-documented, and easily accessible to all stakeholders. Monitoring the effectiveness of the policy framework is critical; administrators must track policy compliance rates, identify gaps in policy coverage, and ensure that the policy lifecycle is being actively managed.
[CLI: PowerShell command to query the GRC platform API for the status of a policy review cycle]

### Common Misconceptions
!!! warning
    A common misconception is that security policies are "shelfware"—documents that are created to satisfy auditors and then ignored. In reality, a policy is only effective if it is operationalized, enforced, and integrated into the daily workflows of the organization. Another error is assuming that a "one-size-fits-all" policy framework is effective; policies must be tailored to the specific risks and operational realities of the business, ensuring that they are both enforceable and relevant.

### Interview Angle
1. Question: How do you architect a security policy framework that is both comprehensive and adaptable to the rapid pace of change in a Tier-1 banking environment?
   Answer: We implement a modular policy framework, where high-level policies define the "what" and "why," while detailed standards and procedures define the "how." This allows us to update technical standards and procedures frequently without needing to re-approve the high-level policies. We also integrate our policy framework with our GRC platform, which automates the review and approval process, ensuring that our policies remain current and compliant.
2. Question: How do you ensure that your security policies are effectively communicated and understood across a global, decentralized organization?
   Answer: We use a multi-channel communication strategy, including mandatory training, policy awareness campaigns, and integration with our internal knowledge management systems. We also implement a "policy-as-code" approach where possible, embedding policy requirements directly into our automated deployment pipelines, ensuring that security is "baked in" rather than "bolted on."
3. Question: What is your strategy for managing policy exceptions in a highly regulated banking environment?
   Answer: We implement a formal exception management process that requires a documented risk assessment, a compensating control plan, and approval from the appropriate risk owner. Exceptions are time-bound and must be periodically reviewed to ensure they remain valid and that the compensating controls are still effective. This ensures that we maintain accountability and that we are not accumulating unmanaged risk.

### Related Concepts
- Section 3.6: Security Monitoring & SIEM/SOC Operations
- Section 3.7: Data Protection & Information Security

## 2. Regulatory compliance mapping (SOC 2, ISO 27001, NIST, etc.)

### Technical Definition
Regulatory compliance mapping is the strategic process of aligning an organization's internal security controls with the requirements of external frameworks and mandates (e.g., SOC 2, ISO 27001, NIST CSF, PCI-DSS). This involves creating a "control cross-walk" or mapping matrix that demonstrates how specific technical and operational controls satisfy the requirements of multiple frameworks simultaneously, thereby reducing audit fatigue and ensuring consistent compliance posture across the enterprise.

### Underlying Mechanism
The mechanism relies on a GRC platform that maintains a centralized Control Framework (the "Common Control Set"). Each internal control is mapped to one or more regulatory requirements. When an auditor requests evidence for a specific framework (e.g., ISO 27001 Annex A controls), the GRC platform automatically pulls the relevant evidence—such as system configuration logs, access review reports, or vulnerability scan results—from the automated evidence pipelines. These pipelines interface with the SIEM and other monitoring systems, as referenced in Section 3.6, to provide continuous, real-time assurance. This mapping architecture allows the organization to "test once, comply many," where a single control validation satisfies multiple regulatory obligations, significantly streamlining the audit process.

[DIAGRAM: Matrix showing the mapping of internal security controls to multiple regulatory frameworks (SOC 2, ISO 27001, NIST CSF)]

### Why It Exists
Modern enterprises are subject to an increasing number of overlapping and often conflicting regulatory requirements. Managing compliance for each framework in isolation is operationally unsustainable, leading to redundant control testing, inconsistent security postures, and excessive audit costs. Compliance mapping exists to harmonize these requirements into a unified control framework, providing a single, authoritative view of the organization's compliance status and enabling efficient, scalable audit management.

### Enterprise / Banking Reality
In Tier-1 banking, compliance mapping is a critical operational function that must satisfy global, multi-jurisdictional mandates. Banks must demonstrate compliance with frameworks like DORA, PCI-DSS v4.0, and various national banking regulations simultaneously. The reality is that compliance mapping must be dynamic; as frameworks evolve or new regulations are introduced, the mapping matrix must be updated to reflect these changes. Furthermore, the mapping must be integrated with the bank's risk management platform, ensuring that compliance gaps are treated as business risks and prioritized accordingly.

### Operational Considerations
Operationalizing compliance mapping requires a disciplined approach to control management and evidence collection. Administrators must ensure that the mapping matrix is accurate, that controls are clearly defined, and that evidence collection is automated wherever possible. Monitoring the effectiveness of the compliance mapping is critical; administrators must track compliance status across all frameworks, identify gaps in control coverage, and ensure that the mapping remains aligned with the evolving regulatory landscape.
[CLI: PowerShell command to query the GRC platform for the compliance status of a specific control across multiple frameworks]

### Common Misconceptions
!!! warning
    A common misconception is that "compliance equals security." In reality, compliance is a baseline of minimum requirements; an organization can be fully compliant with a framework and still be vulnerable to sophisticated attacks. Another error is assuming that compliance mapping is a "set and forget" process; it requires continuous maintenance, as regulatory requirements change and the organization's infrastructure evolves.

### Interview Angle
1. Question: How do you manage the complexity of mapping internal controls to multiple, overlapping regulatory frameworks?
   Answer: We implement a "Common Control Set" (CCS) approach. We identify the most stringent requirements across all our frameworks and build our internal controls to meet those requirements. We then map these CCS controls to the specific requirements of each framework. This allows us to maintain a single, unified control set that satisfies all our regulatory obligations, significantly reducing the burden of audit preparation.
2. Question: How do you handle the challenge of "compliance drift," where controls are no longer effective or mapped correctly?
   Answer: We implement continuous control monitoring. We use automated evidence pipelines to validate the effectiveness of our controls in real-time, and we integrate this data into our GRC platform. If a control fails or drifts, the GRC platform automatically flags it as a compliance gap, triggering an investigation and remediation workflow. This ensures that our compliance posture is always accurate and up-to-date.
3. Question: What is your strategy for managing the impact of new or updated regulatory frameworks on your existing compliance mapping?
   Answer: We perform a "gap analysis" whenever a new framework is introduced or an existing one is updated. We map the new requirements to our existing CCS controls and identify any gaps. We then prioritize the remediation of these gaps based on business risk, ensuring that we maintain compliance without disrupting critical business operations.

### Related Concepts
- Section 3.8.1: Security policy framework & lifecycle
- Section 3.8.4: Audit preparation & evidence collection
