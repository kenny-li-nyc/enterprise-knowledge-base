# Federation

## 1. AD FS

### Technical Definition
Active Directory Federation Services (AD FS) is a server role in Windows Server that provides a standards-based identity federation and web single sign-on (SSO) capability. It acts as a Security Token Service (STS), allowing organizations to extend their internal Active Directory identities to external applications, cloud services, and partner organizations by issuing security tokens (SAML, WS-Federation, OAuth) rather than relying on direct credential sharing.

### Underlying Mechanism
AD FS operates by establishing a trust relationship between a Claims Provider (the identity source, typically AD DS) and a Relying Party (the application consuming the identity). When a user attempts to access a federated application, they are redirected to the AD FS server. AD FS authenticates the user against Active Directory—often leveraging the local authentication context established via Kerberos as described in Section 1.7—and then issues a cryptographically signed security token containing claims about the user. This token is then presented to the Relying Party, which validates the signature against the AD FS token-signing certificate to establish the user's identity and authorization level.

### Why It Exists
AD FS exists to bridge the gap between internal, directory-based authentication (Kerberos/NTLM) and the requirements of modern, web-based applications that rely on claims-based identity protocols. It provides a centralized, secure mechanism for managing external access without requiring the creation of duplicate accounts in external systems or the exposure of internal credentials to third-party applications.

### Enterprise / Banking Reality
In Tier-1 banking, AD FS is a critical component of the identity perimeter, often deployed in a high-availability configuration behind Web Application Proxies (WAP) to isolate the internal STS from the public internet. Banking compliance frameworks (e.g., FFIEC, DORA) require that token-signing certificates be managed with the highest level of security, often utilizing Hardware Security Modules (HSMs) to protect the private keys. Furthermore, AD FS is a primary target for attackers; securing the AD FS farm, monitoring for token-signing certificate theft, and enforcing strict MFA requirements are non-negotiable operational mandates.

### Operational Considerations
AD FS requires rigorous lifecycle management, particularly for token-signing and token-decrypting certificates.
[CLI: Get-AdfsCertificate -CertificateType Token-Signing]
[CLI: Update-AdfsCertificate -CertificateType Token-Signing]
Monitoring the health of the AD FS farm, including the WAP servers and the underlying SQL database (if used), is essential for maintaining SSO availability.

### Common Misconceptions
!!! warning
    A common misconception is that AD FS is a "cloud" service. This is false. AD FS is an on-premises server role. While it is frequently used to federate with cloud providers like Microsoft Entra ID (formerly Azure AD), the AD FS infrastructure itself must be managed, patched, and secured by the organization.

### Interview Angle
1. **Scenario:** You are designing an AD FS architecture for a bank. How do you ensure the security of the token-signing certificates?
   *Model Answer:* I would store the token-signing certificate private keys in a FIPS 140-2 Level 3 compliant Hardware Security Module (HSM). This ensures that even if the AD FS server is compromised, the private keys cannot be exported, preventing attackers from forging security tokens.
2. **Scenario:** What is the role of the Web Application Proxy (WAP) in an AD FS deployment?
   *Model Answer:* The WAP acts as a reverse proxy, terminating external connections and pre-authenticating requests before they reach the internal AD FS server. It provides a critical layer of isolation, ensuring that the internal STS is not directly exposed to the public internet.
3. **Scenario:** How do you handle the transition from AD FS to a cloud-native identity provider?
   *Model Answer:* I would implement a phased migration, starting with non-critical applications and using a hybrid identity model (e.g., Entra Connect) to synchronize identities. I would then gradually move the federation trust from AD FS to the cloud provider, ensuring that all security policies and MFA requirements are maintained throughout the transition.

### Related Concepts
*   [Trust Architecture (Section 1.2)] - For understanding the cryptographic trust anchors and forest trust boundaries.
*   [Authentication (Section 1.7)] - For understanding the local authentication context and Kerberos/NTLM handshakes.
