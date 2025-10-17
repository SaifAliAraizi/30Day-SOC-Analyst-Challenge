# Day 11 — Brute Force Attacks (SOC Analyst Challenge)

[![Day 11](https://img.shields.io/badge/Day-11-blue)](./)
[![Challenge](https://img.shields.io/badge/Challenge-SOC--Analyst-orange)](./)

---

## Overview

This single-file lesson covers **Brute Force attacks**: what they are, common variants and tools, practical defenses, detection ideas, a short lab, and useful resources. It's formatted for GitHub with a table of contents, code samples, checklists, and ready-to-copy detection snippets.

> **Audience:** SOC analysts, students, and blue-team practitioners who want a compact, practical reference.

## Table of Contents

1. [What is Brute Force?](#what-is-brute-force)
2. [Common Brute Force Types](#common-brute-force-types)
3. [How to Protect Yourself](#how-to-protect-yourself)
4. [Common Offensive Tools](#common-offensive-tools)
5. [Detection & Hunting (SIEM Examples)](#detection--hunting-siem-examples)
6. [Quick Hardening Checklist](#quick-hardening-checklist)
7. [Lab Exercise (safe, controlled)](#lab-exercise-safe-controlled)
8. [Resources & Further Reading](#resources--further-reading)
9. [Credits & Ethics](#credits--ethics)

---

## What is Brute Force?

A brute force attack is when an attacker systematically attempts many passwords (or credential pairs) to gain unauthorized access to an account or system. Think of trying every PIN on a luggage lock — same idea, but automated at scale against services like SSH, RDP, web login pages, or mail servers.

**Why it's common:** public-facing services, leaked credential dumps, and easy-to-use tools make brute forcing accessible to many attackers.

## Common Brute Force Types

- **Simple brute force:** Try every combination in sequence (e.g., `0000`, `0001`, ...). Usually automated.
- **Dictionary attack:** Use a wordlist of common passwords / patterns (often derived from credential dumps).
- **Credential stuffing:** Use username:password pairs from breaches and try them en masse across targets.

## How to Protect Yourself

**1) Use long passwords / passphrases**

- Aim for **15+ characters**. Prefer passphrases over short complex strings.
- Use a password manager to avoid reuse.

**2) Enable Multi-Factor Authentication (MFA)**

- Use authenticator apps or hardware tokens (preferred over SMS/email).

**3) Reduce attack surface & enforce controls**

- Avoid exposing SSH/RDP to the Internet. Use VPNs, bastion hosts, or jump boxes.
- Implement rate-limiting, IP allowlists, and geo-restrictions where possible.
- Consider progressive delays or lockouts after failed attempts (weigh DoS risk).

**4) Monitor & educate**

- Subscribe to breach notification services and rotate passwords if leaked.
- Be suspicious of unexpected login-related emails and phishing attempts.

## Common Offensive Tools (for lab/defensive analysis only)

- **Hydra** — network login cracker supporting many protocols.
- **Hashcat** — GPU-accelerated password recovery (offline hashes).
- **John the Ripper** — flexible password cracking framework.

> **Note:** Use these tools **only** on systems you own or have written permission to test.

## Detection & Hunting (SIEM Examples)

Below are example detection ideas and pseudo-queries. Adapt to your SIEM/log schema.

### Detection rules (examples)

- **Brute-force burst:** alert when an IP has more than **N failed logins** within **M minutes**.
- **Distinct usernames from one IP:** alert when a single IP attempts > X unique usernames in Y minutes.
- **Successful login after failures:** correlate success events with preceding failure burst.

### Example pseudo-SIEM queries

**Count distinct usernames by source IP (last 15 minutes)**

```
index=auth_logs sourcetype=ssh
| where _time >= relative_time(now(), "-15m")
| stats dc(user) as distinct_usernames, count as attempts by src_ip
| where distinct_usernames > 5
```

**Failed then successful login correlation (last 1 hour)**

```
index=auth_logs (action=failure OR action=success)
| bin _time span=1m
| stats count(eval(action=="failure")) as failures, count(eval(action=="success")) as successes by src_ip, user
| where failures >= 10 AND successes >= 1
```

**RDP example (Windows Event ID snippets)**

- Monitor Event IDs: `4625` (failed logon), `4624` (successful logon). Alert on `4624` following many `4625` from same IP.

## Quick Hardening Checklist

- [ ] Enforce password length (prefer passphrases) and discourage reuse.
- [ ] Enable MFA across all high-value accounts.
- [ ] Block or restrict RDP/SSH on public IPs; use bastion/VPN.
- [ ] Deploy rate-limiting / progressive delays for auth endpoints.
- [ ] Add alerting for credential stuffing patterns (many usernames, repeated IPs).
- [ ] Keep an inventory of internet-exposed services.
- [ ] Use fail2ban, CrowdSec, or equivalent to throttle brute force sources.

## Recommended Config Snippets

### Example: `sshd_config` minimal hardening

```ini
# /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication yes   # if you must allow passwords — prefer key-based auth
MaxAuthTries 3
AllowUsers soc_analyst
# Consider: Use AllowUsers/AllowGroups and listen on non-standard port behind firewall
```

### Example: `fail2ban` jail for SSH (simple)

```ini
# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 3600
```

## Lab Exercise (safe, controlled)

> **Do this only in a lab environment you control.**

1. Deploy two VMs: attacker (Kali) and target (Ubuntu with SSH).
2. On target: enable SSH and configure logging (syslog -> SIEM or local file).
3. On attacker: run a **controlled** Hydra test using a _small_ wordlist:

```
hydra -l testuser -P small-wordlist.txt ssh://TARGET_IP -t 4
```

4. Observe logs on the target. Implement a detection rule in your SIEM and test alerting.
5. Harden the target (apply `sshd_config` changes, enable fail2ban), then repeat the test to confirm mitigations.

## Resources & Further Reading

- OWASP Authentication Cheat Sheet
- Have I Been Pwned — breach notification
- Tool docs: Hydra, Hashcat, John the Ripper
- Fail2Ban / CrowdSec documentation
