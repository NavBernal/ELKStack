I built a fully functional SOC lab from scratch using the ELK Stack, Sysmon, and Mythic C2, configuring endpoints to forward logs, creating detection rules and dashboards, and simulating real-world attacks like brute force activity. By generating and investigating my own attack data, I gained hands-on experience with log ingestion, threat detection, alert tuning, and incident response workflows. This project strengthened both my technical understanding of security operations and my ability to document and present complex findings in a clear format.
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