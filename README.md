I built a fully functional, on-prem SOC lab from scratch using the ELK Stack, Sysmon, and Mythic C2, configuring endpoints to forward logs, creating detection rules and dashboards, and simulating real-world attacks like brute force activity. By generating and investigating my own attack data, I gained hands-on experience with log ingestion, threat detection, alert tuning, and incident response workflows. This project strengthened both my technical understanding of security operations and my ability to document and present complex findings in a clear format.
# Creating a Logical Diagram
Using draw.io, I created the following diagram to clearly and effectively illustrate the lab environment:

![](attachments/Pasted%20image%2020260418222119.png)

### ELK Stack Introduction

The **ELK Stack** is a powerful combination of tools used for centralized logging, analysis, and visualization of security data.
#### **Elasticsearch (E): Data Storage & Search**
- A high-performance database for storing logs (Windows events, syslogs, firewall logs, etc.)
- Enables fast, flexible searching across large datasets
- Uses **ESQL** for querying data
- Supports RESTful APIs and JSON, making it easy to integrate with other tools and automate data retrieval
#### **Logstash (L): Data Collection & Processing**
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
#### **Kibana (K): Visualization & Analysis**
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
### Elasticsearch Setup
1. Download [Ubuntu Server 24.04 LTS](https://ubuntu.com/download/server), which we will be hosting using VirtualBox
2. Within VirtualBox, will first setup our network by doing the following:
	1. Click on **File -> Tools -> Network Manager**
	2. Click on the **NAT Networks** column, then click on **Create**
	3. Change the name of the NatNetwork to something more fitting (i.e. ELK-SOC-Lab)
	4. Change the **IPv4 Prefix** to match what we set it to in our logic diagram: **172.31.0.0/24**
	5. Once done, click **Apply** to save these changes
3. Now that our network has been configured, we can now proceed with setting up our Ubuntu Server VM:
	1. Back on VirtualBox, click on **Machine -> New**
	2. We will name this **ELK-VM**
	3. Choose to set the **Folder** as the default option, or choose a dedicated directory
	4. For **ISO Image**, choose the Ubuntu Server ISO file downloaded using the link above
	5. Make sure to check **Skip Unattended Installation**, then click **Next**
	6. On the following page, set the **Base Memory** to **16384 MB**
	7. Set the **Processors** to **2**, then click **Next**
	8. Set the **Virtual Hard Disk** size to **50 GB**
	9. Once finished, right-click on the **ELK-VM**, and open up the **Settings**
	10. Navigate to **Network**, set Attached to: **NAT Network** and choose **ELK-SOC-Lab**
	11. Once done, click OK
4. We can now proceed with setting up our Ubuntu VM by starting it in VirtualBox:
	1. Proceed with clicking **Enter** until we get to the **Profile setup** screen, and set the credentials to anything you'd like
	2. Continue pressing enter until you reach the **SSH Setup** page, where you will press **Spacebar** to toggle the **Install OpenSSH Server** option
	3. Once done, the server will finalize the installation and prompt to reboot
	4. Don't worry about receiving a **Failed unmounting cdrom.mount** error message, simply press **Enter** and the VM will reboot
5. We must now ensure we can SSH into this server:
	1. Once logged in, enter the `ip a` command to receive local IP address assigned by the **NAT Network's DHCP**
	2. In my case, my IP for **ELK-VM** is **172.31.0.4**
	3. Now, let's ensure SSH is enabled and running by entering the following commands: `systemctl enable ssh` and `systemctl start ssh`
	4. Once done, return to the VirtualBox main menu and re-open the **NAT Networks** menu
	5. On the bottom of the screen, click on **Port Forwarding**, then click on the the **green create icon** on the right-hand side
	6. Set **Name** to SSH, **Host IP** to 127.0.0.1, **Host Port** to 2222, **Guest IP** to 172.31.0.4, and **Guest Port** to 22
	7. Now, establish an SSH connection with the VM from your host machine using either **PowerShell, Terminal, etc.**
	8. Enter the command `ssh <username>@127.0.0.1 -p 2222`
	9. Enter **yes** to establish the connection, enter the login credentials, and you should now be SSH'd into **ELK-VM**
6. Actually setting up Elasticsearch:
	1. Update the VM by entering `sudo apt update && sudo apt upgrade`
	2. Once updated, navigate to the open VirtualBox VM window and click on **Machine -> Take Snapshot**
	3. Set the **Snapshot Name** to **Updated-BaseInstall**
	4. We now want to download the **deb x86_64** version of [Elasticsearch](https://www.elastic.co/downloads/elasticsearch) by right-clicking on the blue download button and clicking **Copy link address**
	5. In the VM, enter the command `wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.3.3-amd64.deb`
	6. Once downloaded, type in `ls` to confirm the .deb file is available in the current directory
	7. To install this, enter `sudo dpkg -i elasticsearch-9.3.3-amd64.deb`
	8. Once installed, make sure to take note of the **[[Security autoconfiguration information]]**, as this will contain the auto-generated Elasticsearch credentials