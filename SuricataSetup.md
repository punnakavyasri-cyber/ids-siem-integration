# 🛡️ Suricata Installation & Configuration  

This guide explains how to install and configure **Suricata IDS** on Ubuntu, enable JSON logging, create custom rules, and test detection with malicious PCAP files.  

## 1️⃣ Install Suricata  

Update system and install Suricata from Ubuntu repo (Suricata 7.x is available in 22.04+).  

```bash
sudo apt update
sudo apt install -y suricata jq git curl wget unzip tcpdump
