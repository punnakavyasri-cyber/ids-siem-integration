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
Save & quit `(:wq)`.

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

## 5️⃣ Generate Events for Verification  

Before checking Splunk, we need Suricata to produce logs from real traffic.  
We will create a lab folder, download a malicious PCAP file, replay it with `tcpreplay`, and then validate both locally (`eve.json`) and inside Splunk.  

---

### 5.1 — Prepare the Lab Directory and PCAP  
1. Create folders for PCAPs:  
```bash
mkdir -p /home/ubuntu/pcaps/lab2
cd /home/ubuntu/pcaps/lab2
```
2. Download a malicious PCAP (Log4j exploitation attempts in this example):
``` bash
wget https://www.malware-traffic-analysis.net/2021/12/13/2021-12-13-server-traffic-with-log4j-attempts.pcap.zip
```
3. Unzip the PCAP:
```bash
unzip 2021-12-13-server-traffic-with-log4j-attempts.pcap.zip
```
This will create the file:
<img width="226" height="10" alt="image" src="https://github.com/user-attachments/assets/ccd9b280-66ff-474e-bad4-c8516743e216" />

### 5.2 — Replay the PCAP with tcpreplay
Run the PCAP through your active interface so Suricata can analyze it:
```bash
sudo tcpreplay -i <IFACE> --pps=200 2021-12-13-server-traffic-with-log4j-attempts.pcap
```
Explanation:
- `-i enp0s3` → replace with your actual interface
- `--pps=200` → send 200 packets per second (you can adjust)
- `.pcap` → the downloaded malicious traffic file

Expected Output:
<img width="257" height="182" alt="image" src="https://github.com/user-attachments/assets/533d95eb-eca8-4096-bc1f-bc342b48c81d" />

### 5.3 — Validate Logs Locally with jq
Check that Suricata generated events before moving to Splunk:
```bash
cat /var/log/suricata/eve.json | jq
```
Example output:
![08-LogsLoadingInToEveJSON](https://github.com/user-attachments/assets/3bb20e24-42dc-4ff9-9b5a-1a3b07f378a6)

You can also inspect full JSON entries by sorting with event_type:
```bash
cat /var/log/suricata/eve.json | jq '.event_type' | sort | uniq -c
```

### 5.4 — Verify in Splunk  

1. Log in to Splunk Web:  
   - Open your browser → `http://localhost:8000` (or your VM’s IP:8000)  
   - Use either:  
     - **Admin credentials** you created during setup, **or**  
     - A **dedicated user** you created (e.g., `soc_user` or `monitor_user`) with access to the `suricata` index.  

2. Navigate to the Search app:  
   - Top menu → **Apps → Search & Reporting**  
   - Click **Search** in the left menu.  

3. Run the following SPL query:  
```spl
index=suricata sourcetype=suricata:json | stats count by event_type
```

4. Review the results. You should see event types such as:
- `alert`
- `http`
- `dns`
- `tls`
- `flow`

**Example — Splunk Stats Page**
![09-SplunkAlertCount](https://github.com/user-attachments/assets/54bc38e1-9401-48ec-b18e-199e30769371)

</br>
</br>

✅ Suricata is now fully integrated with Splunk. Logs from /var/log/suricata/eve.json are being parsed as JSON in the suricata index, ready for dashboards, SPL queries, and alerts.

**Next step →** <a href="https://github.com/punnakavyasri-cyber/ids-siem-integration/blob/main/SplunkGUI.md"> Dashboards, SPL Queries & Alerts </a>

