## 🧠 **Day 3: Setting Up Elasticsearch**

### 🎯 Objective

Deploy and configure **Elasticsearch** on a cloud-hosted Ubuntu VM (Vultr or Google Cloud Platform) as part of the SOC (Security Operations Center) lab environment.

---

### 🧩 **Steps Followed**

#### **1. Create Virtual Private Cloud (VPC)**

* Log in to **Vultr (vultr.com)** or GCP and navigate to **Network → VPC 2.0**.
* Create a new VPC network with:

  * **Region:** Toronto (or same region as your VM)
  * **IP Range:** `172.31.0.0/24`
  * **Name:** `mydef-soc-challenge`
* ✅ *Ensure all VMs (Elasticsearch, Kibana, SOC Analyst, etc.) are in the same region.*

---

#### **2. Deploy the Virtual Machine**

* Go to **Products → Deploy New Server**
* Select:

  * **Location:** Same as VPC (e.g., Toronto)
  * **OS Image:** Ubuntu 22.04 LTS
  * **CPU & RAM:** 4 vCPU / 16 GB RAM
  * **VPC:** Select your created VPC
  * **Hostname:** `elk-vm`
* Deploy the server and wait until the status shows **Running**.

---

#### **3. Connect via SSH**

From **PowerShell** or **Terminal**, connect using:

```bash
ssh -i ~/.ssh/id_rsa username@<public_ip>
```

Example:

```bash
ssh -i ~/.ssh/id_rsa araizii007@34.28.79.237
```

If needed, generate a new SSH key pair:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email"
```

---

#### **4. Update and Upgrade the System**

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

---

#### **5. Download and Install Elasticsearch**

Visit [Elastic Downloads](https://www.elastic.co/downloads/elasticsearch) → copy the `.deb` link.
Then run:

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.1.5-amd64.deb
sudo dpkg -i elasticsearch-9.1.5-amd64.deb
```

Upon installation, note the **auto-generated credentials**:

```
Elastic Superuser: elastic
Password: Ue8s5z8Ust-_jnphajSn
```

If lost, you can reset it:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

---

#### **6. Enable and Start Elasticsearch Service**

```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
```

Verify it’s running:

```bash
sudo systemctl status elasticsearch.service
```

✅ Output should show `active (running)`

---

#### **7. Configure Elasticsearch**

Switch to root:

```bash
sudo -i
```

Then navigate and edit configuration:

```bash
cd /etc/elasticsearch
nano elasticsearch.yml
```

Modify:

```yaml
network.host: <your_private_ip>  # e.g., 10.128.0.6 or 172.31.0.3
http.port: 9200
```

Save and restart:

```bash
sudo systemctl restart elasticsearch.service
```

---

#### **8. Apply Firewall Rules**

In Vultr or GCP:

* Go to your **VM Firewall Settings**
* Allow only:

  * SSH (22/tcp) — from your IP
  * HTTP (9200/tcp) — internal use
* Apply the rule to your `elk-vm`

---

### ✅ **Verification**

Run:

```bash
curl -X GET "localhost:9200/"
```

You should see a JSON response confirming Elasticsearch is active.

---

### 🧾 **Key Takeaways**

* Successfully deployed **Elasticsearch** using `.deb` package.
* Configured **systemd** for automatic service management.
* Applied firewall rules for restricted access.
* Prepared system for **Kibana integration (Day 4)**.

---

**Next Step:** Proceed to **Day 4** — Install and Configure **Kibana**.
