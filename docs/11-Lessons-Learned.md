# 11. Lessons Learned

This document serves as a **post-implementation review (PIR)** of the Splunk SIEM control monitoring lab. In a professional GRC environment, documenting lessons learned is essential for demonstrating a "Continuous Improvement" mindset required by ISO 27001 Clause 10.2.

## 11.1 What Went Well (Successes)
*   **Authentic Attack Simulation:** Using the native `runas` command with incorrect credentials generated authentic Windows Event ID 4625 entries. This validated the log forwarding pipeline from the Windows endpoint to the Splunk indexer with zero false positives.
*   **Threshold Logic Implementation:** The SPL query using `stats count by Account_Name | where count >= 3` proved highly effective. It successfully flagged both the privileged (`Dr.Nee`) and standard (`testuser`) accounts immediately upon crossing the 3-attempt threshold.
*   **Rapid Deployment:** The entire lab environment (Splunk Enterprise, Universal Forwarder, Bridged Network) was established and operational within 45 minutes.

## 11.2 What Could Be Improved (Challenges)
*   **Alert Routing Delay:** Currently, the Alert simply logs an event to the Splunk internal index. To be truly operational, it needs to be integrated with **Email, Slack, or a SOAR platform** (e.g., Splunk Phantom or Sentinel) immediately.
*   **Missing Source IP Context:** The initial search query grouped failures *only* by `Account_Name`. In a real SOC environment, correlating by `src_ip` is critical to identifying distributed brute-force attacks (multiple accounts targeted from a single machine). 
*   **Reliance on a 60-Day Trial:** The Splunk Enterprise trial license will expire after 60 days. For a proof-of-concept, this is acceptable, but it limits the ability to demonstrate long-term continuous compliance.

## 11.3 Next Iteration Improvements (The "V2" Plan)
If this project were repeated or expanded, the following changes would be prioritized:
1.  **Expand Log Sources:** Incorporate `Microsoft-Windows-Sysmon/Operational` logs to capture more granular process creation events alongside authentication failures.
2.  **Enhance Correlation:** Modify the SPL query to include `src_ip`, `Logon_Type`, and `ComputerName` in the `stats` aggregation to map distributed brute-force patterns.
3.  **Automate Incident Response:** Implement a SOAR playbook to automatically block the offending `src_ip` at the firewall upon the first alert trigger.

## 11.4 Final Reflection (GRC Perspective)
This project demonstrated the critical difference between **design** and **operationalization** of a control. While the SIEM was configured correctly and the SPL logic was sound, the *absence* of Account Lockout and MFA was a glaring warning. 

> *"As a GRC analyst, my biggest takeaway is this: A fully operational *detective* control on a vulnerable *preventive* control simply alerts you to your failure. True security maturity happens when preventive and detective controls work in tandem."*
