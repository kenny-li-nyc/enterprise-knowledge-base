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

## 2. SAML/OIDC trusts

### Technical Definition
SAML (Security Assertion Markup Language) and OIDC (OpenID Connect) are the industry-standard protocols for federated identity. SAML is an XML-based framework for exchanging authentication and authorization data, while OIDC is a lightweight identity layer built on top of the OAuth 2.0 protocol, using JSON Web Tokens (JWTs).

### Underlying Mechanism
SAML relies on XML-based assertions signed by the Identity Provider (IdP) and consumed by the Service Provider (SP). OIDC uses an ID Token (a JWT) to convey identity information, often accompanied by an Access Token for API authorization. Both protocols rely on a trust relationship established via metadata exchange (e.g., SAML metadata XML or OIDC discovery endpoints) and cryptographic signing keys. The IdP signs the assertion or token, and the SP validates the signature against the IdP's public key to establish the user's identity.

### Why It Exists
These protocols exist to provide a standardized, interoperable way to federate identity across heterogeneous environments. They decouple the identity provider from the relying party, allowing for seamless SSO across web, mobile, and API-based applications, regardless of the underlying platform or technology stack.

### Enterprise / Banking Reality
In Tier-1 banking, SAML and OIDC are the primary protocols for modernizing identity. SAML is often used for legacy web applications, while OIDC is the standard for modern, cloud-native, and mobile applications. Banking security standards require that these protocols be implemented with strong cryptographic signing (e.g., SHA-256 or higher), secure token handling, and strict validation of the IdP's metadata to prevent "man-in-the-middle" or "token-replay" attacks.

### Operational Considerations
Managing these trusts requires careful lifecycle management of signing certificates and metadata.
[CLI: Get-AdfsRelyingPartyTrust -Name "BankingApp"]
[CLI: Update-FederationMetadata -Url "https://idp.bank.com/metadata"]
Monitoring for token validation errors, certificate expiration, and metadata updates is critical for maintaining SSO availability.

### Common Misconceptions
!!! warning
    A common misconception is that SAML and OIDC are interchangeable. This is false. While both provide identity federation, they are designed for different use cases. SAML is typically used for web-based SSO, while OIDC is optimized for modern, API-driven, and mobile applications. Choosing the wrong protocol can lead to significant integration challenges and security gaps.

### Interview Angle
1. **Scenario:** You are migrating a legacy application from SAML to OIDC. What are the key architectural considerations?
   *Model Answer:* I would focus on the differences in token handling, the shift from XML to JSON, and the need for a modern OAuth 2.0-compliant authorization server. I would also ensure that the application's security policies are correctly mapped to the new OIDC claims.
2. **Scenario:** How do you secure the trust relationship between an IdP and an SP?
   *Model Answer:* I would enforce strict validation of the IdP's metadata, use strong cryptographic signing (e.g., RSA-2048 or higher), and implement token binding or other mechanisms to prevent token replay attacks. I would also regularly rotate the signing certificates.
3. **Scenario:** Why is OIDC preferred for mobile applications?
   *Model Answer:* OIDC is lightweight, JSON-based, and designed for modern web and mobile architectures. It integrates seamlessly with OAuth 2.0, making it the ideal choice for securing APIs and mobile app backends, whereas SAML's XML-based structure is often too heavy and complex for these use cases.

### Related Concepts
*   [AD FS (Section 1.9)] - For the STS implementation.
*   [Authentication (Section 1.7)] - For the underlying authentication context.

## 3. Cross-forest vs cross-organization federation

### Technical Definition
Cross-forest federation refers to the connection of two distinct Active Directory forests, typically within the same organization (e.g., following a merger or acquisition) or between closely related business units, often utilizing Kerberos-based trust relationships or AD FS. Cross-organization federation, by contrast, involves connecting an organization's identity provider to an external entity, such as a SaaS provider, a partner organization, or a public cloud, using standardized claims-based protocols like SAML or OIDC.

### Underlying Mechanism
Cross-forest federation often leverages the trust architecture described in Section 1.2, where Kerberos referrals and SID filtering are used to validate identities across forest boundaries. When AD FS is used for cross-forest federation, it acts as a claims-transformation engine, mapping identities from one forest to another. Cross-organization federation, however, relies entirely on claims-based protocols (SAML/OIDC), where the external organization acts as a Relying Party or Claims Provider, and identity is exchanged via cryptographically signed tokens rather than direct directory-level trust.

### Why It Exists
Cross-forest federation exists to facilitate collaboration and resource sharing between distinct identity silos within an organization, often as a temporary measure during M&A or as a permanent solution for business unit isolation. Cross-organization federation exists to enable secure, scalable access to external applications and services without the need for complex, high-risk directory-level trusts, which are difficult to manage and secure at scale.

### Enterprise / Banking Reality
In Tier-1 banking, the distinction between these two models is critical for risk management. Cross-forest federation is treated as a high-risk activity, requiring strict SID filtering and rigorous security reviews to prevent lateral movement between forests. Cross-organization federation is the preferred model for external access, as it provides a clear, claims-based boundary that can be easily audited and revoked. Banking regulations (e.g., DORA, FFIEC) mandate that external federation be implemented with "least privilege" claims transformation, ensuring that only the minimum necessary identity information is shared with external partners.

### Operational Considerations
Managing these federation models requires distinct operational strategies.
[CLI: New-ADTrust -Name "PartnerForest.com" -Direction Outbound]
[CLI: New-AdfsClaimsProviderTrust -Name "PartnerIdP" -MetadataUrl "https://partner.com/metadata"]
Monitoring for trust-related errors, claims transformation failures, and unauthorized access attempts is essential for maintaining the security of both models.

### Common Misconceptions
!!! warning
    A common misconception is that cross-forest federation is "safer" than cross-organization federation because it uses "internal" trusts. This is false. Internal trusts are often more dangerous because they can allow for deeper, more persistent access if not properly filtered. Cross-organization federation, when implemented correctly with claims-based protocols, provides a much cleaner, more auditable security boundary.

### Interview Angle
1. **Scenario:** You are tasked with integrating a newly acquired company's forest into your existing environment. What is your federation strategy?
   *Model Answer:* I would first evaluate the security posture of the acquired forest. If it is not fully trusted, I would implement a cross-forest AD FS federation rather than a direct Kerberos trust. This allows me to control the claims transformation and isolate the two environments, minimizing the risk of lateral movement.
2. **Scenario:** Why is claims transformation critical in cross-organization federation?
   *Model Answer:* Claims transformation allows us to map external identities to internal roles or attributes, ensuring that we only grant the minimum necessary access. It also allows us to strip sensitive internal attributes from the tokens before they are sent to external partners, protecting our internal data.
3. **Scenario:** What are the risks of using direct Kerberos trusts for external partners?
   *Model Answer:* Direct Kerberos trusts are extremely risky for external partners because they grant the partner's domain controllers a level of trust that is difficult to contain. It can lead to SID history injection, privilege escalation, and other attacks that are hard to detect and remediate. Claims-based federation is always the preferred approach for external partners.

### Related Concepts
*   [Trust Architecture (Section 1.2)] - For understanding the cryptographic trust anchors and SID filtering.
*   [AD FS (Section 1.9)] - For the STS implementation.
