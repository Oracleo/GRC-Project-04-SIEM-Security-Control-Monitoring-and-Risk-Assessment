# Complete Setup Guide - Mini SOC Lab

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Phase 1: Splunk Installation on Kali Linux](#phase-1-splunk-installation-on-kali-linux)
3. [Phase 2: Network Configuration](#phase-2-network-configuration)
4. [Phase 3: Splunk Universal Forwarder on Windows](#phase-3-splunk-universal-forwarder-on-windows)
5. [Phase 4: Log Collection Setup](#phase-4-log-collection-setup)
6. [Phase 5: Attack Simulation](#phase-5-attack-simulation)
7. [Phase 6: Detection & Analysis](#phase-6-detection--analysis)
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Hardware Requirements

**Kali Linux VM:**
- RAM: 4 GB minimum (6 GB recommended)
- Storage: 30 GB
- Processor: 2 cores minimum

**Windows 10:**
- RAM: 4 GB minimum
- Storage: 10 GB free space

### Software Requirements

- VirtualBox (latest version)
- Kali Linux ISO or pre-built VM
- Windows 10 (host or VM)
- Splunk account (free) - create at https://www.splunk.com

### Network Requirements

- Both systems must be on the same network
- Ports required:
  - 8000 (Splunk Web)
  - 8089 (Splunk Management)
  - 9997 (Splunk Receiving)

---

## Phase 1: Splunk Installation on Kali Linux

### Step 1.1: Download Splunk Enterprise

1. Open browser on Kali Linux
2. Go to: https://www.splunk.com/en_us/download/splunk-enterprise.html
3. Sign in or create free account
4. Select: **Linux** tab
5. Choose: **.deb** package (for Debian/Kali)
6. Click **Download Now**
7. Save to: `~/Downloads`

**Expected file:** `splunk-10.2.0-<build>-linux-amd64.deb` (~1.3 GB)

### Step 1.2: Install Splunk

Open terminal and run:

```bash
# Navigate to downloads
cd ~/Downloads

# Verify file exists
ls -lh splunk*.deb

# Install Splunk (enter sudo password when prompted)
sudo dpkg -i splunk-10.2.0-*.deb
```

**Expected output:**
```
Selecting previously unselected package splunk.
Unpacking splunk...
Setting up splunk...
complete
```

### Step 1.3: Start Splunk for First Time

```bash
# Start Splunk and accept license
sudo /opt/splunk/bin/splunk start --accept-license
```

**You will be prompted to create admin credentials:**

```
Please enter an administrator username: admin
Password must contain at least 8 characters.
Please enter a new password: [enter password]
Please confirm new password: [re-enter password]
```

**IMPORTANT:** Remember these credentials - you'll need them to log in!

**Installation takes 2-3 minutes. Wait for:**

```
Splunk> Take the sh out of IT.

All preliminary checks passed.
Starting splunk server daemon (splunkd)...
Done

The Splunk web interface is at http://kali:8000
```

### Step 1.4: Access Splunk Web Interface

1. Open Firefox in Kali
2. Go to: `http://127.0.0.1:8000`
3. Login with credentials created in Step 1.3
4. You should see Splunk home page

✅ **Checkpoint:** Splunk is installed and accessible!

---

## Phase 2: Network Configuration

### Step 2.1: Configure Kali VM Network

1. **Shut down Kali VM** (if running)
2. In VirtualBox Manager:
   - Right-click Kali VM → **Settings**
   - Go to **Network** tab
   - **Adapter 1:**
     - Attached to: **Bridged Adapter**
     - Name: Select your physical network adapter
   - Click **OK**
3. **Start Kali VM**

### Step 2.2: Find Kali IP Address

In Kali terminal:

```bash
ip addr show
```

Look for `inet` under `eth0`:

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 10.232.194.7/24 brd 10.232.194.255
```

**Your Kali IP:** `10.232.194.7` (example - yours will be different)

**📝 WRITE DOWN THIS IP - You'll need it later!**

### Step 2.3: Test Connectivity from Windows

On Windows, open **Command Prompt**:

```cmd
ping 10.232.194.7
```

**Expected output:**
```
Reply from 10.232.194.7: bytes=32 time<1ms TTL=64
Reply from 10.232.194.7: bytes=32 time<1ms TTL=64
```

✅ **Checkpoint:** Windows can reach Kali!

### Step 2.4: Configure Splunk Receiving Port

1. In Kali, open Splunk web interface: `http://127.0.0.1:8000`
2. Click **Settings** (top menu) → **Forwarding and receiving**
3. Under **Receive data**, click **Configure receiving**
4. Click **+ Add new**
5. Enter port: `9997`
6. Click **Save**

✅ **Checkpoint:** Splunk is listening on port 9997!

---

## Phase 3: Splunk Universal Forwarder on Windows

### Step 3.1: Download Universal Forwarder

1. On Windows, open browser
2. Go to: https://www.splunk.com/en_us/download/universal-forwarder.html
3. Sign in with same Splunk account
4. Select: **Windows** tab
5. Choose: **64-bit** installer
6. Click **Download Now**
7. Save to Downloads

**Expected file:** `splunkforwarder-10.2.0-<build>-x64-release.msi`

### Step 3.2: Install Universal Forwarder

1. Run the downloaded `.msi` file
2. Click **Next** through installation wizard
3. Accept license agreement
4. Choose installation directory (default: `C:\Program Files\SplunkUniversalForwarder`)
5. **Create deployment user** (or skip)
6. Click **Install**
7. Click **Finish**

### Step 3.3: Configure Forwarder

Open **Command Prompt as Administrator:**

```cmd
# Navigate to Splunk bin directory
cd "C:\Program Files\SplunkUniversalForwarder\bin"

# Start forwarder for first time
splunk.exe start --accept-license
```

**Create admin credentials when prompted:**

```
Please enter an administrator username: drnee
Password: [enter password]
Confirm password: [re-enter]
```

**Wait for:**
```
SplunkForwarder: Starting (pid 1234)
Done
```

### Step 3.4: Add Forward Server

**IMPORTANT:** Replace `10.232.194.7` with YOUR Kali IP from Step 2.2!

```cmd
splunk.exe add forward-server 10.232.194.7:9997
```

**When prompted for credentials, enter the ones you just created**

**Expected output:**
```
Added forwarding to: 10.232.194.7:9997.
```

✅ **Checkpoint:** Forwarder is configured to send logs to Kali!

---

## Phase 4: Log Collection Setup

### Step 4.1: Configure Windows Security Log Input

```cmd
# Navigate to config directory
cd "C:\Program Files\SplunkUniversalForwarder\etc\system\local"

# Open inputs.conf in Notepad
notepad inputs.conf
```

**If file doesn't exist, Notepad will ask to create it - click YES**

**Add these lines:**

```ini
[WinEventLog://Security]
disabled = 0
index = main
```

**Save and close Notepad**

### Step 4.2: Restart Forwarder

```cmd
# Navigate back to bin directory
cd "C:\Program Files\SplunkUniversalForwarder\bin"

# Restart forwarder
splunk.exe restart
```

**Expected output:**
```
SplunkForwarder: Stopped
SplunkForwarder: Starting (pid 5678)
Done
```

### Step 4.3: Enable Windows Auditing

Still in Command Prompt (Admin):

```cmd
# Enable logon auditing
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```

**Expected output:**
```
The command was successfully executed.
```

### Step 4.4: Verify Logs in Splunk

1. Go to Kali → Splunk web interface
2. Click **Search & Reporting** (left sidebar)
3. In search bar, enter:

```spl
index=main sourcetype=WinEventLog:Security
```

4. Set time range: **Last 15 minutes**
5. Click **Search** (green magnifying glass)

**You should see Windows Security events appearing!**

✅ **Checkpoint:** Windows logs are flowing to Splunk!

---

## Phase 5: Attack Simulation

### Step 5.1: Create Test User Account

On Windows (Command Prompt as Admin):

```cmd
net user testuser Test@123 /add
```

**Expected output:**
```
The command completed successfully.
```

### Step 5.2: Generate Failed Login Attempts

In Windows (regular Command Prompt - not admin):

```cmd
runas /user:testuser cmd
```

**When prompted for password, enter WRONG passwords:**

1. Try: `wrongpass1` → Press Enter
2. Try: `wrongpass2` → Press Enter
3. Try: `wrongpass3` → Press Enter
4. Try: `wrongpass4` → Press Enter
5. Try: `wrongpass5` → Press Enter

**Each attempt will show:**
```
RUNAS ERROR: Unable to run - cmd
1326: The user name or password is incorrect.
```

**This is expected! You're generating failed login events.**

✅ **Checkpoint:** 5 failed login events generated!

---

## Phase 6: Detection & Analysis

### Step 6.1: Search for Failed Logins

In Splunk:

```spl
index=main sourcetype=WinEventLog:Security EventCode=4625
```

**You should see 5+ events for failed logins**

### Step 6.2: Run Detection Query

```spl
index=main sourcetype=WinEventLog:Security EventCode=4625
| stats count by Account_Name
| where count >= 3
| sort - count
```

**Expected result:**

| Account_Name | count |
|--------------|-------|
| testuser     | 5     |

✅ **Checkpoint:** Brute-force attack detected!

### Step 6.3: Create Alert

1. In search results, click **Save As** → **Alert**
2. Fill in:
   - **Title:** `Brute Force Detection - Failed Login Attempts`
   - **Description:** `Alert triggers when 3+ failed login attempts detected`
   - **Alert type:** Real-time
3. Under **Trigger Actions**, click **+ Add Actions**
4. Select **Log Event**
5. Enter event text: `Brute Force Alert: User $result.Account_Name$ has $result.count$ failed login attempts`
6. Click **Save**

✅ **Checkpoint:** Alert created!

---

## Troubleshooting

### Issue: Splunk Web Interface Not Accessible

**Problem:** `http://127.0.0.1:8000` doesn't load

**Solution:**
```bash
# Check if Splunk is running
sudo /opt/splunk/bin/splunk status

# If not running, start it
sudo /opt/splunk/bin/splunk start
```

### Issue: Windows Can't Ping Kali

**Problem:** `ping 10.232.194.7` shows "Request timed out"

**Solution:**
1. Verify Kali VM network is set to **Bridged**
2. Check firewall on Kali:
```bash
sudo ufw status
# If active, allow ICMP
sudo ufw allow from any to any
```

### Issue: No Windows Logs in Splunk

**Problem:** Search returns 0 results

**Solution:**
1. Verify forwarder is running on Windows:
```cmd
cd "C:\Program Files\SplunkUniversalForwarder\bin"
splunk.exe status
```

2. Check inputs.conf exists and has correct content:
```cmd
type "C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf"
```

3. Restart forwarder:
```cmd
splunk.exe restart
```

### Issue: EventCode 4625 Not Showing

**Problem:** Search shows events but no EventCode=4625

**Solution:**
Verify Windows auditing is enabled:
```cmd
auditpol /get /subcategory:"Logon"
```

Should show:
```
Logon    Success and Failure
```

If not, enable it:
```cmd
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```

### Issue: Alert Not Triggering

**Problem:** Alert created but never fires

**Solution:**
1. Check alert is enabled (green checkmark)
2. Verify search returns results when run manually
3. Check alert trigger conditions (threshold set correctly)
4. Review Splunk internal logs:
```spl
index=_internal source=*scheduler* alert_actions
```

---

## Verification Checklist

Before proceeding, ensure all these are complete:

- [ ] Splunk Enterprise installed on Kali
- [ ] Splunk accessible at http://127.0.0.1:8000
- [ ] Kali VM on Bridged network
- [ ] Windows can ping Kali IP
- [ ] Splunk receiving port 9997 configured
- [ ] Universal Forwarder installed on Windows
- [ ] Forward server configured (Kali IP:9997)
- [ ] inputs.conf created with Security log monitor
- [ ] Windows auditing enabled
- [ ] Windows Security logs visible in Splunk
- [ ] Test user account created
- [ ] Failed login events generated
- [ ] Detection query returns results
- [ ] Alert created and enabled

---

## Next Steps

After completing this setup:

1. **Documentation:** Create incident report (see `/docs/`)
2. **Screenshots:** Capture evidence for GitHub
3. **GitHub:** Push project to repository
4. **Portfolio:** Add project to CV and LinkedIn

---

## Support

If you encounter issues not covered in troubleshooting:

1. Check Splunk logs: `index=_internal`
2. Review Splunk documentation: https://docs.splunk.com
3. Kali Linux forums: https://forums.kali.org
4. Windows Event Log documentation

---

*Created by: Niladri Biswas*  
*Last Updated: 6th February 2026*
