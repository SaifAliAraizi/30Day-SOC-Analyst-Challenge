# Day 2 — Understanding the ELK Stack

## Overview

On Day 2 of the **30-Day SOC Analyst Challenge**, the focus was on learning the **ELK Stack** — Elasticsearch, Logstash, and Kibana.
These tools form the backbone of modern **Security Information and Event Management (SIEM)** systems, allowing SOC analysts to **collect, store, search, and visualize logs** efficiently.

---

## What is the ELK Stack?

The **ELK Stack** consists of three open-source components developed by **Elastic**:

| Component             | Description                                                                           | Function                                             |
| --------------------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| **Elasticsearch (E)** | A NoSQL database designed for fast storage and searching of logs and structured data. | Stores and indexes large volumes of log data.        |
| **Logstash (L)**      | A data processing pipeline that collects, filters, and sends logs to Elasticsearch.   | Transforms and routes data from multiple sources.    |
| **Kibana (K)**        | A web-based interface for visualizing and analyzing data stored in Elasticsearch.     | Provides dashboards, queries, and alerting features. |

---

## 1. Elasticsearch

* Acts as the **central log database**.
* Stores logs such as:

  * Windows Event Logs
  * Syslogs
  * Firewall Logs
  * Application Logs
* Uses **EQL (Elasticsearch Query Language)** for querying data.
* Supports **RESTful APIs** and **JSON**, allowing integration with other systems and automation tools.
* Can be queried via the **Kibana web console** without needing API calls.

---

## 2. Logstash

* Works as the **data pipeline** between endpoints and Elasticsearch.
* **Collects**, **transforms**, and **forwards** telemetry (log data).
* Supports multiple input sources such as Beats or Elastic Agent.
* Example use case:

  * Collect Windows Event Logs using *Winlogbeat* or *Elastic Agent*.
  * Filter specific events (e.g., Event ID 4624 – Successful Login).
  * Forward the filtered logs into Elasticsearch for storage.

### Logstash Functions:

* **Filtering** – Selects only logs matching defined criteria.
* **Parsing** – Maps keywords or fields within a log to readable field names (e.g., source IP, destination port).
* **Customization** – Allows creation of custom parsers for unstructured logs.

---

## 3. Kibana

* The **visualization and analysis layer** for Elasticsearch data.
* Provides:

  * **Discover tab** – Query logs interactively.
  * **Kibana Lens** – Drag-and-drop dashboard builder.
  * **Machine Learning** – Detect anomalies automatically.
  * **Maps** – Visualize geolocation-based data.
  * **Alerting & Reporting** – Create automated alerts or generate reports.
* Acts as the **SOC Analyst’s interface** for monitoring and investigation.

---

## Beats and Elastic Agent

To send telemetry to Logstash or Elasticsearch, Elastic offers:

* **Beats** – Lightweight data shippers for specific data types:

  * *Filebeat* (logs)
  * *Metricbeat* (system metrics)
  * *Packetbeat* (network data)
  * *Winlogbeat* (Windows event logs)
  * *Auditbeat* (audit logs)
  * *Heartbeat* (uptime monitoring)
* **Elastic Agent** – A unified agent replacing multiple Beats, simplifying deployment and management.

In this 30-day challenge, the **Elastic Agent** will be used for all endpoints.

---

## Comparison with Splunk

| ELK Stack Component   | Equivalent in Splunk |
| --------------------- | -------------------- |
| Elasticsearch         | Indexer              |
| Logstash              | Heavy Forwarder      |
| Kibana                | Search Head / Web UI |
| Beats / Elastic Agent | Universal Forwarder  |

This makes transitioning between ELK and Splunk relatively easy for SOC analysts.

---

## Benefits of the ELK Stack

1. **Centralized Logging**
   Collects logs from multiple sources to support compliance and incident investigation.

2. **Flexible Data Ingestion**
   Supports various collection methods (Beats, Elastic Agent, APIs).

3. **Powerful Visualizations**
   Create dashboards to present metrics and KPIs — highly valuable for reporting to executives.

4. **Scalability**
   Can easily scale across distributed environments as the organization grows.

5. **Open Ecosystem & Integrations**
   ELK integrates with numerous platforms; many commercial SIEMs are built on top of it.

---

## Key Takeaways

* ELK provides the **core foundation for a SOC’s SIEM** solution.
* Understanding data flow — from endpoints → Logstash → Elasticsearch → Kibana — is essential.
* Elastic Agent simplifies endpoint data collection.
* ELK’s flexibility, scalability, and visualization capabilities make it a leading open-source SOC toolset.

---

## Next Steps

For Day 3, the focus will shift to **setting up Elasticsearch and Kibana** in a virtualized lab environment, and verifying connectivity for Elastic Agents.

---
