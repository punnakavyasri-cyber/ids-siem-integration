# IDS-SIEM INTEGRATION
This project demonstrates the integration of Suricata IDS with Splunk SIEM to replicate real-world SOC workflows. It begins with downloading malicious PCAP files from trusted sources like Malware Traffic Analysis, configuring Suricata to detect suspicious traffic, and forwarding logs to Splunk for analysis and visualization.

The project covers custom Suricata rule creation, SPL query development, dashboard building, and alert configuration. By simulating attacks like Log4j exploitation and WarmCookie malware campaigns, it showcases practical skills in threat detection, alert triage, and incident investigation that align with professional SOC operations and incident response workflows.

## 🎯 Purpose of the Project  

- Gain hands-on experience with **IDS (Suricata)** and **SIEM (Splunk)** integration  
- Practice **SOC analyst workflows** such as detection, triage, and investigation  
- Analyze **malicious PCAP files** to understand real-world attack traffic  
- Develop and test **custom Suricata detection rules** for known threats  
- Create **SPL queries, dashboards, and alerts** to monitor attacker activity  
- Simulate and analyze multiple attack scenarios:  
  - **Log4j exploitation** (CVE-2021-44228)  
  - **WarmCookie malware campaign** (2024-08-15)
  - **You Dirty Rat!** (2024-07-30)  
- Build a repeatable lab environment that mirrors **professional SOC operations**  
 
## 🛠️ Technologies Used  

- **Suricata IDS/IPS** – PCAP replay, log generation, custom rules  
- **Splunk Enterprise** – SIEM for log ingestion, SPL queries, dashboards, alerts  
- **Malware Traffic Analysis PCAPs** – Log4j, WarmCookie 
- **Ubuntu 22.04** – Lab environments  
- **EVE JSON + SPL** – Log format and query language  
- **Utilities** – tcpdump, wget/curl, jq, Bash, unzip  

## 🏗️ Project Architecture

### High-Level Flow

![Architecture Diagram](https://github.com/user-attachments/assets/d9cae879-4264-42c9-8ef6-d99771cdcaf0)

## 📋 Prerequisites  

- **Ubuntu 22.04 VM** or any Linux VM (4+ vCPU, 12 GB RAM, 60 GB disk)  
- **Splunk Enterprise** (trial or free edition)  
- **PCAP files** (malicious traffic samples from trusted sources)  
- **Foundational Knowledge**: basic networking protocols, ports, and log management  

## 🚀 Project Workflow  

1. Set up **Suricata IDS** on Ubuntu, enable EVE JSON logging, and test detection using PCAP replay and custom rules. <br>
2. Install and configure **Splunk Enterprise**, create indexes and users, and prepare it to ingest Suricata logs.  
3. Integrate **Suricata with Splunk** by modifying configuration files and setting up data inputs in the Splunk UI.  
4. Build **dashboards, SPL queries, and alerts** in Splunk to detect and visualize threats.  
5. Run a new **PCAP file** through Suricata and validate detections in dashboards and alerts.

## 📑 Table of Contents  

1. Step 1 — <a href="https://github.com/punnakavyasri-cyber/ids-siem-integration/blob/main/SuricataSetup.md"> Suricata Installation & Configuration </a>   
2. Step 2 — <a href="https://github.com/punnakavyasri-cyber/ids-siem-integration/blob/main/SplunkSetup.md"> Splunk Installation & Configuration </a>  
3. Step 3 — <a href="https://github.com/punnakavyasri-cyber/ids-siem-integration/blob/main/SuricataSplunkIntegration.md"> Suricata–Splunk Integration </a>  
4. Step 4 — <a href="https://github.com/punnakavyasri-cyber/ids-siem-integration/blob/main/SplunkGUI.md"> Dashboards, SPL & Alerts </a>
5. Step 5 — <a href="https://github.com/punnakavyasri-cyber/ids-siem-integration/blob/main/FinalOutput.md"> Final Output & SOC Workflow </a>

## 🔮 Future Expansions  

- **Multi-Host Environment** – Add two or more additional hosts to Splunk for distributed log collection and analysis.  
- **Splunk Forwarder Integration** – Use the Universal Forwarder to send Suricata logs instead of local file monitoring.  
- **Enable IPS Mode** – Configure Suricata not only as an IDS but also as an Intrusion Prevention System.  
- **Log Source Expansion** – Ingest additional logs (e.g., Windows Event Logs, Linux Syslogs, Firewall logs) to enrich analysis.  
- **Threat Intelligence Feeds** – Integrate Suricata with external threat intel sources (ET Pro, Abuse.ch, MISP) for improved detection.  
- **Automation & SOAR** – Automate responses to alerts with scripts or integrate with SOAR platforms.  
- **Cloud Deployment** – Replicate the entire lab in AWS or Azure for scalable testing and training.  
