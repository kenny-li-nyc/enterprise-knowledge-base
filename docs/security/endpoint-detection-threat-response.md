# 3.4 Endpoint Detection & Threat Response

## 1. EDR/XDR Architecture & Telemetry

### Technical Definition
Endpoint Detection and Response (EDR) and Extended Detection and Response (XDR) represent the evolution of endpoint security from static, signature-based prevention to dynamic, behavioral-based detection and automated response. EDR focuses on the continuous monitoring and recording of endpoint activities—such as process execution, network connections, file modifications, and registry changes—to provide deep visibility into potential security incidents. XDR extends this paradigm by ingesting and correlating telemetry not just from endpoints, but across the entire security stack, including network, cloud, identity, and email vectors, to provide a unified, context-aware view of the threat landscape.

### Underlying Mechanism
The architecture relies on lightweight kernel-mode filter drivers and user-mode agents that hook into critical operating system subsystems to intercept and log events in real-time. On Windows, this involves leveraging Event Tracing for Windows (ETW), specifically providers like Microsoft-Windows-Kernel-Process and Microsoft-Windows-Kernel-Network, to capture process creation, thread injection, and socket activity. These agents perform local behavioral analysis, comparing observed activity against known malicious patterns or anomalies, and stream enriched telemetry to a centralized data lake. The telemetry pipeline is designed for high-throughput ingestion, where complex event processing engines correlate disparate signals—such as a PowerShell script spawning a network connection to an external IP followed by a credential dumping attempt—to reconstruct the attack chain. This runtime surveillance operates independently of static code integrity controls, which, as referenced in Section 2.6, handle the initial validation of binary execution, whereas EDR/XDR focuses on the post-execution behavior of those binaries.

[DIAGRAM: High-level architecture showing endpoint agents streaming telemetry to a centralized XDR data lake for correlation and analysis]

### Why It Exists
Traditional antivirus solutions were architected to detect known malware signatures, leaving a massive detection gap for fileless attacks, living-off-the-land (LotL) techniques, and zero-day exploits that do not rely on static files. As attackers shifted toward sophisticated, multi-stage campaigns, the need for continuous visibility and retrospective analysis became paramount. EDR/XDR exists to bridge this gap by providing the granular telemetry required to detect subtle indicators of compromise (IoCs) and indicators of attack (IoAs) that would otherwise remain invisible, enabling security teams to reconstruct the timeline of an intrusion and respond before significant damage occurs.

### Enterprise / Banking Reality
In Tier-1 banking environments, EDR/XDR is the cornerstone of the Security Operations Center (SOC) visibility strategy. The architecture must be designed for massive scale, supporting tens of thousands of endpoints while maintaining strict performance SLAs to