# 6. Remediation & Treatment Plan Tracker

This artifact tracks the remediation plan with a **Cost-Benefit Analysis** to justify security spending to management.

## Cost-Benefit Context
*   **Cost of Inaction (Breach):** A successful brute-force attack against a privileged account could lead to a ransomware incident or BEC (Business Email Compromise), costing an organization an average of **$100,000 - $500,000+** in downtime, data recovery, and legal fees.
*   **Cost of Remediation:** Implementing an Account Lockout policy and MFA for privileged users are configuration changes. The cost is usually **< $500** in engineering time.

| Priority | Finding | Remediation Action | Assigned To | Due Date | Remediation Cost (Effort) | Cost of Inaction | Status |
|:---:|:---|:---|:---|:---:|:---|:---|:---|
| **P1** | **Absent Account Lockout (Preventive Gap)** | 1. Implement Local Group Policy: `5` invalid attempts within `15` minutes.<br>2. Account lockout duration: `30` minutes. | Windows Admin | **Immediate** | **~$200** (1 Engineer hour) | **~$100,000+** (Ransomware/Breach) | In Progress |
| **P2** | **Absent MFA on Privileged Accounts (`Dr.Nee`)** | 1. Enroll Admin accounts in an MFA solution (e.g., Duo, Microsoft Authenticator).<br>2. Enforce Conditional Access policies. | IAM Admin | 7 Days | **~$300** (Setup + licensing setup) | **~$500,000** (Full domain takeover) | Not Started |
| **P2** | **Alert Trigger Notifications** | 1. Integrate Splunk alert with Email or Slack.<br>2. Connect to SOAR system (e.g., Splunk Phantom) for automated IP blocking. | SOC / Security Engineer | 30 Days | **~$100** (Webhook/Email config) | **~$50,000** (Incident response delays) | Not Started |

## Budget Justification Summary
The **P1** and **P2** remediation actions represent a total estimated investment of **~$600**. Failure to implement these controls could result in a **$100,000+** breach. 
**GRC Recommendation:** Immediate funding is requested for the Group Policy update and the MFA implementation to reduce the overall authentication risk to an acceptable residual level.
