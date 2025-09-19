# Splunk Dashboard: Threat Analysis Dashboard  

In this step, we build a **Threat Analysis Dashboard** in Splunk using Suricata logs.  
This dashboard provides SOC-style visibility into alerts, attacker IPs, severity trends, and recent activity.  

## 1️⃣ Create a New Dashboard  

1. In Splunk Web → **Apps → Search & Reporting**  
2. Click **Dashboards → Create New Dashboard**  
3. Fill in the details:  
   - **Dashboard Title:** Threat Analysis Dashboard  
   - **Dashboard Studio** (not Classic)  
   - **Layout:** Absolute  
4. Click **Create**

![01-DashboardCreation](https://github.com/user-attachments/assets/8ef1c91b-3f13-407c-8367-525d8241f527)


## 2️⃣ Add Panels (Boxes)  

### Box 1 — Alerts Count (Single Value)  
**Query:**  
```spl
index=suricata (sourcetype=suricata OR sourcetype=suricata:json) event_type=alert
| stats count as total_alerts
```

### Box 2 — Signature-Based Alert Count
**Query:**
```spl
index=suricata (sourcetype=suricata OR sourcetype=suricata:json) event_type=alert
| stats count as alerts by alert.signature
```

### Box 3 — Top Attacker IPs
(Excludes local/private IPs)
**Query:**
```spl
index=suricata (sourcetype=suricata OR sourcetype=suricata:json) event_type=alert
| where src_ip!="10.8.15.133"
    AND NOT (
      cidrmatch("10.0.0.0/8", src_ip)
      OR cidrmatch("172.16.0.0/12", src_ip)
      OR cidrmatch("192.168.0.0/16", src_ip)
    )
| stats count as alerts by src_ip
| sort -alerts
| head 10
| rename src_ip as "Attacker IP", alerts as "Alert Count"
```

### Box 4 — Attack Trends (Last 10 Days)
Query:
```spl
index=suricata (sourcetype=suricata OR sourcetype=suricata:json) event_type=alert
| timechart span=1d count as total_alerts
| tail 10
| rename total_alerts as "Alert Count"
```

### Box 5 — Severity Mapping Over Time
Query:
```spl
index=suricata (sourcetype=suricata OR sourcetype=suricata:json) event_type=alert
| eval severity_num=tonumber(alert.severity)
| eval sev_label=case(
    severity_num=1,"High",
    severity_num=2,"Medium",
    severity_num=3,"Low",
    true(),"Unknown"
)
| timechart span=5m count by sev_label
```

### Box 6 — Protocol Breakdown
Query:
```spl
index=suricata (sourcetype=suricata OR sourcetype=suricata:json) event_type=alert
| eval app=coalesce(app_proto, proto)
| stats count as alerts by app
| sort -alerts
| rename app as Protocol, alerts as "Alert Count"
```

### Box 7 — Recent Alerts (Table)
Query:
```spl
index=suricata (sourcetype=suricata OR sourcetype=suricata:json) event_type=alert
| eval severity_num = tonumber(alert.severity)
| eval sev_label = case(
    severity_num==1, "High",
    severity_num==2, "Medium",
    severity_num==3, "Low",
    true(), "Unknown"
)
| table _time sev_label src_ip dest_ip alert.signature alert.category app_proto http.hostname http.url
| sort -_time
| head 20
```

**My Dashboard:**
![04-SplunkDashboardComplete](https://github.com/user-attachments/assets/f63d0283-c5fa-4d7a-876a-b359ccb0dbc5)


## 3️⃣ Set as Home Dashboard
1. Go to the **Splunk Home Page**
2. Click **Dashboard → Choose Home Dashboard**
3. Select **Threat Analysis Dashboard**

**My Dashboard:**
![07-FinalHomeDashboard](https://github.com/user-attachments/assets/a58251e4-cc4e-45c5-8313-609cd8e00e35)

Now, whenever you log in, Splunk opens directly to your security dashboard.

## 4️⃣ Access Control — Share the Dashboard with Other Users  

By default, the dashboard is owned by the `admin` user. To allow other users (like `monitor_user` or `soc_user`) to view it, update the permissions:  

1. In Splunk Web → **Apps → Search & Reporting**  
2. Go to **Dashboards** → Find **Threat Analysis Dashboard**  
3. Click **Edit → Edit Permissions**  
4. In the **Edit Permissions** window:  
   - Choose **App** (so it’s visible to all users of the Search app)  
   - Under Roles, check **Read** for `user` and/or `power` (depending on the roles your custom users have)  
   - Optionally grant **Write** if you want them to edit the dashboard  
5. Click **Save**  

Now your dashboard will be visible for:  
- The `admin` user  
- Any additional users you created (e.g., `monitor_user`) who belong to roles with **Read** access.  

**Example — Edit Permissions Window**  
![10-Permissions](https://github.com/user-attachments/assets/f803c244-e33c-4246-bd17-8cb163becbd4)
 

---

## 5️⃣ Create Alerts (Splunk UI Only)  

In addition to dashboards, Splunk lets you configure alerts that fire when Suricata logs match certain conditions.  
Since email notifications are not configured in this lab, alerts will trigger **inside Splunk UI** under *Triggered Alerts*.  

---

### 5.1 — Create a New Alert  

1. In Splunk Web → **Apps → Search & Reporting**  
2. Run the SPL query you want to monitor. Example:  

```spl
index=suricata (sourcetype=suricata OR sourcetype=suricata:json) event_type=alert
| stats count by alert.signature
```
3. From the search window, click Save As → Alert

### 5.2 — Configure Alert Settings
- **Title:** Suricata – High Severity Alerts
- **Description:** Optional
- **Alert Type:** `Scheduled`
- **Schedule:** Every 15 minutes (`*/15 * * * *`)
- **Time Range:** Last 15 minutes
- **Expires:** 24 days (default)
![01-AlertNotifications](https://github.com/user-attachments/assets/51d30926-3c59-4b63-b2e1-f12f8e545e73)


### 5.3 — Trigger Conditions
- **Trigger alert when:** Number of results is greater than 0
- **Trigger:** Once
- **Throttle:** Enabled
- **Suppress results by:** `src_ip`
- **Suppress triggering for:** 10 minutes (to avoid repeated alerts for the same IP)
![02-AlertNotifications2](https://github.com/user-attachments/assets/4eaf8ba0-8376-4f81-905a-df40c16f6bfb)


### 5.4 — Alert Permissions  

After saving an alert:  
1. Go to **Search & Reporting → Alerts**  
2. Find your alert → **Edit → Edit Permissions**  
3. Set **Display For: App** (or All apps if needed)  
4. Grant **Read** to `user` / `power` roles (and Write if needed)  
5. Save  

This makes the alert visible to non-admin users (e.g., `monitor_user`).  
<br>
<br>
**Next step** → <a href="https://github.com/punnakavyasri-cyber/ids-siem-integration/blob/main/FinalOutput.md"> **Final Output & SOC Workflow** </a>
