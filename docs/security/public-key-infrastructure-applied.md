# 3.3 Public Key Infrastructure (Applied)

## 1. CA Hierarchy Trust Consumption (Root/Intermediate Validation)

### Technical Definition
CA hierarchy trust consumption refers to the process by which an endpoint or application validates the authenticity of a digital certificate by traversing a chain of trust back to a trusted root anchor. This hierarchy typically consists of a Root Certificate Authority (CA), which is self-signed and kept offline, and one or more Intermediate CAs, which are used to issue end-entity certificates. Trust consumption is the mechanism of verifying that each certificate in the chain is cryptographically signed by the issuer above it, ultimately terminating at a root certificate that is explicitly trusted by the client's local trust store.

### Underlying Mechanism
The mechanism relies on X.509 path validation algorithms. When a client receives a certificate, it parses the certificate's issuer field and attempts to locate the corresponding issuer certificate in its local trust store. This process repeats recursively until a self-signed certificate is reached. The client verifies the digital signature of each certificate using the public key of the issuer, checks the validity period, and ensures that the certificate's usage constraints (e.g., Basic Constraints, Key Usage) are satisfied. As noted in Section 2.2, the endpoint's trust store is pre-populated with root anchors, providing the necessary context for this validation process. Furthermore, as discussed in Section 2.7, this validation is the foundational step for establishing secure transport layers, ensuring that the remote gateway's identity is verified before any data encapsulation occurs.

[DIAGRAM: Flowchart illustrating the recursive path validation process from end-entity certificate to root anchor]

### Why It Exists
This hierarchy exists to provide a scalable and manageable trust model. A single root CA cannot issue all certificates for an enterprise without creating a single point of failure and a massive security risk. By delegating issuance to intermediate CAs, organizations can isolate the root CA, manage different certificate policies