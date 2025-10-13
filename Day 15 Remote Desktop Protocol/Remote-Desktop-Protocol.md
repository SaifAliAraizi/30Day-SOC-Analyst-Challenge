# Day 15 — RDP (Remote Desktop Protocol)

> Part of the **30‑Day MyDFIR for SOC Analyst Challenge** — Day 15

---

## Overview

Remote Desktop Protocol (RDP) is one of the most commonly abused protocols in incident response and ransomware incidents. This session explains what RDP is, why teams use it, how attackers abuse it, how to find exposed RDP services on the Internet, and practical steps to protect your environment.

**Goal:** By the end of this day you should be able to: identify RDP services exposed to the Internet, understand common abuse patterns (including brute force and lateral movement), and implement practical mitigations.

---

## TL;DR

* RDP uses TCP and defaults to port **3389**.
* Exposed RDP is a frequent initial access vector in ransomware and other intrusions.
* Use Shodan and Censys to identify exposed hosts (`port:3389` or service filters).
* Do **not** connect to or brute-force external RDP services — this is illegal and unethical.
* Protect systems by turning RDP off when unused, enforcing MFA, restricting access (VPN/firewall), using strong passwords and PAM, and disabling default local admin accounts.

---

## What is RDP?

RDP (Remote Desktop Protocol) is Microsoft’s protocol for remote graphical connections between a terminal server and a client. It allows authorized users to access a remote desktop environment or server over a network.

**Key points:**

* Operates over TCP (default port **3389**).
* Commonly used for remote administration and troubleshooting.
* Very convenient but high-risk when exposed to the Internet.

---

## Why people use RDP

* Remote troubleshooting without physical access.
* Remote work access for employees and admins.
* Saves commute and speeds up operations.

These conveniences make RDP attractive — both for legitimate use and for attackers.

---

## How RDP gets abused

Common abuse patterns:

1. **Internet exposure + brute-force** — attackers scan for RDP on the Internet and try credential guessing or brute-force attacks.
2. **Credential reuse / credential stuffing** — leaked credentials from third‑party breaches are tested on RDP endpoints.
3. **Phishing & stolen credentials** — valid account credentials obtained via phishing are used to log in.
4. **Post‑access activity** — once inside, attackers may perform credential dumping, escalate privileges, and move laterally using RDP to other hosts.
5. **Ransomware / data theft** — after gaining privileged access, adversaries may exfiltrate data or deploy ransomware.

---

## How to find exposed RDP servers (high level)

Two useful public reconnaissance platforms are **Shodan** and **Censys**. They index Internet‑facing services and make it easy to search for exposed ports or service names.

**Example queries:**

* Shodan search: `port:3389` — returns hosts where the scanner observed TCP/3389 open.
* Censys search: `3389` then filter the `service_name` or `protocol` to refine results to RDP.

**Tips:**

* Use additional filters (geography, ASN, product string, hostnames) to narrow results.
* If you find your organization’s IPs exposed, open an internal ticket — do not attempt connecting or testing externally.

**Legal & ethical reminder:**

* Do not connect to or attempt access against servers you do not own or have explicit authorization to test. Scanning and brute forcing third‑party hosts is illegal in many jurisdictions and against acceptable use policies.

---

## Practical example (what to look for)

When you inspect a Shodan/Censys result look for:

* Open port `3389`.
* Service or product strings that indicate `RDP` or `Remote Desktop`.
* Screenshots (sometimes indexed) that reveal hostnames or logged‑in users.
* Associated services on the same host (e.g., 443, SSH, MySQL) which could indicate a multi‑service host.

If you identify one of your public IPs:

1. Confirm whether RDP should be enabled on that host.
2. If not required, disable the service or block 3389 on the external firewall.
3. If required, restrict access via firewall rules or place the host behind a VPN.
4. Review authentication logs for any suspicious successful logins (unusual locations, time, or user accounts).

---

## How to protect yourself — 5 recommendations

1. **Turn it off when not required.**

   * Developers often enable RDP and forget to disable it. Keep an inventory of systems and services.

2. **Use Multi‑Factor Authentication (MFA).**

   * Apply MFA for accounts that can authenticate via RDP (where supported by your stack or via gateway solutions).

3. **Restrict access (VPN / firewall / allowlists).**

   * Only allow RDP from known IP ranges or force connections through an internal VPN or bastion host.

4. **Use better password hygiene and PAM.**

   * Enforce long passwords (15+ chars) and use password managers. Prefer time‑bound or one‑time credentials delivered via a Privileged Access Management (PAM) solution.

5. **Disable default accounts / rename administrators.**

   * Remove or disable built‑in local accounts where possible and create named admin accounts instead.

**Note:** Passwords and account renaming do not fully stop credential stuffing. Combine controls (MFA, network restrictions, PAM) for defense in depth.

---

## Detection ideas & alerting (high level)

* **Brute force / multiple failed logins:** Alert when a single source has many failed RDP authentication attempts in a short time window.
* **Unusual successful logins:** Alert on successful RDP logins from geographies your org does not operate in.
* **New RDP host appearing externally:** Alert when a new public IP reports `port:3389` or RDP in your asset scans.
* **Credential dumping & lateral movement:** Watch for processes and tools commonly used for credential dumping after an RDP session.

*In the next session (video) we’ll build a concrete RDP Brute‑Force detection rule and test it with the Windows Server logs from our cloud lab.*

---

## Recommended immediate actions for defenders

1. Run an external inventory (Shodan/Censys) for your org’s public IP ranges and flag any `3389` results.
2. For flagged hosts: confirm necessity, then either disable RDP or restrict access to specific IP ranges or VPNs.
3. Enable MFA for remote access and consider a jump/bastion host for remote administration.
4. Rotate any credentials that may have been exposed and enforce complex passphrases and PAM where possible.
5. Add detection rules for RDP brute-force and suspicious successful RDP logins.

---

## Example incident playbook (short)

1. **Triage:** Identify the impacted host(s) and whether RDP was used for the initial access.
2. **Contain:** Isolate the host from the network (or block RDP at the firewall) and revoke any sessions.
3. **Preserve evidence:** Collect RDP logs, Windows Event Logs (Security, Sysmon if present), and relevant system artifacts.
4. **Eradicate & recover:** Remove persistence, reset credentials, deploy patches, and apply the protections from the recommendations above.
5. **Lessons learned:** Update asset inventory, review why RDP was exposed, and improve preventive controls.

---

## Additional notes & resources

* Do not attempt offensive testing against hosts you do not own — spin up your own RDP test server if you want to practice brute‑force detection safely.
* Use publicly available scanners for asset discovery (Shodan/Censys) only for reconnaissance and inventory — never for unauthorized access.
* In the next video we will implement a Kibana/Elastic alert for RDP brute‑force attempts and review the Windows Server logs from the cloud lab.

---

## Appendix — Quick checklist (copy/paste)

```
[ ] Inventory: List public IPs and check for port 3389
[ ] Disable RDP on hosts that don’t need it
[ ] Enforce VPN or bastion host for admin access
[ ] Enable MFA for remote access accounts
[ ] Implement PAM for privileged sessions
[ ] Add detection for: multiple failed logins, unusual successful logins
[ ] Rotate passwords / reset exposed credentials
[ ] Disable default local administrator accounts
```

---