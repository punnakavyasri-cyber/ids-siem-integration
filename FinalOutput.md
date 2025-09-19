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

**Screenshot:**

## 2️⃣ Live Dashboard Monitoring

As soon as the replay begins, switch to the **Threat Analysis Dashboard** in Splunk.
- Within a few minutes, panels will update:
  - Alerts Count increases
  - Top Attacker IPs populated
  - Recent Alerts table shows new entries

**Screenshot:**

## 3️⃣ Alert Triggered

Alerts configured earlier (e.g., High Severity, Log4j, or WarmCookie) should fire automatically within ~5 minutes.
- Navigate: **Search & Reporting → Alerts → View Triggered Alerts**
- Confirm the new alert fired

**Screenshot:**

## 4️⃣ Investigating the Alert (SOC Workflow)
Once the alert is triggered, a SOC analyst investigates:
- **Step 1:** Identify the signature
```spl
index=suricata event_type=alert | stats count by alert.signature
```
**Screenshot:**

- **Step 2:** Find source and destination IPs
```spl
index=suricata event_type=alert | table _time src_ip dest_ip alert.signature
```
**Screenshot:**

- **Step 3:** Enrich with protocol and host details
```spl
index=suricata event_type=alert 
| table _time src_ip dest_ip app_proto http.hostname http.url alert.signature
```
**Screenshot:**

This shows exactly what traffic triggered the alert.

✅ With this flow, you demonstrate the real SOC cycle:
- Detection → Dashboard
- Alerting → Triggered Alerts
- Investigation → Log & query analysis
