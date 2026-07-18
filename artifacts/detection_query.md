# Brute-Force Detection Query (SPL)

## Primary Detection Query

### Query Name: Brute-Force Failed Login Detection

**Description:**
This query identifies potential brute-force authentication attacks by counting failed login attempts per user account and filtering for accounts with 3 or more failures.

**Event ID:** 4625 (An account failed to log on)

**SPL Query:**
```spl
index=main sourcetype=WinEventLog:Security EventCode=4625
| stats count by Account_Name
| where count >= 3
| sort - count
```

**Query Components:**

1. **index=main** 
   - Searches the "main" index where Windows Security logs are stored

2. **sourcetype=WinEventLog:Security**
   - Filters for Windows Security Event Logs

3. **EventCode=4625**
   - Event ID 4625 = "An account failed to log on"
   - This captures all failed authentication attempts

4. **| stats count by Account_Name**
   - Aggregates failed attempts by username
   - Creates a count of failures for each account

5. **| where count >= 3**
   - Filters results to show only accounts with 3 or more failures
   - This threshold indicates potential brute-force activity

6. **| sort - count**
   - Sorts results in descending order (highest failures first)
   - Helps prioritize investigation

---

## Alternative Queries

### Query 2: Time-Based Brute-Force Detection

**Description:** Detects brute-force attempts within a specific time window

```spl
index=main sourcetype=WinEventLog:Security EventCode=4625
| bucket _time span=5m
| stats count by _time, Account_Name
| where count >= 3
| sort - count
```

**Use Case:** Identifies rapid-fire brute-force attacks within 5-minute windows

---

### Query 3: Failed Login with Source IP

**Description:** Identifies brute-force attempts and shows source IP addresses

```spl
index=main sourcetype=WinEventLog:Security EventCode=4625
| stats count by Account_Name, src_ip
| where count >= 3
| sort - count
```

**Use Case:** Helps identify distributed attacks from multiple IPs or single-source attacks

---

### Query 4: Failed Login Timeline Visualization

**Description:** Creates timeline of failed login attempts

```spl
index=main sourcetype=WinEventLog:Security EventCode=4625 Account_Name=*
| timechart count by Account_Name
```

**Use Case:** Visualizes attack patterns over time for dashboard creation

---

### Query 5: Successful Login After Failed Attempts

**Description:** Detects potential successful compromise after brute-force

```spl
index=main sourcetype=WinEventLog:Security (EventCode=4625 OR EventCode=4624) Account_Name=testuser
| transaction Account_Name maxspan=10m
| search eventcount>3
| where EventCode=4624
```

**Use Case:** Critical alert - indicates attacker may have gained access after brute-force

**Event IDs:**
- 4625 = Failed logon
- 4624 = Successful logon

---

## Query Usage in Alert Configuration

### Real-Time Alert Setup

**Alert Name:** Brute Force Detection - Failed Login Attempts

**Search Query:**
```spl
index=main sourcetype=WinEventLog:Security EventCode=4625
| stats count by Account_Name
| where count >= 3
| sort - count
```

**Alert Settings:**
- **Alert Type:** Real-time
- **Trigger Condition:** Per-Result
- **Trigger When:** Number of Results is greater than 0
- **Throttle:** Suppress triggering for 5 minutes (prevents alert spam)

**Alert Actions:**
- Log Event: "Brute Force Alert: User $result.Account_Name$ has $result.count$ failed login attempts"
- (Optional) Send Email to SOC team
- (Optional) Trigger webhook for SOAR integration

---

## Field Reference

Common fields in Event ID 4625:

| Field Name | Description | Example Value |
|------------|-------------|---------------|
| EventCode | Windows Event ID | 4625 |
| Account_Name | Target username | testuser |
| Failure_Reason | Why login failed | "Unknown user name or bad password" |
| Logon_Type | Type of logon attempt | 2 (Interactive), 3 (Network), 10 (RDP) |
| Source_Network_Address | IP of attacker | 10.232.194.7 |
| Workstation_Name | Computer name | NATIONALIST |
| ComputerName | Target system | NATIONALIST |

---

## Threshold Tuning

### Recommended Thresholds

| Environment | Threshold | Timespan | Rationale |
|-------------|-----------|----------|-----------|
| **Production** | 5 failures | 15 minutes | Reduces false positives from users forgetting passwords |
| **Lab/Testing** | 3 failures | Any time | More sensitive for demonstration |
| **High-Security** | 3 failures | 5 minutes | Maximum sensitivity for critical systems |
| **Executive Accounts** | 2 failures | Any time | VIP protection - lower threshold |

### Adjusting Thresholds

To change the failure threshold, modify the `where` clause:

```spl
| where count >= 5    # More lenient (fewer false positives)
| where count >= 2    # More strict (higher sensitivity)
```

---

## False Positive Reduction

### Exclusions

Exclude known service accounts or expected failures:

```spl
index=main sourcetype=WinEventLog:Security EventCode=4625
| search NOT Account_Name IN ("ServiceAccount1", "BackupUser", "MonitoringAgent")
| stats count by Account_Name
| where count >= 3
| sort - count
```

### Whitelist Legitimate IPs

Exclude internal scanning tools or known admin IPs:

```spl
index=main sourcetype=WinEventLog:Security EventCode=4625
| search NOT src_ip IN ("10.0.0.50", "192.168.1.100")
| stats count by Account_Name, src_ip
| where count >= 3
```

---

## Dashboard Query

For creating a Splunk dashboard panel:

```spl
index=main sourcetype=WinEventLog:Security EventCode=4625
| timechart span=1h count by Account_Name limit=10
```

**Visualization:** Line chart showing failed login trends per hour

---

## Notes

- **Event ID 4625** is logged only if auditing is enabled for "Logon/Logoff" events
- Enable auditing: `auditpol /set /subcategory:"Logon" /success:enable /failure:enable`
- Logs are stored in Windows Security Event Log: `C:\Windows\System32\winevt\Logs\Security.evtx`
- Real-time detection requires Splunk Universal Forwarder configured with `[WinEventLog://Security]`

---

## References

- [Microsoft Event ID 4625 Documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)
- [Splunk SPL Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference)
- [MITRE ATT&CK T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/)

---

*Created by: Niladri Biswas*  
*Date: 6th February 2026*  
*Project: Mini SOC Lab - Brute-Force Detection*
