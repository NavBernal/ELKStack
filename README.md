I built a fully functional, on-prem SOC lab from scratch using the ELK Stack, Sysmon, and Mythic C2, configuring endpoints to forward logs, creating detection rules and dashboards, and simulating real-world attacks like brute force activity. By generating and investigating my own attack data, I gained hands-on experience with log ingestion, threat detection, alert tuning, and incident response workflows. This project strengthened both my technical understanding of security operations and my ability to document and present complex findings in a clear format.
## Creating a Logical Diagram
Using draw.io, I created the following diagram to clearly and effectively illustrate the lab environment:

![](attachments/Pasted%20image%2020260418222119.png)

## ELK Stack Introduction

The **ELK Stack** is a powerful combination of tools used for centralized logging, analysis, and visualization of security data.
### **Elasticsearch (E): Data Storage & Search**
- A high-performance database for storing logs (Windows events, syslogs, firewall logs, etc.)
- Enables fast, flexible searching across large datasets
- Uses **ESQL** for querying data
- Supports RESTful APIs and JSON, making it easy to integrate with other tools and automate data retrieval
### **Logstash (L): Data Collection & Processing**
- Acts as a pipeline to collect and process telemetry from multiple sources
- Normalizes and forwards data into Elasticsearch
- Supports log collection via:
    - **Beats** (lightweight shippers):
        - Filebeat: Logs
        - Metricbeat: System metrics
        - Packetbeat: Network data
        - Winlogbeat: Windows events
        - Auditbeat: Audit data
        - Heartbeat: Uptime monitoring
    - **Elastic Agent**: A unified agent that collects multiple data types from a single host
- Allows filtering and transformation of logs for highly customized ingestion
### **Kibana (K): Visualization & Analysis**
- A web-based interface for interacting with Elasticsearch data
- Key features include:
    - **Discover**: Query and explore logs using ESQL
    - **Kibana Lens**: Build visualizations and dashboards
- Enables analysts to:
    - Search and analyze data
    - Create visualizations and dashboards
    - Generate reports
    - Configure alerts
### **Key Benefits of the ELK Stack**
- **Centralized Logging**: Aggregate and search logs in one place (useful for compliance and investigations)
- **Flexibility**: Highly customizable data ingestion and processing
- **Visualization**: Gain insights quickly through dashboards
- **Scalability**: Designed to handle growth in data and infrastructure
- **Ecosystem**: Strong community support and extensive integrations
## Elasticsearch Setup

### 1. Download Ubuntu Server
- Download [Ubuntu Server 24.04 LTS](https://ubuntu.com/download/server )
- This will be used inside VirtualBox

---

### 2. Configure VirtualBox NAT Network
1. Open **VirtualBox**
2. Navigate to: **File → Tools → Network Manager**
3. Select **NAT Networks** → Click **Create**
4. Configure:
   - **Name:** `ELK-SOC-Lab`
   - **IPv4 Prefix:** `172.31.0.0/24`
5. Click **Apply**

---

### 3. Create Ubuntu Server VM
1. Go to: **Machine → New**
2. Configure:
   - **Name:** `ELK-VM`
   - **ISO Image:** Ubuntu Server ISO
   - Enable: **Skip Unattended Installation**
3. Hardware:
   - **Memory:** 16384 MB
   - **CPU:** 2 processors
4. Storage:
   - **Disk Size:** 50 GB
5. After creation:
   - Right-click **ELK-VM → Settings**
   - Go to **Network**
   - Set:
     - **Attached to:** NAT Network
     - **Network Name:** `ELK-SOC-Lab`
   - Click **OK**

---

### 4. Install Ubuntu Server
1. Start the VM
2. Proceed through installer:
   - Press **Enter** through most prompts
   - Set credentials at **Profile Setup**
3. On **SSH Setup** screen:
   - Press **Spacebar** to enable **Install OpenSSH Server**
4. Finish installation and reboot
   - If you see `Failed unmounting cdrom.mount`, press **Enter** (safe to ignore)

---

### 5. Enable SSH Access

#### Get VM IP
```bash
ip a
```
- Example: `172.31.0.4`

#### Enable SSH
```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

#### Configure Port Forwarding (VirtualBox)
1. Go to **Network Manager → NAT Networks**
2. Click **Port Forwarding**
3. Add rule:
   - **Name:** SSH
   - **Host IP:** 127.0.0.1
   - **Host Port:** 2222
   - **Guest IP:** 172.31.0.4
   - **Guest Port:** 22

#### Connect via SSH
```bash
ssh <username>@127.0.0.1 -p 2222
```
- Type `yes` when prompted
- Enter credentials

---

### 6. Install Elasticsearch

#### Update System
```bash
sudo apt update && sudo apt upgrade
```

#### Take Snapshot (Recommended)
- VirtualBox → **Machine → Take Snapshot**
- **Name:** `Updated-BaseInstall`

#### Download Elasticsearch
- Copy `.deb` link from:  
  https://www.elastic.co/downloads/elasticsearch

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.3.3-amd64.deb
```

#### Install Package
```bash
ls
sudo dpkg -i elasticsearch-9.3.3-amd64.deb
```

#### Important
- Save the **Security autoconfiguration information**
  - Contains auto-generated credentials