# 🧠 Day 10 — Sysmon & Microsoft Defender Log Ingestion into Elasticsearch

Welcome to **Day 10** of the **30-Day MyDFIR for SOC Analyst Challenge**!  
In this session, you will learn how to **successfully ingest Sysmon and Microsoft Defender event logs** from your **Windows Server** into your **Elasticsearch instance** using Elastic Agent and Kibana Integrations.

---

## 🎯 Objective

By the end of this lab, you will be able to:
- Add and configure custom **Windows Event Logs** integrations in Kibana.
- Ingest **Sysmon** and **Microsoft Defender** event logs into **Elasticsearch**.
- Troubleshoot connectivity and ingestion issues between Elastic Agent and Elasticsearch.

---

## ⚙️ Step 1: Access Elasticsearch and Add Integration

1. Log in to your **Elasticsearch** instance.  
2. On the homepage, click the **“Add Integrations”** button.
3. Search for **Sysmon** — you will notice it shows **Sysmon for Linux**, which isn’t what we want.
4. Instead, search for **Windows Event**.
5. Select **Custom Windows Event Logs** integration.

This integration allows you to **ingest any Windows Event Log channel** into Elasticsearch.  
Scroll down to view the **field mappings**, which define how each Windows event field is exported and displayed.

Example:
- `winlog.computer_name`: The name of the computer that generated the event.
- If any field lacks a description, you may need to research it manually.

---

## 🪟 Step 2: Add Sysmon Custom Event Log Integration

1. Click **“Add Custom Windows Event Logs”**.
2. Set the **Integration Name** to:  
   `araizii-win-sysmon`
3. Add a **Description**:  
   `Collect Sysmon logs`
4. For the **Channel Name**, open **Event Viewer** on your Windows Server:
   - Expand **Applications and Services Logs → Microsoft → Windows → Sysmon → Operational**.
   - Right-click **Operational → Properties** and copy the **Full Name** (this is your channel name).
5. Paste the channel name into the Kibana integration form.
6. Leave **Data Set** and other options as default.
7. Assign this integration to your existing host by selecting the **Agent Policy** (e.g., `araizii-windows-policy`).
8. Click **Save and Continue**, then **Save and Deploy Changes**.

Your Sysmon integration is now added to your policy.

---

## 🛡️ Step 3: Add Microsoft Defender Custom Event Log Integration

1. Click **“Add Custom Windows Event Logs”** again.
2. Name it:  
   `araizii-win-defender`
3. Description:  
   `Collect Defender logs`
4. For **Channel Name**, open **Event Viewer** again:
   - Navigate to **Applications and Services Logs → Microsoft → Windows → Windows Defender → Operational**.
   - Copy the **Full Name** from the **Properties** window.
5. Paste the channel name into the integration form.

---

## 🧩 Step 4: Configure Event IDs for Defender

We don’t want to ingest every log since some are informational (e.g., 1150 and 1151 — Health reports).  
Instead, we’ll focus on **critical event IDs**:

| Event ID | Description |
|-----------|--------------|
| **1116** | Malware or potentially unwanted software detected |
| **1117** | Action performed to protect against malware |
| **50001** | Real-time protection disabled |

These are the most relevant for detecting security incidents.

In the integration settings:
- Scroll down to **Advanced Options → Event ID**.
- Add the following values:  
  `1116,1117,50001`

*(If you wanted to exclude an ID, add a minus sign before it, e.g., `-1116`.)*

Assign it to the same **Agent Policy** (`araizii-windows-policy`), then **Save and Deploy Changes**.

---

## 🧠 Step 5: Generate Defender Events for Testing

To verify the ingestion:
1. On your Windows Server, open **Windows Security**.
2. Go to **Virus & Threat Protection → Manage Settings**.
3. Temporarily disable **Real-Time Protection**.
4. Refresh the **Defender Operational Log** — you should see **Event ID 50001** generated.
5. Re-enable Real-Time Protection.

This confirms Defender is producing the correct events.

---

## 🔍 Step 6: Verify Sysmon and Defender Logs in Kibana

1. In Kibana, click the **☰ (hamburger menu)** → **Discover**.
2. Search for: sysmon

Initially, you might not see any logs — this is fine.

### Troubleshooting:

If no data appears:
- Return to **Integrations → Custom Windows Event Logs**.
- Check the **Field Mapping** for `winlog.event_id`.
- Use this field to filter logs by event ID: winlog.event_id : 1

- If you still see nothing, proceed with connectivity troubleshooting.

---

## 🧰 Step 7: Troubleshoot Elastic Agent Communication

1. Go to **☰ → Fleet → Agents**.
2. Check the **Last Activity** and ensure your host shows as **Healthy**.
3. If **CPU** or **Memory** show as **N/A**, your agent may not be communicating with Elasticsearch.

### Fix:
- On your **Elasticsearch server firewall**, open port **9200 (TCP)**.
- Protocol: TCP  
- Port: 9200  
- Source: Anywhere (for testing) or restrict to your agent’s IP.
- Restart the Elastic Agent service: services.msc → Elastic Agent → Right-click → Restart


Wait a few minutes and refresh the Kibana Fleet dashboard.

Once the communication is restored, CPU and memory usage will display values, and logs will start appearing.

---

## 🧾 Step 8: Confirm Data Ingestion

After reconnecting:
- In **Discover**, search: winlog.event_id:1

You should now see:
- `event.provider: Microsoft-Windows-Sysmon`

- Next, search: winlog.event_id:50001

You should now see:
- `event.provider: Microsoft-Windows-Windows Defender`

This confirms successful ingestion of both **Sysmon** and **Defender** logs into Elasticsearch.

---

## ✅ Recap & Takeaways

- **Sysmon** and **Defender** logs are critical for endpoint visibility and detection.  
- You used **Custom Windows Event Log integration** to ingest them into **Elasticsearch**.  
- **Troubleshooting Tip:**  
If CPU or Memory in Fleet show as *N/A*, your agent cannot reach Elasticsearch — open port **9200** to fix it.
- You can now **analyze, visualize, and create detections** from Sysmon and Defender events.

---



