# 🧠 Day 6: Elastic Agent & Fleet Server

### 🎯 Objective
Learn about **Elastic Agent** and **Fleet Server**, and understand how they help in centralized management of endpoints for log forwarding and monitoring.

---

## 🧩 Overview

Imagine installing an agent on 100 Windows machines and then realizing that you forgot to configure it to forward **PowerShell logs** to your **Elasticsearch** instance.  

You have a few options:

1. **Manually update all endpoints** – Painful and time-consuming for large environments.  
2. **Use Group Policy** – Update all endpoints at once with the latest configuration.  
3. **Fleet Server** – Connect agents to a centralized server for easy policy and integration management.

---

## 📝 Elastic Agent

**Elastic Agent** provides a unified approach to collect:

- Logs
- Metrics
- Security events
- And other types of telemetry

Key points:

- Works via **policies** to determine what data to collect and where to send it (Elasticsearch or Logstash).  
- Supports **Integrations** and **protections** that can be added or updated dynamically.  
- Two installation methods:
  1. **Standalone**
  2. **Fleet-managed** (used in this challenge)

### Beats vs. Elastic Agent

| Feature                | Beats                     | Elastic Agent                    |
|------------------------|---------------------------|---------------------------------|
| Installation            | Multiple Beats per host   | Single agent handles all logs   |
| Data Collection         | Specific type only        | Multiple data types             |
| Management              | Manual or scripts         | Centralized via Fleet Server    |
| Target                  | Elasticsearch/Logstash   | Elasticsearch/Logstash          |

💡 **Tip:** For most cases, **Elastic Agent** is sufficient.

---

## 📝 Fleet Server

**Fleet Server** is a component that connects Elastic Agents to a **centralized fleet**, enabling:

- Centralized **policy management**  
- Easy addition of **new integrations**  
- Switching data destinations between **Elasticsearch** and **Logstash**  
- **Version upgrades** and agent unenrollment  

Without a Fleet Server, updating agent policies requires manual intervention or other cumbersome methods.

---

## ✅ Summary

- **Elastic Agent**: Unified agent for logs, metrics, and security telemetry.  
- **Fleet Server**: Centralized management of Elastic Agents, simplifying configuration and monitoring across many endpoints.  
- **Use Case**: Makes large-scale deployments (like 100+ machines) manageable.  
