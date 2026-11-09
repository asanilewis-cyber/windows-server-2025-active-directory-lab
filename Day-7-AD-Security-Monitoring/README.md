# Day 7 — Active Directory Security Monitoring & Incident Investigation

## 1. Project Overview

This hands-on cybersecurity lab focused on investigating Active Directory account lockouts, analyzing Windows Security logs, identifying missing authentication failure events, and correcting audit policy settings.

The investigation was conducted in an isolated Windows Server 2025 and Windows 11 environment using Oracle VirtualBox.

**Objective:** Identify a security monitoring gap, correct the configuration, generate a controlled authentication failure, verify the resulting security event, and preserve the evidence.

## 2. Lab Environment

| Component | Configuration |
|---|---|
| Domain Controller | DC01 |
| Server Operating System | Windows Server 2025 |
| Client Computer | CLIENT01 |
| Client Operating System | Windows 11 |
| Active Directory Domain | adlab.test |
| Client IP Address | 10.0.2.100 |
| Virtualization | Oracle VirtualBox |
| Tools | PowerShell, Event Viewer, auditpol, Get-WinEvent |
| Test Account | test.lockout |

## 3. Investigation Objectives

- Investigate Active Directory account lockouts.
- Analyze Windows Security Event IDs 4625, 4740, 4771, and 4776.
- Identify affected accounts and originating computers.
- Examine authentication failure codes.
- Review advanced Windows audit policy settings.
- Enable missing failure auditing.
- Generate a controlled failed authentication attempt.
- Verify that the domain controller records the event.
- Export investigation evidence.

## 4. Initial Account Lockout Investigation

Windows Security Event ID 4740 indicates that a user account was locked out.

Using Event Viewer and PowerShell on DC01, I identified two previously recorded account lockouts.

| Account | Date and Time | Caller Computer |
|---|---|---|
| jsmith | October 28, 2026, 11:40:18 AM | CLIENT01 |
| test.lockout | October 28, 2026, 12:26:35 PM | CLIENT01 |

PowerShell command:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4740
} -MaxEvents 20 |
Select-Object TimeCreated, Message |
Format-List
```

**Finding:** Both lockout events identified CLIENT01 as the caller computer. These records established where to begin investigating but did not independently establish the complete sequence of failed authentication attempts.

## 5. Authentication Event Investigation

The following Windows Security event IDs were investigated:

| Event ID | Description |
|---|---|
| 4625 | An account failed to log on |
| 4740 | A user account was locked out |
| 4771 | Kerberos pre-authentication failed |
| 4776 | The computer attempted to validate credentials |

I searched DC01 for Events 4625, 4771, and 4776 during the October 28 lockout investigation window.

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625,4740
    StartTime = '2026-10-28 11:30:00'
    EndTime = '2026-10-28 12:40:00'
} |
Sort-Object TimeCreated |
Select-Object TimeCreated, Id |
Format-Table -AutoSize
```

The query returned the two Event 4740 lockout records, but no Event 4625 records in that window.

Separate searches for Events 4771 and 4776 also returned no matching records for the same time window.

**Finding:** The account lockouts were confirmed, but supporting authentication failure records were unavailable in the queried DC01 Security log for that period.

## 6. Audit Policy Investigation

I checked the domain controller's advanced audit policy configuration.

```powershell
auditpol /get /category:"Account Logon"
```

Initial results:

| Audit Subcategory | Initial Setting |
|---|---|
| Kerberos Service Ticket Operations | Success |
| Other Account Logon Events | No Auditing |
| Kerberos Authentication Service | Success |
| Credential Validation | Success |

The Kerberos Authentication Service and Credential Validation subcategories were not configured to audit failures.

I also checked:

```powershell
auditpol /get /category:"Logon/Logoff"
```

The Logon subcategory was already configured for Success and Failure, while Account Lockout was configured for Success.

**Finding:** Missing failure auditing in the Account Logon category represented a security monitoring gap.

## 7. Corrective Action — Enable Failure Auditing

On DC01, I enabled success and failure auditing for Kerberos authentication and credential validation.

```powershell
auditpol /set /subcategory:"Kerberos Authentication Service" /success:enable /failure:enable
```

```powershell
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable
```

Both commands completed successfully.

I then verified the settings:

```powershell
auditpol /get /category:"Account Logon"
```

Verified results:

| Audit Subcategory | Final Setting |
|---|---|
| Kerberos Service Ticket Operations | Success |
| Other Account Logon Events | No Auditing |
| Kerberos Authentication Service | Success and Failure |
| Credential Validation | Success and Failure |

**Result:** The targeted audit policy changes were successfully applied and verified on DC01.

## 8. Controlled Authentication Failure Test

To test the updated auditing configuration, I used the disposable Active Directory account `test.lockout` on CLIENT01.

Command executed:

```powershell
runas /user:ADLAB\test.lockout cmd
```

I deliberately entered an incorrect password once.

Windows returned:

```text
RUNAS ERROR: Unable to run - cmd
1326: The user name or password is incorrect.
```

This was an authorized, controlled test performed within my isolated lab environment.

## 9. Kerberos Event Verification

After the failed authentication attempt, I searched DC01 for recent security events.

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4771,4776,4625
    StartTime = (Get-Date).AddMinutes(-30)
} -ErrorAction SilentlyContinue |
Select-Object TimeCreated, Id |
Format-Table -AutoSize
```

The query returned Event ID 4771.

I examined its details:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4771
    StartTime = (Get-Date).AddMinutes(-30)
} -MaxEvents 1 |
Format-List TimeCreated, Id, Message
```

### Verified Security Event

| Field | Value |
|---|---|
| Event ID | 4771 |
| Timestamp | November 9, 2026, 7:52:08 AM |
| Account | test.lockout |
| Client Address | 10.0.2.100 |
| Domain Controller | DC01.adlab.test |
| Authentication | Kerberos |
| Failure Code | 0x18 |
| Meaning | Kerberos pre-authentication failed due to an incorrect password |

The client address appeared in the event as `::ffff:10.0.2.100`, an IPv4-mapped IPv6 address.

**Result:** The controlled failed password attempt generated a Kerberos authentication failure record on DC01.

This confirmed that the updated Kerberos failure auditing configuration was capturing the expected event.

## 10. Security Evidence Export

I created a dedicated investigation directory on DC01.

```powershell
New-Item -Path "C:\SecurityLab" -ItemType Directory -Force
```

I exported the recent Event 4771 evidence.

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4771
    StartTime = (Get-Date).AddDays(-1)
} |
Format-List TimeCreated, Id, MachineName, Message |
Out-File "C:\SecurityLab\Day7-Kerberos-Investigation.txt"
```

I verified the exported file:

```powershell
Get-Item "C:\SecurityLab\Day7-Kerberos-Investigation.txt" |
Select-Object Name, Length, LastWriteTime
```

Verification results:

| Property | Value |
|---|---|
| File | Day7-Kerberos-Investigation.txt |
| Size | 2,680 bytes |
| Last Modified | November 9, 2026, 7:58 AM |
| Location | C:\SecurityLab |

**Result:** The investigation evidence was successfully exported and its existence verified.

## 11. Investigation Findings

| Investigation Area | Finding |
|---|---|
| Account lockout investigation | Two Event 4740 records identified |
| Source workstation | CLIENT01 |
| Historical Event 4625 search | No matches in selected window |
| Historical Event 4771 search | No matches in selected window |
| Historical Event 4776 search | No matches in selected window |
| Kerberos failure auditing | Initially not enabled |
| Credential validation failure auditing | Initially not enabled |
| Audit policy correction | Successful |
| Controlled failed authentication | Successful test |
| New Event 4771 | Captured |
| Incorrect-password failure code | 0x18 |
| Evidence export | Verified |

## 12. Root Cause and Resolution

### Monitoring Gap

The initial audit policy inspection showed that Kerberos Authentication Service and Credential Validation were configured to record successful activity only.

This limited the domain controller's visibility into certain authentication failures.

### Resolution

I enabled failure auditing for both subcategories using `auditpol`.

A subsequent controlled failed password attempt generated Event ID 4771, confirming that Kerberos authentication failure logging was operational.

### Investigation Limitation

The original October 28 account lockout events were confirmed, but the available logs did not establish the full authentication failure sequence that caused them.

The new November 9 test verified the auditing correction. It did not retroactively recreate historical events.

## 13. Security Principles Demonstrated

**Security Monitoring:** Recording authentication activity provides evidence for troubleshooting and security investigations.

**Authentication:** Kerberos verifies user credentials before granting access to domain resources.

**Incident Investigation:** Event IDs, timestamps, usernames, source addresses, and failure codes help analysts understand authentication activity.

**Audit Policy Management:** Security logging settings must be checked and verified rather than assumed.

**Evidence Preservation:** Exporting security events supports investigation documentation and later review.

**Validation:** Security configuration changes should be followed by controlled testing to confirm their effectiveness.

## 14. Skills Demonstrated

- Windows Server 2025 administration
- Active Directory security monitoring
- Windows Event Viewer
- Windows Security Event analysis
- Kerberos authentication troubleshooting
- PowerShell investigation commands
- Get-WinEvent filtering
- Advanced Audit Policy configuration
- Authentication failure analysis
- Account lockout investigation
- Security control validation
- Evidence export and documentation
- SOC analyst investigation fundamentals

## 15. Interview Explanation

"In my Windows Server 2025 Active Directory home lab, I investigated account lockout events and discovered that certain authentication failure auditing settings were not enabled.

I reviewed Windows Security Event IDs, checked the domain controller's advanced audit policies, and enabled failure auditing for Kerberos authentication and credential validation.

To verify the correction, I generated a controlled failed login from a domain-joined Windows 11 computer. I then identified Event ID 4771, confirmed the affected account, source IP address, and incorrect-password failure code, and exported the evidence using PowerShell.

The exercise strengthened my skills in Windows Server administration, security monitoring, troubleshooting, and technical documentation."

## 16. Conclusion

This lab demonstrated a complete security monitoring improvement workflow:

**Investigate → Identify Gap → Correct Configuration → Test → Verify → Document**

The work was performed in an isolated personal training environment using Windows Server 2025, Windows 11, Active Directory, and Oracle VirtualBox.
## 17. Screenshot Evidence

The following screenshots document the Day 7 Active Directory security monitoring and incident investigation lab.

| # | Investigation Evidence | Screenshot |
|---|---|---|
| 1 | Failed logon Event 4625 analysis | [View](screenshots/01-failed-logon-event-4625-analysis.png) |
| 2 | Failed logon source IP investigation | [View](screenshots/02-failed-logon-source-ip-analysis.png) |
| 3 | Account lockout Event 4740 summary | [View](screenshots/03-account-lockout-event-4740-summary.png) |
| 4 | Account lockout Event 4740 details | [View](screenshots/04-account-lockout-event-4740-details.png) |
| 5 | Account lockout timeline | [View](screenshots/05-lockout-event-timeline.png) |
| 6 | Audit policy before changes | [View](screenshots/06-audit-policy-before-changes.png) |
| 7 | Logon audit policy review | [View](screenshots/07-logon-audit-policy-review.png) |
| 8 | Audit policy after changes | [View](screenshots/08-audit-policy-after-changes.png) |
| 9 | Kerberos Event 4771 discovery | [View](screenshots/09-kerberos-event-4771-discovery.png) |
| 10 | Kerberos pre-authentication failure details | [View](screenshots/10-kerberos-preauthentication-failure-details.png) |
| 11 | Kerberos event verification on DC01 | [View](screenshots/11-kerberos-event-dc01-verification.png) |
| 12 | Security evidence directory creation | [View](screenshots/12-security-evidence-directory.png) |
| 13 | Kerberos evidence export command | [View](screenshots/13-kerberos-evidence-export-command.png) |
| 14 | Kerberos evidence export verification | [View](screenshots/14-kerberos-evidence-export-verification.png) |

No production systems or real-world security incidents were involved.
