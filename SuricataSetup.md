# 🛡️ Suricata Installation & Configuration  

This guide explains how to install and configure **Suricata IDS** on Ubuntu, enable JSON logging, create custom rules, and test detection with malicious PCAP files.  

## 1️⃣ Install Suricata  
Update system and install Suricata and Utilities from Ubuntu repo (Suricata 7.x is available in 22.04+).  

```bash
sudo apt update
sudo apt install -y suricata jq git curl wget unzip tcpdump
```

jq, wget, curl, unzip, tcpdump are helper tools for PCAP handling and log parsing.

## 2️⃣ Verify Installation
Check Suricata version:
```bash
suricata --build-info
```
Expected Output:

<img width="760" height="110" alt="image" src="https://github.com/user-attachments/assets/6cd79ecc-3bac-49f7-b8bc-515e0229dd4f" />

## 3️⃣ Update Suricata Rules
Suricata uses rule sets (Emerging Threats Open). Install updater and fetch rules:
```bash
sudo suricata-update
sudo systemctl restart suricata
```

## 4️⃣ Configure Core Settings (suricata.yaml)
> File: `/etc/suricata/suricata.yaml`  
> Tip: In **vim**, type `/` followed by the **Search for** text to jump.  
> Example: `/outputs:` → press **Enter** → navigate with `n` (next match) or `N` (previous).

### 4.1 Enable fast.log (optional, human-readable triage)
**Search for:** `outputs:` 
```yaml
outputs:
  - fast:
      enabled: yes
      filename: /var/log/suricata/fast.log    # absolute path
      append: yes
```

Notes:
- Keep only one top-level `outputs`: key in the file. If you see multiple, merge them.
- `fast.log` is handy for quick local checks during PCAP testing.

### 4.2 Rule path (where Suricata expects rule files)
**Search for:** `default-rule-path:`
Set/confirm
```yaml
default-rule-path: /var/lib/suricata/rules
```

### 4.3 Rule files list (which files to load)
**Search for:** `rule-files:`
Set/confirm
```yaml
rule-files:
  - suricata.rules          # main compiled set from suricata-update
  # - local.rules           # (optional) include only if you want it auto-loaded
```

### 4.4 Network variables: HOME_NET / EXTERNAL_NET
**Search for:** `ars:` (first occurrence that contains `address-groups:`)
Use one 'vars:` block; remove duplicates elsewhere.
```yaml
vars:
  address-groups:
    HOME_NET: "[192.168.0.0/16,10.0.0.0/8,172.16.0.0/12,fc00::/7]"
    EXTERNAL_NET: "!$HOME_NET"
```
Notes:
- Including `fc00::/7` covers IPv6 ULA ranges used in labs.
- Having a precise `HOME_NET` reduces false positives.

### 4.5 Server groups (generic for mixed PCAPs)
**Search for:** `HTTP_SERVERS:`
Set:
```yaml
    HTTP_SERVERS: "any"
```

**Search for:** `SMTP_SERVERS:`
Set:
```yaml
    SMTP_SERVERS: "$HOME_NET"
```

Notes:
- `HTTP_SERVERS: "any"` is helpful when replaying diverse PCAPs where servers may not live in `HOME_NET`.

### 4.6 Live capture config (optional; ignored for -r <pcap>)
**Search for:** `af-packet:`
This is not required for pcap, but required for suricata test
```yaml
af-packet:
   - interface: <IFACE>   # remove <IFACE>, add your interface name
     cluster-id: 99
     cluster-type: cluster_flow
     defrag: yes
```

### 4.7 Validate config and restart
Run a config test:
```yaml
sudo suricata -T -c /etc/suricata/suricata.yaml
```
Expected output:

<img width="455" height="49" alt="image" src="https://github.com/user-attachments/assets/ab042e29-23e1-4b22-bc90-64d3b095f70e" />

Restart Suricata:
```yaml
sudo systemctl restart suricata
```


## 5️⃣ Test Suricata with a Custom Rule & Live Traffic  

In this step we will:  
1. Create a custom Suricata rule (`BadAgent`)  
2. Organize a test directory for rules and logs  
3. Generate malicious traffic locally  
4. Validate alerts in Suricata logs  


### 5.1 Create a working directory  

```bash
mkdir -p /home/ubuntu/suricatatest/logs
cd /home/ubuntu/suricatatest
```

### 5.2 Add a custom rule
Create a new rules file:
```bash
vim /home/ubuntu/suricatatest/local.rules
```
Add this rule:
```bash
alert http any any -> any any (msg:"TEST BadAgent seen"; content:"BadAgent"; http_header; sid:1000001; rev:1;)
```
Save & quit (:wq).

### 5.3 Configure logging directory  

1. **Copy the base config** from `/etc/suricata/` into your test folder:  
```bash
cp /etc/suricata/suricata.yaml /home/ubuntu/suricatatest/suricta_test.yaml
```

2. **Edit the copy** with `vim`:
```bash
vim /home/ubuntu/suricatatest/suricta_test.yaml
```

3. **Set the log directory**
Search for `/default-log-dir` and change it to your test path:
```yaml
default-log-dir: /home/ubuntu/suricatatest/logs
```

4. **Enable fast.log output**
Search for `/outputs:` and ensure this block exists:
```yaml
outputs:
  - fast:
      enabled: yes
      filename: fast.log    # relative to default-log-dir
      append: yes
```

5. **Save & exit:** `:wq`

### 5.4 Run Suricata in live mode
Run Suricata on your active network interface (replace \<IFACE\> with the correct NIC, e.g., enp0s3,eth0):
```bash
sudo suricata \
  -c /home/ubuntu/suricatatest/suricta_test.yaml \
  -S /home/ubuntu/suricatatest/local.rules \
  -i <IFACE> \
  -l /home/ubuntu/suricatatest/logs \
  -vv
```

### 5.5 Generate malicious traffic in another tab
Send an HTTP request with the BadAgent user-agent.
⚠️ Must use plain HTTP (not HTTPS) so Suricata can parse headers.
```bash
curl --http1.1 -A "BadAgent" http://neverssl.com/ >/dev/null 2>&1
```

### 5.6 Check the logs
cat /home/ubuntu/suricatatest/logs/fast.log

**Expected Output:**
<img width="694" height="22" alt="image" src="https://github.com/user-attachments/assets/393f93ec-8bd4-408a-aa4f-e67b04e4b38f" />










