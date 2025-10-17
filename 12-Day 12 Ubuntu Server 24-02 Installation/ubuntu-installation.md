# Day 12 — Ubuntu Server Setup & Live Log Review (SOC Analyst Challenge)

[![Day 12](https://img.shields.io/badge/Day-12-blue)](./)
[![Challenge](https://img.shields.io/badge/Challenge-SOC--Analyst-orange)](./)

---

## Overview

This single-file lesson (ready to paste into a `.md` file) walks you step-by-step through deploying a lightweight Ubuntu SSH server in the cloud, connecting to it, exploring authentication logs in real time, and using simple command-line parsing to extract useful signals such as failed logins and attacker IPs. All text and examples are contained inside this one markdown section so it displays nicely on GitHub.

> **Goal:** Deploy an SSH VM, connect to it, view `/var/log/auth.log` activity, and extract failed-login IPs using `grep`/`cut`.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Deploying the VM (example: Vultr)](#deploying-the-vm-example-vultr)
3. [Connect to your server (SSH)](#connect-to-your-server-ssh)
4. [Update & upgrade the server](#update--upgrade-the-server)
5. [Authentication logs location and basics](#authentication-logs-location-and-basics)
6. [Command-line log filtering examples](#command-line-log-filtering-examples)
7. [Field extraction with `cut` — how it works](#field-extraction-with-cut----how-it-works)
8. [Keep it running: generate logs & monitor in real time](#keep-it-running-generate-logs--monitor-in-real-time)
9. [Conclusion & next steps]
10. [Safety & ethics]

---

## Prerequisites

- A cloud provider account (Vultr, AWS, GCP, DigitalOcean, etc.). The demo uses **Vultr** as an example.
- Basic familiarity with SSH and a terminal (PowerShell, macOS Terminal, or Linux shell).
- Optional: small wordlist and a lab attacker VM if you want to test detection locally (only in a lab you control).

## Deploying the VM (example: Vultr)

1. Sign in to `vultr.com` and click **Deploy** → **Deploy New Server**.
2. Choose a lightweight compute plan (e.g., 1 CPU / 1 GB RAM) — we don't need a beefy VM.
3. Choose an image: **Ubuntu 24.04** (or latest LTS/available version).
4. Choose region (demo used Toronto).
5. Set server hostname. If participating in any giveaway, follow naming guidelines (e.g., `araizii-linux-<your_handle>`).
6. Deploy and wait until the server status shows **Running**.

## Connect to your server (SSH)

From PowerShell / terminal run:

```bash
ssh root@<SERVER_IP>
# Accept the host key by typing 'yes' when prompted, then paste the root password from the cloud console
```

If using an SSH key pair, upload/add the public key from your account during provisioning and login with:

```bash
ssh -i ~/.ssh/id_rsa root@<SERVER_IP>
```

## Update & upgrade the server

Once connected, update package lists and upgrade installed packages:

```bash
apt-get update && apt-get upgrade -y
clear
```

## Authentication logs location and basics

- Authentication-related logs are stored in: `/var/log/auth.log` (on Debian/Ubuntu systems).
- Other useful logs in `/var/log/` include `syslog`, `kern.log`, and service-specific logs.

Quick check:

```bash
cd /var/log
ls -la
# view raw auth log
cat auth.log | less
```

When the machine is new, you may see little to no activity. Leaving it exposed for some hours or a day will typically generate many automated failed SSH attempts (credential scanning and brute force activity).

## Command-line log filtering examples

Filter for failed SSH authentication attempts (case-insensitive):

```bash
grep -i "failed" /var/log/auth.log
```

Filter for attempts targeting a specific user (e.g., `root`):

```bash
grep -i "failed" /var/log/auth.log | grep -i "root"
```

Show lines that mention `invalid user` (attackers trying random usernames):

```bash
grep -i "invalid user" /var/log/auth.log
```

Extract only the IP addresses from `auth.log` failed entries (see explanation below):

```bash
# crude approach: find lines with 'Failed' then print the 9th field
grep -i "failed" /var/log/auth.log | grep -i "root" | cut -d ' ' -f9
```

> Note: field positions can vary depending on syslog formatting. Always inspect raw lines to confirm which field contains the IP address before automating extraction.

## Field extraction with `cut` — how it works

`cut -d ' ' -fN` splits each line by the delimiter (here a single space) and selects field `N`.

Example log line breakdown (illustrative — your formatting may vary):

```
Oct 12 03:14:21 mydfir-linux-root sshd[1234]: Failed password for root from 203.0.113.45 port 54722 ssh2
```

Split by spaces:

- Field 1: `Oct`
- Field 2: `12`
- Field 3: `03:14:21`
- Field 4: `mydfir-linux-root`
- Field 5: `sshd[1234]:`
- Field 6: `Failed`
- Field 7: `password`
- Field 8: `for`
- Field 9: `root`
- Field 10: `from`
- Field 11: `203.0.113.45`
- Field 12: `port`
- Field 13: `54722`

So to extract the **IP** (here field 11):

```bash
grep -i "failed" /var/log/auth.log | grep -i "root" | cut -d ' ' -f11
```

Because log formats vary (some use multiple spaces or tabs), you may prefer `awk` which is more flexible:

```bash
grep -i "failed" /var/log/auth.log | grep -i "root" | awk '{ for(i=1;i<=NF;i++) if ($i == "from") print $(i+1) }'
```

This `awk` script searches for the token `from` and prints the next field — a robust way to extract the IP address.

## Keep it running: generate logs & monitor in real time

To watch the auth log in real time:

```bash
tail -f /var/log/auth.log
```

To filter live for failed attempts:

```bash
tail -f /var/log/auth.log | grep -i --line-buffered "failed"
```

To count distinct attacker IPs in the file:

```bash
grep -i "failed" /var/log/auth.log | awk '{ for(i=1;i<=NF;i++) if ($i == "from") print $(i+1) }' | sort | uniq -c | sort -nr
```

This pipeline:

- extracts IPs after the token `from`,
- sorts them,
- counts unique occurrences,
- and sorts the counts in descending order.

## Conclusion & next steps

You have now:

- Deployed a minimal SSH server in the cloud,
- Connected to it via SSH,
- Located and inspected authentication logs at `/var/log/auth.log`,
- Used `grep`, `cut`, and `awk` to filter and extract failed login events and attacker IPs,
- Learned how to monitor logs in real time with `tail -f`.
