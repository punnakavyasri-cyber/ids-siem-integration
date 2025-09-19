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
```

## 3️⃣ Start Splunk for the First Time
Run the following command to start Splunk and accept the license agreement:  
```bash
sudo /opt/splunk/bin/splunk start --accept-license
```
- When prompted, type `y` to agree to the license.
- You will be asked to set the admin username and password.
  - These are your Splunk administrator credentials.
  - They will have full administrative rights inside Splunk.
  - Choose a strong password and keep it safe.

## 4️⃣ Enable Splunk to Start on Boot
To make sure Splunk runs automatically after a reboot:
```bash
sudo /opt/splunk/bin/splunk enable boot-start
```
- If prompted, type `y` to confirm.

## 5️⃣ Access the Splunk Web Interface
- Open your browser and go to: http://localhost:8000 (or) http://127.0.0.1:8000
  - If accessing remotely, replace `localhost` with the IP address of your VM (e.g., `http://192.168.1.100:8000`). <br>
- Login using the admin credentials you just created.
  - Username: `admin` (or the username you set)
  - Password: the one you chose during setup

**Splunk Login Page**
![01-LoginPage](https://github.com/user-attachments/assets/3fe562ae-01a3-4f1b-b97a-75a10c81c3b1)

## 6️⃣ Create a Dedicated User (Optional but Recommended)

Instead of always using the `admin` account, it is a good practice to create a separate user for SOC or monitoring tasks.

1. In Splunk Web → **Settings → Access Controls → Users → New User**  
2. Provide:
   - **Username:** e.g., `monitor_user` or `soc_user`
   - **Full Name:** optional (e.g., SOC Analyst, Monitor User)
   - **Password:** choose a strong password (at least 8 characters)
   - **Time Zone:** select your preference (default is fine)
   - **Default App:** `Search & Reporting`
   - **Assign Roles:** at minimum `user` (add `power` if you want extended search capabilities)
3. Click **Create**.

**Splunk User Creation Page**  
![01-MonitorUserCreation](https://github.com/user-attachments/assets/a9527130-baa8-437e-88ef-a3eed803a9a5)

<br>
<br>

**✅ Splunk Enterprise is now installed and configured with administrative access and a dedicated user.**
<br>
<br>
**Next step** → <a href="https://github.com/punnakavyasri-cyber/ids-siem-integration/blob/main/SuricataSplunkIntegration.md"> **[Suricata–Splunk Integration]** </a>




  
