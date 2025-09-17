# Splunk Installation & Configuration  

This section explains how to install **Splunk Enterprise**, configure an index and users, and prepare it to ingest Suricata logs.  


## 1️⃣ Download Splunk Enterprise  

- Go to the [Splunk Downloads Page](https://www.splunk.com/en_us/download/splunk-enterprise.html)  
- Choose **Splunk Enterprise for Linux (.deb package)**  
- Download it to your Ubuntu VM (e.g., into `/tmp/`)

## 2️⃣ Install Splunk  

1. Navigate to the directory where you saved the `.deb` file.  
2. Install the package:  
```bash
sudo dpkg -i splunk-<version>.deb

