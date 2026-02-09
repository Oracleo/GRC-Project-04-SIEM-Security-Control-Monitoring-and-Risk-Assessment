# Mini SOC Lab – Brute-Force Detection & Incident Response

[![Splunk](https://img.shields.io/badge/Splunk-Enterprise-green)](https://www.splunk.com/)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red)](https://attack.mitre.org/)
[![Status](https://img.shields.io/badge/Status-Complete-success)]()

A hands-on SOC project demonstrating real-time brute-force attack detection using Splunk SIEM, Windows Event Logs, and incident response workflow.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Project Objectives](#project-objectives)
- [Architecture](#architecture)
- [Technologies Used](#technologies-used)
- [Setup Guide](#setup-guide)
- [Detection Method](#detection-method)
- [Key Findings](#key-findings)
- [MITRE ATT&CK Mapping](#mitre-attck-mapping)
- [Deliverables](#deliverables)
- [Screenshots](#screenshots)
- [Lessons Learned](#lessons-learned)
- [Future Enhancements](#future-enhancements)
- [Author](#author)

---

## 🎯 Overview

This project simulates a real-world SOC (Security Operations Center) environment where a brute-force authentication attack is detected, investigated, and documented using industry-standard tools and methodologies. The lab demonstrates practical skills in SIEM deployment, log analysis, threat detection, and incident response.

**Incident Summary:**
- **Attack Type:** Brute-Force Authentication Attack
- **Target:** Windows 10 user account (`testuser`)
- **Detection Time:** Real-time (< 1 minute)
- **Failed Attempts:** 5+ within short timespan
- **Status:** Successfully detected and documented

---

## 🎓 Project Objectives

1. **Deploy SIEM Infrastructure:** Set up Splunk Enterprise on Kali Linux for centralized log monitoring
2. **Configure Log Collection:** Forward Windows Security Event Logs to Splunk using Universal Forwarder
3. **Simulate Attack:** Generate brute-force login attempts to trigger security events
4. **Detect Threats:** Create SPL queries to identify suspicious authentication patterns
5. **Create Alerts:** Configure real-time alerting for brute-force detection
6. **Document Incident:** Follow SOC workflow to investigate and document findings
7. **Map to Framework:** Align attack with MITRE ATT&CK framework

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    SOC Lab Architecture                      │
└─────────────────────────────────────────────────────────────┘

Windows 10 Host (Victim)                    Kali Linux VM (SIEM)
IP: 10.232.194.141                         IP: 10.232.194.7
┌──────────────────┐                       ┌──────────────────┐
│                  │                       │                  │
│  Windows User    │                       │  Splunk          │
│   Account:       │                       │  Enterprise      │
│   - testuser     │                       │  - Port 8000     │
│                  │                       │  - Port 9997     │
│  Security Logs   │                       │                  │
│  (EventID 4625)  │                       │  Detection       │
│        │         │                       │  & Alerting      │
│        ▼         │                       │       ▲          │
│  Universal       │    Log Forwarding     │       │          │
│  Forwarder   ────┼──────────────────────►│   Receives       │
│  (Port 9997)     │   TCP Connection      │   Logs           │
│                  │                       │                  │
└──────────────────┘                       └──────────────────┘
         ▲                                          │
         │                                          │
         │ Brute-Force Attack                       │
         │ (Failed Login Attempts)                  │
         │                                          ▼
    ┌────┴────┐                            ┌────────────────┐
    │ Attacker│                            │ SOC Analyst    │
    │ (Local) │                            │ Investigation  │
    └─────────┘                            └────────────────┘
```

**Network Configuration:**
- Both systems on bridged network (same subnet)
- Splunk receiving port: 9997 (configured)
- Splunk web interface: http://10.232.194.7:8000

---

## 🛠️ Technologies Used

| Technology | Purpose | Version |
|------------|---------|---------|
| **Splunk Enterprise** | SIEM platform for log collection and analysis | 10.2.0 |
| **Splunk Universal Forwarder** | Lightweight agent for log forwarding | 10.2.0 |
| **Kali Linux** | SIEM hosting platform | 2026 (VirtualBox) |
| **Windows 10** | Target system generating security events | Build 19045 |
| **VirtualBox** | Virtualization platform | Latest |
| **SPL** | Search Processing Language for queries | - |
| **Windows Event Viewer** | Native log viewing tool | - |

---

## 📖 Setup Guide

### Prerequisites

- VirtualBox installed
- Kali Linux VM (minimum 4GB RAM)
- Windows 10 (host or VM)
- Splunk account (free) - [https://www.splunk.com](https://www.splunk.com)

### Step 1: Install Splunk Enterprise on Kali

```bash
# Download Splunk Enterprise for Linux (.deb)
cd ~/Downloads

# Install Splunk
sudo dpkg -i splunk-10.2.0-*.deb

# Start Splunk and accept license
sudo /opt/splunk/bin/splunk start --accept-license

# Create admin account when prompted
# Username: admin
# Password: <your-password>
```

### Step 2: Configure Splunk Receiving Port

```bash
# Access Splunk web interface
# Open browser: http://127.0.0.1:8000

# Navigate to: Settings > Forwarding and receiving
# Click: Configure receiving > Add new
# Port: 9997
# Save
```

### Step 3: Network Configuration

```bash
# Configure Kali VM network adapter to Bridged mode
# In VirtualBox: Settings > Network > Adapter 1 > Bridged Adapter

# Restart Kali VM and verify IP
ip addr show

# Note the IP address (e.g., 10.232.194.7)
```

### Step 4: Install Splunk Universal Forwarder on Windows

```cmd
# Download Universal Forwarder for Windows
# Install with default settings

# Open Command Prompt as Administrator
cd "C:\Program Files\SplunkUniversalForwarder\bin"

# Start forwarder
splunk.exe start --accept-license

# Create admin account (e.g., drnee)

# Add forward server (use Kali IP)
splunk.exe add forward-server 10.232.194.7:9997
```

### Step 5: Configure Windows Event Log Collection

```cmd
# Navigate to local config directory
cd "C:\Program Files\SplunkUniversalForwarder\etc\system\local"

# Edit inputs.conf (create if not exists)
notepad inputs.conf

# Add these lines:
[WinEventLog://Security]
disabled = 0
index = main

# Save and restart forwarder
cd "C:\Program Files\SplunkUniversalForwarder\bin"
splunk.exe restart
```

### Step 6: Enable Windows Auditing

```cmd
# Enable logon auditing
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```

### Step 7: Verify Log Collection

```
# In Splunk web interface > Search & Reporting
# Run this query:

index=main sourcetype=WinEventLog:Security

# You should see Windows Security events flowing in
```

---

## 🔍 Detection Method

### Detection Query (SPL)

```spl
index=main sourcetype=WinEventLog:Security EventCode=4625
| stats count by Account_Name
| where count >= 3
| sort - count
```

**Query Breakdown:**
- `EventCode=4625` - Failed logon attempts
- `stats count by Account_Name` - Count failures per user
- `where count >= 3` - Filter users with 3+ failures (brute-force threshold)
- `sort - count` - Show highest failure counts first

### Alert Configuration

**Alert Name:** Brute Force Detection - Failed Login Attempts

**Settings:**
- **Type:** Real-time
- **Trigger Condition:** Per-Result
- **Threshold:** 3 or more failed login attempts
- **Action:** Log Event with user details

**Alert Message:**
```
Brute Force Alert: User $result.Account_Name$ has $result.count$ failed login attempts
```

---

## 📊 Key Findings

### Attack Details

| Attribute | Value |
|-----------|-------|
| **Incident ID** | INC-2026-0206-001 |
| **Date & Time** | 6th February 2026, 13:47 IST |
| **Severity** | Medium |
| **Attack Type** | Brute-Force Authentication |
| **Target System** | Windows 10 (10.232.194.141) |
| **Targeted Account** | testuser |
| **Failed Attempts** | 5+ |
| **Success Rate** | 0% (All attempts blocked) |
| **Event ID** | 4625 (Failed Logon) |
| **Logon Type** | Type 2 (Interactive) |

### Investigation Summary

- **Detection Time:** < 1 minute (real-time alert)
- **Failure Reason:** "Unknown user name or bad password"
- **Source Computer:** NATIONALIST
- **Impact:** None - all attempts failed
- **Status:** Detected and documented

---

## 🎯 MITRE ATT&CK Mapping

| MITRE Component | Details |
|-----------------|---------|
| **Tactic** | Credential Access (TA0006) |
| **Technique** | Brute Force (T1110) |
| **Sub-technique** | Password Guessing (T1110.001) |
| **Detection** | Monitor authentication logs for multiple failed attempts |

**Reference:** [MITRE ATT&CK - Brute Force](https://attack.mitre.org/techniques/T1110/)

---

## 📦 Deliverables

This repository contains:

1. **Incident Report** - Professional Word document with findings (`docs/SOC_Brute_Force_Incident_Report.docx`)
2. **Detection Queries** - SPL queries used for detection (`queries/detection_query.txt`)
3. **Screenshots** - Visual evidence of setup and detection (`screenshots/`)
4. **Setup Guide** - Detailed installation instructions (`setup/setup_guide.md`)
5. **README** - Complete project documentation (this file)

---

## 📸 Screenshots

### Splunk Login Page
![Splunk Login](screenshots/splunk_login.png)

### Failed Login Events Detection
![Failed Events](screenshots/failed_login_events.png)

### Detection Query Results
![Search Results](screenshots/splunk_search_results.png)

### Alert Configuration
![Alert Setup](screenshots/alert_configuration.png)

---

## 💡 Lessons Learned

### Technical Skills Gained

✅ **SIEM Deployment:** Hands-on experience with Splunk Enterprise installation and configuration  
✅ **Log Forwarding:** Configured Splunk Universal Forwarder for centralized log collection  
✅ **SPL Queries:** Developed Search Processing Language queries for threat detection  
✅ **Windows Event Logs:** Deep understanding of Event ID 4625 and authentication logging  
✅ **Alert Creation:** Built real-time alerts with custom thresholds  
✅ **Incident Response:** Followed SOC workflow from detection to documentation  
✅ **MITRE ATT&CK:** Mapped real attacks to industry-standard framework  

### Challenges Faced

**Challenge 1: Network Connectivity**
- **Issue:** Initial NAT networking prevented Windows-to-Kali communication
- **Solution:** Changed VirtualBox network adapter to Bridged mode

**Challenge 2: Log Forwarding Configuration**
- **Issue:** Universal Forwarder not sending logs initially
- **Solution:** Manually edited `inputs.conf` and restarted forwarder service

**Challenge 3: Detection Query Optimization**
- **Issue:** Initial query returned too many results
- **Solution:** Added threshold filter (`where count >= 3`) to reduce false positives

---

## 🚀 Future Enhancements

Potential improvements for this project:

1. **Automated Response:** Implement automatic account lockout after threshold breach
2. **Geolocation Analysis:** Add IP geolocation lookup for source identification
3. **Dashboard Creation:** Build Splunk dashboard with visualizations (charts, graphs, heatmaps)
4. **Multiple Attack Types:** Expand to detect other attacks (privilege escalation, lateral movement)
5. **Email Alerting:** Configure email notifications for critical alerts
6. **Threat Intelligence Integration:** Integrate with threat intel feeds (AbuseIPDB, VirusTotal)
7. **Correlation Rules:** Create multi-stage detection rules (e.g., failed login → successful login)
8. **Linux Log Analysis:** Add SSH brute-force detection from Linux auth logs

---

## 👤 Author

**Niladri Biswas**  
M.Tech Information Security (Pursuing)  
West Bengal University of Technology (MAKAUT)

- 📧 Email: dr.niladribiswas@gmail.com
- 💼 LinkedIn: [linkedin.com/in/dr-niladri-biswas](https://linkedin.com/in/dr-niladri-biswas)
- 🐙 GitHub: [github.com/oracleo](https://github.com/oracleo)
- 🎯 TryHackMe: [tryhackme.com/p/dr.nee](https://tryhackme.com/p/dr.nee)

---

## 📝 License

This project is for educational purposes only. Do not use these techniques on systems you do not own or have explicit permission to test.

---

## 🙏 Acknowledgments

- Splunk Community for documentation and resources
- MITRE ATT&CK for the threat framework
- TryHackMe for SOC training pathway
- Security community for best practices

---

**⭐ If you found this project helpful, please star the repository!**

---

*Last Updated: 6th February 2026*
