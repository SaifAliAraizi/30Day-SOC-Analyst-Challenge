# Day 18 — Command & Control (C2)  
**My DFIR / SOC Analyst — 30-Day Challenge**  
**Date:** 14 October, 2025
**Author:** Syed Saif Araizi

---

## Overview (single-section documentation)
This document covers Day 18 of the 30-day SOC analyst challenge: **Command & Control (C2)**. It explains what C2 is, why it's important to defenders, common C2 frameworks seen in the wild, high-level notes on one framework (Mythic), defensive and detection guidance, safe lab guidance, learning objectives for the day, and suggested exercises and artifacts to collect. **All content below is contained inside this single markdown section** for direct paste into a `.md` file for GitHub.

---

## Goal for Day 18
- Understand what Command & Control (C2) means from an adversary and defender perspective.  
- Learn the common C2 frameworks and high-level behaviours to watch for.  
- Practice detection and investigative techniques in a controlled, isolated lab environment (not on production).  
- Prepare artifacts and example detections to add to your SOC playbooks.

---

## What is Command & Control (C2)?
Command & Control (C2, aka CnC or CNC) is the infrastructure and set of techniques adversaries use to communicate with systems they have compromised inside a victim network. After initial execution of a malicious binary, many samples perform discovery (e.g., `ipconfig`, `whoami`, `net user`, DNS lookups), attempt persistence (services, scheduled tasks), and then open a C2 channel. That channel allows the operator to issue commands, transfer files, escalate privileges, move laterally, or exfiltrate data.

According to the MITRE ATT&CK model, C2 spans many techniques (HTTP[S] callbacks, DNS tunnelling, custom protocols, peer-to-peer, etc.). From a defender viewpoint, C2 is one of the key stages that enables an attacker to achieve their objectives (credential theft, lateral movement, ransomware, data exfiltration, etc.).

---

## Why is establishing C2 important for an attacker?
- **Control**: Enables remote command execution and interactivity with the compromised host.  
- **Flexibility**: Operators can instruct victims to perform follow-up actions (discovery, lateral movement, data staging).  
- **Persistence & Resilience**: C2 channels can be designed to survive reboots or evade simple blocking (fallback servers, domain fronting, multi-protocol).  
- **Exfiltration**: Provides a channel for staged or continuous data exfiltration.

---

## Common C2 Tools & Frameworks (high-level)
> The following are commonly referenced in blue-team resources. This is a *high-level list* for defensive familiarization only.

1. **Metasploit** (Rapid7) — Popular red-team/exploit framework used in labs and engagements. Good to know for detection patterns (payloads, beaconing behaviours).  
2. **Cobalt Strike** (Fortra) — Commercial adversary emulation product. Frequently seen in real-world intrusions; many detections and community resources exist to help identify it.  
3. **Sliver** (Bishop Fox) — Open-source adversary emulation framework supporting multiple transport types (HTTP/HTTPS, DNS, WireGuard, mTLS).  
4. **Mythic** — Modern open-source C2 framework with a web UI and plugin/agent model. Useful to study conceptually for how operators stage payloads and manage callbacks.

> **Note:** This list is provided to help SOC analysts recognize behaviours and telemetry associated with these frameworks. Do **not** deploy offensive tools on production networks.

---

## Focus: Mythic (conceptual)
- **What it is:** Mythic is an open-source C2 framework that manages payloads, agents, and C2 profiles via a web UI. It supports modular agents and C2 profiles which define how an agent communicates back to the server.  
- **Why study it (defensive reasons):** Understanding how Mythic structures callbacks, payload packaging, and C2 profiles helps analysts map observed telemetry (periodic beacons, unusual user-agent strings, custom DNS queries) back to likely C2 behaviours.  
- **Important defensive takeaways (non-actionable):**
  - Agents often beacon on a predictable interval — look for regular, low-volume callbacks.
  - Custom C2 profiles can mimic normal protocols (HTTP, HTTPS, DNS) — inspect payload sizes, frequencies, and unusual headers or encrypted blobs inside otherwise normal-looking sessions.
  - Recorded metadata (e.g., which payload triggered a callback) matters in a lab — defenders should map binaries to observed network behaviour.

---

## Safe Lab Guidance (mandatory)
- **ISOLATE**: Use a completely isolated lab network (host-only virtualization or an isolated VLAN) — never connect offensive tooling to production or the internet.  
- **Use snapshots**: Take VM snapshots before and after exercises so you can revert to a clean state.  
- **No internet exposure**: Restrict external connectivity; use an internal simulated C2 server if practicing callbacks.  
- **Legal & ethical**: Only practice on systems you own or have explicit permission to use. Do not reuse malicious binaries from the wild — prefer lab payloads that are intentionally benign or intentionally simulated.  
- **Logging & monitoring**: Ensure you have endpoint logs, packet captures, and network telemetry enabled in the lab so you can observe all artefacts.

---

## Learning objectives / Outcomes
By the end of Day 18 you should be able to:
1. Describe what C2 is and why attackers rely on it.  
2. Name at least four common C2 frameworks and the kinds of transports they commonly use (HTTP, HTTPS, DNS, mTLS, WireGuard, etc.).  
3. Explain high-level mitigation and detection strategies (network segmentation, egress filtering, beacon detection).  
4. Produce a short SOC playbook entry for detecting suspected C2 activity and list the telemetry sources required.  
5. Collect and present lab artefacts (PCAP, endpoint process lists, registry/scheduled task evidence) showing a simulated beacon and operator command.

---

## Suggested day exercises (defensive-focused)
> These exercises are *detection and analysis* activities — all steps must be performed in an isolated safe lab.

1. **Telemetry collection**  
   - Start endpoint logging (Sysmon, Windows Event Logs), enable host process monitoring, and capture network traffic (pcap).  
   - Execute a *benign* simulated beacon in the lab (for example, a scripted periodic HTTP GET to a local test server), then collect pcaps and process logs.

2. **Identification**  
   - Use the packet capture to identify periodic beaconing: look for regular intervals, consistent destination ports, or repeating patterns in headers/payload size.  
   - Correlate process execution logs with outbound connections to determine which process is generating the traffic.

3. **Investigation**  
   - Use query tools (Elastic / Splunk / Zeek logs) to search for hosts that have similar periodic connections.  
   - Build a timeline containing: initial execution, persistence actions (services / scheduled tasks), and first periodic callback.

4. **Detection rule creation (SOC playbook)**  
   - Create a detection rule (high-level logic only) for: `Hosts exhibiting repeated, periodic outbound connections to uncommon domains/ports AND spawned by unexpected processes`.  
   - Define required telemetry: DNS logs, proxy logs, IDS/IPS alerts, endpoint process logs, and NetFlow/PCAP.

5. **Documentation & reporting**  
   - Prepare a short incident summary for the simulated event, including indicators of compromise (IOCs), tactics/techniques observed (mapped to MITRE ATT&CK), and recommended containment steps.

---

## Example artifacts to collect (lab)
- PCAP of the callback traffic (full capture).  
- Sysmon or Windows Event logs (process creation events, network connection events).  
- `whoami` / `ipconfig` style outputs if they were part of the simulated commands (only in lab).  
- Registry or file evidence of persistence (service creation, scheduled tasks) — again: lab-only and cleaned up after.  
- Timeline CSV or markdown showing key timestamps and actions.

---

## Detection & Mitigation (high-level)
### Detection strategies
- **Beaconing detection**: Look for periodic outbound connections from endpoints to external hosts (consistent timing, small payloads).  
- **Anomalous destinations**: Monitor for DNS queries to newly registered domains, domains with low TTLs, or high entropy DNS answers.  
- **Unusual process-network mapping**: Processes that normally have no network activity generating outbound traffic.  
- **TLS inspection / metadata analysis**: Even without full decryption, analyze certificate anomalies, JA3/JA3S fingerprints, and connection metadata.  
- **Proxy & Egress controls**: Block or scrutinize direct outbound traffic from endpoints; funnel traffic through proxies with logging.

### Mitigations
- **Egress filtering**: Whitelist egress destinations where possible; block direct outbound connections.  
- **Network segmentation**: Limit lateral movement by restricting internal flows.  
- **Endpoint hardening**: Least privilege, patching, and application control to reduce execution of unknown binaries.  
- **Logging & retention**: Ensure endpoint and network logs are centrally collected and retained long enough for investigations.  
- **User awareness**: Train staff on phishing and suspicious attachments to reduce initial compromise vectors.

---

## SOC Playbook — Quick Template (copyable)
**Title:** Suspected C2 Beaconing / Callback Detection  
**Trigger:** Repeated outbound connections from a host with: (a) consistent interval OR (b) uncommon destination domain/port AND spawning process is anomalous.  
**Required telemetry:** Netflow/PCAP, proxy logs, DNS logs, endpoint process creation logs (Sysmon), firewall logs.  
**Investigation steps (high-level):**
1. Identify and isolate the host from the network (lab/testing: isolate VM).  
2. Capture memory and disk images (lab/test: snapshot).  
3. Search for persistence mechanisms (services, scheduled tasks, registry run keys).  
4. Extract network IOCs (domains, IPs, user-agent strings, certificate details).  
5. Map to MITRE ATT&CK techniques and escalate per incident severity.  
**Containment & remediation:** Block destination, kill malicious process, remove persistence, restore from clean snapshot or image.  
**Post-incident:** Update detection rules, share IOCs, and run a threat-hunt for similar activity.

---

## Mapping to MITRE ATT&CK (examples)
- **Initial execution & discovery:** T1059 (Command and Scripting Interpreter), T1082 (System Information Discovery)  
- **Persistence:** T1053 (Scheduled Task), T1543 (Create or Modify System Process)  
- **Command & Control:** T1071 (Application Layer Protocol), T1572 (Protocol Tunnelling), T1095 (Non-Application Layer Protocol)  
> Refer to the official MITRE ATT&CK pages for detailed technique IDs when writing playbooks or reports.

---

## Notes, ethics, and safety
- This documentation is for **defensive training and SOC analyst education only**.  
- Do **not** run offensive tooling against networks you do not own or manage.  
- Always follow your organization’s legal and policy guidance and use isolated lab environments for practice.

---

## Next steps (Day 19 preview)
- Day 19 will begin designing a Windows Server attack plan in the lab (defensive perspective): mapping expected telemetry for each stage, then simulate detection exercises. We will continue to expand SOC playbooks and detection rules.

---

## References & further reading (recommended)
- MITRE ATT&CK — Command and Control techniques (search MITRE ATT&CK for “Command and Control”).  
- Vendor detection guides (DFIR community write-ups on Cobalt Strike, Sliver, Mythic) — consult reputable DFIR blogs and vendor whitepapers.  
- Official docs for detection tooling you use (Suricata, Zeek, Sysmon, Elastic, Splunk).  

> When linking in the repo, add the specific URLs to the referenced MITRE ATT&CK pages and vendor resources.

---

## Repo assets to include for Day 18
- `day-18/` directory containing:
  - `day-18-notes.md` (this file or a short summary).  
  - `playbooks/detected-c2-playbook.md` (SOC playbook template copy).  
  - `lab-artifacts/` (pcap and logs collected during lab exercises — **ensure they are sanitized and safe**).  
  - `ioc-list.txt` (example IOCs generated during the lab; keep lab-only).  
  - `references.md` (links to MITRE and vendor resources).

---

## Closing
This day aims to make you comfortable identifying C2-like behaviours, designing a basic detection workflow, and documenting an analyst response. Keep your labs isolated, document everything, and map observations to MITRE ATT&CK to communicate findings clearly. Stay curious and practice defensively.

---