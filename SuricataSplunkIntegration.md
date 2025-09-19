# Suricata–Splunk Integration  

This section explains how to integrate **Suricata** with **Splunk Enterprise** by modifying configuration files and setting up data inputs in the Splunk UI.  

---

## 1️⃣ Enable EVE JSON Logging in Suricata  
Edit the default Suricata config:  
```bash
sudo vim /etc/suricata/suricata.yaml
```

Search for `/outputs:` and make sure the eve-log block looks like this:
```yaml
outputs:
  - eve-log:
      enabled: yes
      filetype: regular
      filename: /var/log/suricata/eve.json
      pcap-file: true
      community-id: yes
      community-id-seed: 0
      types:
        - alert:
            tagged-packets: yes
        - anomaly:
            enabled: yes
        - http:
            extended: yes
        - dns
        - tls:
            extended: yes
        - files
        - smtp
        - ssh
        - flow

```
Save & quit `(:wq).`

Restart Suricata to apply changes:
```bash
sudo systemctl restart suricata
```


## 2️⃣ Create a Splunk Index for Suricata Logs
1. In Splunk Web **→ Settings → Indexes → New Index**
2. Fill out the form:
   - **Index Name:** `suricata`
   - **Data Type:** Event
   - **App:** `Search & Reporting` (default)
3. Click **Save**.

<img width="754" height="436" alt="image" src="https://github.com/user-attachments/assets/03afc319-fefd-44c4-b431-8fc5aaae1cf4" />


## 3️⃣ Add a Data Input in Splunk (Web UI Method)
1. In Splunk Web **→ Settings → Data Inputs → Files & Directories → Add New**
2. Enter the Suricata log path:
   - `/var/log/suricata/eve.json`
3. Set:
   - **Sourcetype:** `suricata:json`
   - **Index:** `suricata`
4. Save.

<img width="763" height="433" alt="image" src="https://github.com/user-attachments/assets/7980adbc-88ba-4a10-b186-7a74abb7d046" />


## 4️⃣ Configure Splunk for JSON Parsing (Config File Method)
In addition to the UI, you can configure Splunk’s parsing at the file level.
1. Navigate to your Splunk config directory:
```bash
cd /opt/splunk/etc/system/local/
```
2. Create or edit `props.conf`:
```bash
sudo vim props.conf
```
Add the following block:
```ini
[suricata:json]
INDEXED_EXTRACTIONS = json
KV_MODE = none
NO_BINARY_CHECK = true
TIME_FORMAT = %Y-%m-%dT%H:%M:%S.%3N%z
TIME_PREFIX = "timestamp":
TRUNCATE = 0
```
3. (Optional) If you need custom field extractions, add a `transforms.conf`.
Example:
```ini
sudo vim transforms.conf
```
⚠️ Only add transforms if you need filtering or field renaming. For most Suricata labs, `props.conf` alone is enough.
4. Restart Splunk for changes to take effect:
```bash
sudo /opt/splunk/bin/splunk restart
```

