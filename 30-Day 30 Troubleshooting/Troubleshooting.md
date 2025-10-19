# 🧠 Day 30 – Final Configuration, Troubleshooting & Integration (Elastic Defend + osTicket)

## 🛠️ Overview

Day 30 focuses on troubleshooting, firewall configuration, Elastic Agent enrollment, osTicket database setup, and final network connectivity tests. This day ties together all the components of our Elastic Stack and osTicket integration to ensure seamless log forwarding and alert automation.

---

## 🔥 Connection Timeout Troubleshooting

When encountering **“connection timed out”** errors while accessing Kibana (port 5601), the main cause is often firewall restrictions.

### ✅ Fix Steps:

1. **Check Vulture Firewall Rules:**

   - Default configuration only allows **port 22 (SSH)**.
   - Update firewall to allow the full TCP port range:
     ```bash
     TCP 1-65535
     Source: My IP
     ```

2. **Allow Kibana Port in Ubuntu Firewall:**

   ```bash
   sudo ufw allow 5601
   ```

3. **Verify Services:**
   ```bash
   systemctl status kibana.service
   systemctl status elasticsearch.service
   ```

Both should be active and running.

---

## 🧩 Fleet Server Enrollment Troubleshooting

### ⚠️ Problem:

Elastic Agent failed to enroll due to port and connection issues.

### ✅ Fix Steps:

1. **Allow Fleet Server Port:**

   ```bash
   sudo ufw allow 8220
   ```

2. **Allow HTTPS Traffic (if needed):**

   ```bash
   sudo ufw allow 443
   ```

3. **Correct Fleet Server Port in Kibana UI:**

   - Navigate to: **Management → Fleet → Settings**
   - Change Fleet Server URL from `443` to `8220`
   - Save and redeploy

4. **Bypass x509 Certificate Error:**
   If you see `x509 certificate signed by unknown authority`, use:

   ```bash
   sudo ./elastic-agent install --insecure
   ```

5. **Verify Enrollment:**
   - Once the agent shows **online**, the configuration is correct.

---

## 🧱 ElasticSearch Connectivity Issue

If CPU/Memory shows **N/A**, it means the agent can’t connect to ElasticSearch.

### ✅ Fix:

Allow port 9200 for ElasticSearch:

```bash
sudo ufw allow 9200
```

You can also allow it in your Vulture firewall (port 9200, protocol TCP).

After a few minutes, Kibana will show CPU and memory metrics properly.

---

## 💾 osTicket MySQL Access & Authorization Fix

### ⚠️ Problem:

> “Cannot connect - invalid settings. The host is not allowed to connect to this MariaDB server.”

### ✅ Solution:

1. Edit PHPMyAdmin config:

   - Change bind address back to `127.0.0.1` in `config.inc.php`.

2. In PHPMyAdmin:

   - Go to **User Accounts → root@localhost → Edit Privileges → Login Information**
   - Change **Host name** from `localhost` to your **public IP**.
   - Update the password (e.g., `Winter2024!`).

3. Repeat the same for user **PMA**.

4. Save and restart Apache.

---

## 🗄️ Creating osTicket Database

If osTicket setup fails to create a DB:

1. Log into PHPMyAdmin.
2. Create a new DB manually:
   ```sql
   CREATE DATABASE mydfir_30day_db;
   ```
3. Grant privileges:

   ```sql
   GRANT ALL PRIVILEGES ON mydfir_30day_db.* TO 'root'@'your_public_ip';
   ```

4. Rerun osTicket installer:
   - Database name: `mydfir_30day_db`
   - Username: `root`
   - Password: `Winter2024!`
   - Hostname: `<public_IP>`

Once complete, you’ll see:

```
🎉 Congratulations! osTicket installation completed successfully!
```

---

## 🔔 Kibana Webhook Error – Timeout (60,000ms)

If you see:

> “Error calling webhook: request failed (timeout of 60,000ms)”

### ✅ Fix Steps:

1. SSH into ELK server:
   ```bash
   ssh root@<ELK_IP>
   ```
2. Check private IP:
   ```bash
   ip a
   ```
3. Ping osTicket server private IP.

   - If unreachable, configure internal networking properly.

4. Ensure osTicket VM has a **private IP (172.x.x.x)**.

   - If not, set static IP manually in **Network Adapter Settings → IPv4 → Use the following IP**.

5. Allow webhook traffic by adding private IP in **Kibana → Webhook Configuration**.

6. Run webhook test again.  
   ✅ Test Successful!

---

## 🧠 Summary

| Component         | Issue                 | Fix                                          |
| ----------------- | --------------------- | -------------------------------------------- |
| Kibana            | Connection timed out  | Allowed port 5601 via ufw + Vulture firewall |
| Fleet Server      | Enrollment failed     | Allowed port 8220 + adjusted Fleet settings  |
| ElasticSearch     | Metrics N/A           | Allowed port 9200                            |
| osTicket          | Invalid DB connection | Updated MySQL host to public IP              |
| Kibana → osTicket | Webhook timeout       | Used internal IP & fixed firewall rules      |

---
