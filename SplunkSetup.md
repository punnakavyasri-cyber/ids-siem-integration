# Splunk Installation & Configuration  

This section explains how to install Splunk Enterprise, configure an index and users, and prepare it to ingest Suricata logs.  

---

## 1️⃣ Download & Install Splunk Enterprise  

1. Go to the [Splunk Downloads Page](https://www.splunk.com/en_us/download/splunk-enterprise.html)  
   - Choose the **.deb (Ubuntu)** package.  

2. Install the package:
  
```bash
cd /home/
wget -O splunk.deb "https://download.splunk.com/products/splunk/releases/9.3.0/linux/splunk-9.3.0-xxxxxxx-linux-2.6-amd64.deb"
sudo dpkg -i splunk.deb
```

