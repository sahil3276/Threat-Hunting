- Threat Hunting is Proactive Technique, where Hunter uses `Hypothesis based approach` (assumed breach) and start hunting based on TTP used by APT groups and Malicious actors, 
- so below are the example what happens in each phase of attack, like what attacker might have done and what hunters can do
- 2. we `look for IOCs and IOAs` : Indicators of Compromise and Indicator of Attacks -- so recent attacks related IOCs (Packages, IP address, Hashes and more) we look into our company environment to decide whether we are compromised or not
- 3. We use tools look for anamlous activities like data encryption, file modification, data exfilteration, User account creation, Privilege escalation, Lolbins misuse etc


# Ransomware Attack Lifecycle — Interview Notes

> **Phases are not strictly sequential.** Attackers may repeat steps, exfiltrate before encryption, and evade defenses throughout. Some extortion attacks involve data theft without encryption.

## 1. Reconnaissance

**What attackers do**
- Research employees, technologies, exposed services, and leaked credentials.
- Identify potential entry points using public information and external scanning.
- After gaining access, perform **internal discovery** of hosts, accounts, shares, and security tools.

**What the threat hunter does**
- Review firewall/WAF logs for scanning, repeated probing, and unusual requests.
- Correlate targeted phishing campaigns with exposed employee information.
- Coordinate with exposure-management teams to identify publicly accessible assets.
- Distinguish external reconnaissance from internal discovery occurring after compromise.

**Key point:** Passive OSINT may leave **no evidence in internal logs**.

---

## 2. Initial Access

**What attackers do**
- Enter through phishing, stolen credentials, exposed remote-access services, or application vulnerabilities.
- May use an already-compromised account or legitimate remote-access software instead of obvious malware.

**What the threat hunter does**
- Investigate unusual VPN/identity logins: new devices, unfamiliar sources, unexpected hours, or suspicious MFA activity.
- Trace email delivery → link click/attachment → process execution.
- Check web-server activity followed by unexpected child processes or file creation.
- Identify the **first affected account and host**, not just the first alert.

**Evidence:** Email gateway, identity/VPN logs, web logs, EDR, Windows **4624/4625**.

---

## 3. Execution

**What attackers do**
- Run loaders, scripts, remote-access tools, or other malicious code.
- May use PowerShell, CMD, or legitimate system utilities.
- Execution at this stage does **not necessarily mean encryption has started**.

**What the threat hunter does**
- Analyze process trees, full command lines, executable paths, signatures, and account context.
- Look for unusual relationships such as Office launching a shell or a web-server process starting a command interpreter.
- Correlate execution with downloads, persistence changes, and outbound connections.
- Hunt behavior even when hashes or filenames are unknown.

**Evidence:** EDR, **4688**, **Sysmon 1**, PowerShell **4104** where enabled.

---

## 4. Persistence

**What attackers do**
- Maintain access through scheduled tasks, services, startup entries, accounts, or remote-management software.
- Persistence commonly belongs to the intrusion tooling—not necessarily the final ransomware executable.

**What the threat hunter does**
- Find newly created or modified tasks, services, autorun entries, and accounts.
- Inspect entries pointing to unusual or user-writable locations.
- Determine who created them and whether there was an approved deployment.
- Search other hosts for matching artifacts to establish scope.

**Evidence:** **4698/4702** for tasks, **7045** for service installation, **Sysmon 12–14** for registry changes.

**Key point:** Persistence maintains access; it does **not automatically make activity stealthy**.

---

## 5. Privilege Escalation

**What attackers do**
- Exploit weaknesses or misuse permissions to obtain administrator/SYSTEM access.
- May instead steal or obtain an already-privileged account.
- Higher privileges help reach protected data, disable controls, and affect more systems.

**What the threat hunter does**
- Investigate unexpected administrator-group additions and unusual privileged logons.
- Trace transitions from a low-privilege process/account to elevated execution.
- Correlate suspicious credential access with later privileged authentication.
- Check patch and configuration exposure as context—not as proof of exploitation.

**Evidence:** EDR token/process context, **4672** special privileges assigned, **4728/4732/4756** group membership changes.

**Key point:** A **4672 event alone is not proof of privilege escalation**; legitimate privileged logons generate it.

---

## 6. Lateral Movement

**What attackers do**
- Use compromised credentials and remote administration to access additional machines.
- Common paths include RDP, SMB, WinRM, WMI, remote services, and scheduled tasks.
- Target file servers, backup infrastructure, and identity systems to expand impact.

**What the threat hunter does**
- Identify new host-to-host connections and one account accessing many systems.
- Correlate source logons with destination-side process, service, or task creation.
- Investigate administrative activity originating from ordinary user workstations.
- Build a timeline of affected hosts and accounts; coordinate containment with IR.

**Evidence:** **4624** types 3/10, **4648**, **4769**, **5140/5145**, EDR network/process records.

**Key point:** Successful lateral movement may generate **few or no failed logons**.

---

## 7. Collection & Exfiltration

**What attackers do**
- Collect sensitive files, stage them in folders, and sometimes compress them.
- Upload data to external infrastructure or cloud storage.
- Use stolen information for double extortion, often **before encryption**.

**What the threat hunter does**
- Connect bulk file access → staging/archive creation → outbound transfer.
- Investigate unusual upload destinations, volumes, accounts, and times.
- Check DLP and cloud audit records to identify the data involved.
- Distinguish attempted transfer from confirmed exfiltration; document uncertainty.

**Evidence:** File access logs, EDR, DLP, proxy/firewall logs, cloud-storage audit logs.

**Key point:** Exfiltration can be **slow and distributed**, not just one large upload.

---

## 8. Encryption & Operational Impact

**What attackers do**
- Stop applications to unlock files and interfere with security or backup services.
- Delete recovery options, then encrypt accessible local files and network shares.
- May deploy the encryptor across many hosts through compromised administration systems.

**What the threat hunter does**
- Hunt for recovery tampering, service stops, and rapid file modifications.
- Identify the responsible process/account and affected shares or endpoints.
- Prioritize correlations such as **backup deletion followed by mass file changes**.
- Immediately escalate active impact for containment; preserve available evidence without delaying response.

**Evidence:** EDR file behavior, process creation, **7036** service changes, backup/VSS logs, file-server auditing.

**Key point:** Ransomware often implements encryption itself; it does not need `cipher.exe`.

---

## 9. Ransom Demand

**What attackers do**
- Create ransom notes, change desktop messages, or contact the organization directly.
- Threaten permanent data loss or public disclosure.
- May make claims about stolen data that require independent verification.

**What the threat hunter does**
- Preserve ransom notes, filenames, timestamps, and relevant indicators.
- Trace note creation back to processes and affected hosts.
- Continue investigating the original entry point and remaining access.
- Provide technical evidence to IR, legal, and leadership—not independently handle negotiation.

**Evidence:** Note files, EDR file-creation events, email messages, incident communications.

**Key point:** A ransom note is evidence of extortion—not proof of every claimed theft or a reliable attribution by itself.

---

## 10. Cleanup & Defense Evasion

**What attackers do**
- Clear logs, delete tools, disable monitoring, or alter security settings.
- May retain persistence for future access.
- These actions can occur **throughout the intrusion**, not only at the end.

**What the threat hunter does**
- Investigate log clearing, security configuration changes, and telemetry gaps.
- Reconstruct activity from centralized logs, network records, identity systems, and other hosts.
- Hunt for remaining persistence and compromised accounts.
- Turn findings into detection improvements and support validation after remediation.

**Evidence:** **1102** security log cleared, **104** event log cleared, EDR health/configuration records, centralized logs.

---


# Ransomware Threat Hunting — Technical Interview Notes

**Goal:** Identify attacker behavior **before mass encryption**, connect related events, and determine affected accounts and systems.

> **Core approach:** Suspicious behavior + process ancestry + user context + timeline. A tool name or isolated event alone does not prove ransomware.

## 1. Process Execution & Service Stopping

**What attackers do**
- Stop databases to release locked files before encryption.
- Stop backup or security services to disrupt recovery and detection.
- May use legitimate tools such as `taskkill.exe`, `net.exe`, or `sc.exe`.

**What the threat hunter does**
- Look for **many process terminations or service stops in a short period**.
- Identify which services were targeted—backup, database, and security services deserve priority.
- Examine the initiating process, full command line, account, and remote execution context.
- Compare activity with approved maintenance and software deployments.

**Useful telemetry**
- **Security 4688 / Sysmon 1:** Process creation.
- **System 7036:** Service state changes.
- EDR process termination and service telemetry.

---

## 2. File Manipulation & Encryption

**What attackers do**
- Rapidly read, modify, rename, or replace large numbers of files.
- Target local folders and accessible network shares.
- Create ransom notes across directories.
- Often encrypt directly through their own code—not necessarily through Windows utilities.

**What the threat hunter does**
- Find a process touching **many files across multiple directories or shares**.
- Look for new extensions, repeated note filenames, and sudden increases in file-write activity.
- Correlate file changes with the responsible process, user, and first-seen executable.
- Separate encryption from legitimate backup, compression, synchronization, or migration activity.

**Useful telemetry**
- EDR file-operation events.
- File-server auditing; **4663** where object auditing and appropriate SACLs are configured.
- **Sysmon 11** for file creation—not a complete record of every file modification.

> **Correction:** `cipher.exe` supports legitimate EFS encryption, while its `/w` operation overwrites free space. Its presence alone does not establish ransomware encryption. `wevtutil.exe` relates to event logs, not file encryption.

---

## 3. Network Traffic & Command-and-Control

**What attackers do**
- Communicate with infrastructure for instructions, payload delivery, or status reporting.
- May use HTTPS, cloud services, or legitimate remote-management tools.
- Some ransomware can encrypt **without contacting a server**, using embedded key material.

**What the threat hunter does**
- Investigate connections from unusual processes to rare or newly observed destinations.
- Look for repeated beacon-like connections, unexpected remote-access software, and suspicious DNS activity.
- Correlate destination reputation with process identity, execution time, and user behavior.
- Do not depend solely on known malicious IP lists.

**Useful telemetry**
- DNS, proxy, firewall, and EDR network logs.
- **Sysmon 3:** Network connections, if enabled.
- **Sysmon 22:** DNS queries, if configured.

> HTTPS hides content from ordinary network monitoring, but endpoint and connection metadata can still support investigation.

---

## 4. Persistence & Registry Changes

**What attackers do**
- Establish recurring access through scheduled tasks, services, startup entries, or remote-access tools.
- Persistence often belongs to an earlier intrusion tool rather than the final encryptor.
- Modify autorun locations such as:
```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

**What the threat hunter does**
- Identify new or modified tasks, services, and startup values.
- Investigate entries pointing to user-writable locations such as `AppData`, `Temp`, or unusual `ProgramData` subdirectories.
- Check the creator account, executable path, signature, command line, and creation time.
- Correlate creation with suspicious logons or file downloads.

**Useful telemetry**
- **Security 4698 / 4702:** Scheduled task creation/update.
- **System 7045 / Security 4697:** Service installation.
- **Sysmon 12–14:** Registry changes.
- **Security 4657:** Registry value modification, when configured.

---

## 5. Credential Dumping

**What attackers do**
- Attempt to extract credentials from LSASS memory or other credential stores.
- Use obtained credentials to access additional machines or privileged services.
- May use renamed tools or legitimate diagnostic utilities—not only recognizable malware.

**What the threat hunter does**
- Investigate unusual processes accessing LSASS, especially followed by dump-file creation.
- Examine access rights, signer, parent process, account, and whether the tool normally accesses LSASS.
- Correlate suspicious access with later authentication from new hosts.
- Avoid relying only on filenames such as “Mimikatz.”

**Useful telemetry**
- EDR credential-access detections.
- **Sysmon 10:** Process access, with appropriate configuration.
- Process creation and file creation events.

> Antivirus and diagnostic software may legitimately access LSASS. Context and correlation are essential.

---

## 6. Data Staging & Exfiltration

**What attackers do**
- Collect sensitive files into staging directories.
- Compress data into archives before transferring it externally.
- Use cloud storage, synchronization tools, or encrypted connections.
- May exfiltrate slowly rather than in one large transfer.

**What the threat hunter does**
- Correlate bulk file reads → archive creation → outbound transfer.
- Investigate unusual upload volume, destinations, service accounts, or access times.
- Check whether the uploader normally accesses that type or volume of data.
- Use DLP to assess sensitivity and cloud audit logs to identify upload operations.

**Useful telemetry**
- DLP, proxy, firewall, cloud-storage audit logs.
- EDR process/file/network events.
- File-share access auditing.

> A large transfer is a lead, not proof. Backups and approved migrations can look similar.

---

## 7. Lateral Movement

**What attackers do**
- Use compromised credentials and remote administration mechanisms to access other hosts.
- Common paths include RDP, SMB, WinRM, WMI, remote services, and scheduled tasks.
- Successful movement may occur without a burst of failed logons.

**What the threat hunter does**
- Look for **new source-to-destination relationships** and one account accessing many hosts.
- Investigate workstation-to-workstation administration or privileged activity from non-management devices.
- Correlate a remote logon with subsequent service/task creation or process execution on the destination.
- Compare with approved jump hosts, administrators, and deployment systems.

**Useful telemetry**
- **4624:** Successful logon; type **3** commonly network, type **10** remote interactive.
- **4625:** Failed logon.
- **4648:** Explicit credential use.
- **4769:** Kerberos service ticket requests on domain controllers.
- **5140 / 5145:** Share access events, when enabled.

> Logon type 3 is common legitimate activity; it does not identify a specific attack method by itself.

---

## 8. Recovery Inhibition & Shadow-Copy Deletion

**What attackers do**
- Delete shadow copies or backups.
- Change recovery settings or stop backup services.
- Attempt to remove recovery options before encryption.

**What the threat hunter does**
- Investigate destructive use of utilities such as `vssadmin`, `wbadmin`, or `diskshadow`, as well as API-based activity.
- Correlate command lines with actual backup deletion or recovery-configuration changes.
- Check the account, originating host, targeted systems, and maintenance schedule.
- Treat recovery tampering followed by mass file changes as **high priority**.

**Useful telemetry**
- Process creation events.
- Backup platform audit logs.
- VSS, service, and endpoint telemetry.

> Do not wait for repeated occurrences: a single unauthorized recovery-deletion action can be significant.

---

## 9. Defense Evasion & Log Clearing

**What attackers do**
- Disable security tools, change exclusions, or interfere with logging.
- Clear event logs or delete intrusion tools.
- Obfuscate scripts or rename executables.

**What the threat hunter does**
- Look for security configuration changes and sudden telemetry gaps.
- Correlate log clearing with the account and process responsible.
- Reconstruct activity from centralized logs, identity records, network devices, and neighboring hosts.
- Determine whether gaps result from tampering or an ordinary agent/network failure.

**Useful telemetry**
- **Security 1102:** Security audit log cleared.
- **EventLog 104:** Event log cleared.
- Defender/EDR configuration and health events.
- **PowerShell 4104:** Script-block logging, when enabled.

---

## Key MITRE ATT&CK Mapping

| Behavior | Technique |
|---|---|
| PowerShell / CMD execution | **T1059.001 / T1059.003** |
| Scheduled task persistence | **T1053.005** |
| Registry Run Keys / Startup Folder | **T1547.001** |
| LSASS memory credential dumping | **T1003.001** |
| Remote services for lateral movement | **T1021** |
| Archive collected data | **T1560** |
| Service stopping | **T1489** |
| Inhibit system recovery | **T1490** |
| Data encrypted for impact | **T1486** |
| Disable or modify security tools | **T1562.001** |
| Clear Windows event logs | **T1070.001** |

### Interview Example: Connecting the Evidence

```text
Unexpected remote logon
        ↓
New service or scheduled task
        ↓
Suspicious credential access
        ↓
Same account reaches multiple hosts
        ↓
Backup tampering + rapid file changes
```

**How to explain your response:**  
“I validate the process and account context, correlate activity across hosts, and scope the compromise. If evidence indicates an active attack, I immediately coordinate containment with incident response, preserve evidence, and turn the confirmed behavior into improved detections.”

> **Logging caveat:** Event availability depends on audit policy, Sysmon configuration, and EDR coverage. Missing events do not prove the activity never occurred.
