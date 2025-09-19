# Final Output & SOC Workflow  

This step shows the full end-to-end workflow of a SOC analyst:  
- Replay malicious traffic  
- Monitor dashboards in real time  
- Respond to triggered alerts  
- Drill into logs for investigation


## 1️⃣ Replay the PCAP  

We will use the **2024-08-15-traffic-analysis-exercise.pcap** file.  

```bash
cd /home/ubuntu/pcaps/lab3
sudo tcpreplay -i <IFACE> --pps=200 2024-08-15-traffic-analysis-exercise.pcap
```
- -i enp0s3 → network interface
- --pps=200 → replay at 200 packets per second

**Screenshot:** <br>
<img width="524" height="92" alt="image" src="https://github.com/user-attachments/assets/2566f225-ccfa-462e-980d-ee2029798d79" />


## 2️⃣ Live Dashboard Monitoring

As soon as the replay begins, switch to the **Threat Analysis Dashboard** in Splunk.
- Within a few minutes, panels will update:
  - Alerts Count increases
  - Top Attacker IPs populated
  - Recent Alerts table shows new entries

**Screenshot:** <br>
<img width="761" height="415" alt="image" src="https://github.com/user-attachments/assets/c636123f-d740-42e6-9e9a-fc15bdee7e03" />


## 3️⃣ Alert Triggered  

Alerts configured earlier (e.g., *High Severity*, *Log4j*, or *WarmCookie*) should fire automatically within ~5 minutes.  
- Navigate: **Activity → Triggered Alerts**  
- Confirm the new alert fired  

**Screenshot:** </br>
<img width="761" height="359" alt="image" src="https://github.com/user-attachments/assets/238e986e-607e-4d76-a7dc-18cd4d572197" />


## 4️⃣ Investigating the Alert (SOC Workflow)
Once the alert is triggered, a SOC analyst investigates:
- **Step 1:** Identify the signature
```spl
index=suricata event_type=alert | stats count by alert.signature
```
**Screenshot:** <br>
<img width="761" height="413" alt="image" src="https://github.com/user-attachments/assets/49bf7a98-2331-4679-bd0d-3613473a95f8" />


- **Step 2:** Find source and destination IPs
```spl
index=suricata event_type=alert | table _time src_ip dest_ip alert.signature
```
**Screenshot:** <br>
<img width="761" height="413" alt="image" src="https://github.com/user-attachments/assets/53c0e515-69ff-44a0-94ca-7f71e71c2d01" />


- **Step 3:** Enrich with protocol and host details
```spl
index=suricata event_type=alert 
| where NOT cidrmatch("10.0.0.0/8", src_ip) 
    AND NOT cidrmatch("172.16.0.0/12", src_ip) 
    AND NOT cidrmatch("192.168.0.0/16", src_ip) 
    AND NOT cidrmatch("fc00::/7", src_ip)
| table _time src_ip dest_ip app_proto http.hostname http.url alert.signature
| sort -_time
```
**Screenshot:** <br>
<img width="766" height="413" alt="image" src="https://github.com/user-attachments/assets/f8465cd4-baf6-4f74-891c-9b1cbd1ee033" />

## 5️⃣ Investigating a Specific Event  

Once an alert is triggered, the SOC analyst drills deeper to understand the threat. 


### 5.1 — Drill Down from Signature  

1. From the **Statistics** tab (e.g., `stats count by alert.signature`)  
2. Click on the suspicious signature (e.g., `ET MALWARE BadSpace/WarmCookie CnC Activity (GET)`)  
3. Select **View events**  

This will show all raw events tied to that signature.  

**Screenshot:** <br>
<img width="763" height="410" alt="image" src="https://github.com/user-attachments/assets/38159de0-5c2a-4b4e-a6e1-59ed66bc9833" />

### 5.2 — Detailed Event Analysis 
**Key Fields Observed:**  
- **alert.signature** → Rule triggered (e.g., WarmCookie Command & Control)  
- **alert.category** → Malware / C2 traffic  
- **alert.severity** → Numeric severity (1=High, 2=Medium, 3=Low)  
- **src_ip: 10.8.15.133** → Internal host (victim machine)  
- **src_port: 49882** → Ephemeral source port  
- **dest_ip: 72.5.43.29** → External attacker/C2 server  
- **dest_port: 80** → HTTP service contacted  
- **app_proto: http** → Application layer protocol detected  
- **flow_id: 1998052974103352** → Unique flow identifier (useful to pivot to related events)  
- **timestamp** → Exact time of detection  

**Screenshot:** <br>
<img width="767" height="437" alt="image" src="https://github.com/user-attachments/assets/6dce0b09-5754-4315-8ce5-edfe75a8cdd7" />


**SOC Analyst Interpretation:**  
1. Internal host `10.8.15.133` initiated outbound HTTP traffic.  
2. Destination `72.5.43.29:80` is suspicious and flagged by Suricata as **WarmCookie CnC traffic**.  
3. Signature category indicates **malware activity**.  
4. Next investigative steps:  
   - Pivot in Splunk by `flow_id` to see all related traffic:  
     ```spl
     index=suricata flow_id=1998052974103352
     ```  
   - Expand `http` fields (hostname, URL, user-agent) for further context.  
   - Check if other internal systems contacted the same IP.  
   - Escalate to block the IP/domain and investigate the host for compromise. 

<br>

✅ This demonstrates how Suricata alerts give analysts visibility into **who was attacked, what was targeted, and how the attack occurred**.  
<br>
<br>
**Back to** → <a href="https://github.com/punnakavyasri-cyber/ids-siem-integration/blob/main/README.md"> **Main README** </a>
