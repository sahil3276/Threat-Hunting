# Ransomware Lifecycle — Threat Hunting Interview Notes

> **Not a fixed sequence:** Phases overlap; exfiltration often precedes encryption. Defense evasion can occur throughout. Some attacks steal data without encrypting it.

| Phase | Attacker Activity | What the Threat Hunter Looks For |
|---|---|---|
| **1. Reconnaissance** | Research employees, exposed services, and weak credentials, Phishing, Malware spreading. | Scanning patterns in firewall/WAF logs, phishing campaigns, exposed assets. **External OSINT may leave no internal logs.** |
| **2. Initial Access** | Enter through phishing, compromised accounts, or vulnerable services. | Unusual VPN/identity logins, suspicious email links/attachments, web-server exploit indicators. Correlate login and endpoint activity. |
| **3. Execution** | Run malicious code, scripts, or tools—not necessarily encryption yet. | Suspicious process trees, Office spawning shells, unexpected PowerShell/scripts. Use EDR, command-line logs, and supporting hashes. |
| **4. Persistence** | Maintain access across reboots or credential changes. | New scheduled tasks, services, startup entries, accounts, or unauthorized remote-access tools. |
| **5. Privilege Escalation** | Obtain higher privileges. | Unexpected administrator-group additions, unusual elevated processes, token manipulation, and exploit-related behavior. |
| **6. Lateral Movement** | Access additional systems using compromised accounts or remote tools. | Unusual host-to-host RDP/SMB/WinRM, remote service/task creation, and one account accessing many hosts. |
| **7. Exfiltration / Data Theft** | Collect, stage, and send sensitive data outside the environment. | Bulk file reads, archive creation, staging folders, unusual cloud uploads or outbound transfers. Correlate DLP, proxy, DNS, and endpoint logs. |
| **8. Encryption / Impact** | Encrypt files and disrupt recovery or operations. | Rapid file rewrites/renames, ransom-note creation, backup/shadow-copy deletion, and security or backup services being stopped. |
| **9. Ransom Demand** | Demand payment to restore files or prevent disclosure. | Ransom-note content and distribution, affected hosts, and linked processes. Preserve evidence and scope the incident. |
| **10. Cleanup / Evasion** | Delete evidence or disable monitoring. | Cleared logs, telemetry gaps, EDR tampering, and deleted tools. Reconstruct activity from centralized logs and other hosts. |

## Threat Hunter’s Role
**Form a hypothesis → Query telemetry → Correlate evidence → Identify affected users/hosts → Escalate to incident response → Improve detections.**

- **Hunt behavior, not just hashes:** Tools and filenames change.
- **Investigate precursors:** Credential theft, discovery, backup tampering, and lateral movement can reveal an attack before encryption.
- **Coordinate containment with IR:** Isolation, account revocation, and recovery follow the response process.
- **Keep roles clear:** Patching and segmentation reduce risk; hunting identifies evidence of compromise.

**Interview summary:** “I aim to detect ransomware before encryption by correlating identity, endpoint, and network activity, then scope the compromise and help incident responders disrupt it.”
