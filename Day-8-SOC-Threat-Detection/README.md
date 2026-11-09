# Day 8 — SOC Threat Detection & Incident Response

## 1. Project Overview
This hands-on cybersecurity lab focused on investigating Windows authentication failures, validating domain time synchronization, developing a basic detection rule, and preserving security evidence.

All activities were performed in an isolated Oracle VirtualBox training environment. The authentication failure was deliberately generated using a test account.

## 2. Lab Environment

| Component | Configuration |
|---|---|
| Domain Controller | DC01 — Windows Server 2025 |
| Client | CLIENT01 — Windows 11 |
| Active Directory Domain | adlab.test |
| Client IP Address | 10.0.2.100 |
| Test Account | ADLAB\test.lockout |
| Tools | PowerShell, Get-WinEvent, w32tm, runas |
| Virtualization | Oracle VirtualBox |

## 3. Investigation Objectives
- Establish a baseline of authentication failures.
- Identify source IP addresses and failure codes.
- Verify domain time synchronization.
- Generate a controlled failed authentication attempt.
- Investigate Kerberos Event ID 4771.
- Create a basic repeated-failure detection rule.
- Export and verify investigation evidence.

## 4. Authentication Event Baseline
Queried the Windows Security log on DC01 for the previous seven days.

| Event ID | Description | Count |
|---|---|---:|
| 4625 | Failed logon | 9 |
| 4771 | Kerberos pre-authentication failure | 1 |
| 4740 | Account lockout | 0 |

The baseline counts reflect the seven-day query window at the time of collection.

## 5. Failed Logon Investigation
Analyzed nine Event ID 4625 records.

| Finding | Result |
|---|---|
| Source IP | 10.0.2.100 |
| Account field | Not populated |
| Status | 0xC0000133 |
| Substatus | 0x0 |

Status 0xC0000133 indicates a time difference between the client and server. The records suggested a clock synchronization problem rather than proving malicious activity.

## 6. Time Synchronization Verification
Executed the following commands on CLIENT01 and DC01:

```powershell
Get-Date
w32tm /query /status
```

CLIENT01 reported DC01.adlab.test as its time source and no synchronization warning.

DC01 reported Local CMOS Clock as its time source and no synchronization warning.

The current checks indicated healthy reported synchronization. Historical failures could not be definitively attributed to a specific configuration change.

## 7. Recent Authentication Check
Queried Event ID 4625 for the preceding hour. No matching events were returned on DC01.

This result applied only to the selected log, event ID, and time window.

## 8. Controlled Authentication Failure
From CLIENT01, attempted to run a command prompt under the disposable domain test account:

```powershell
runas /user:ADLAB\test.lockout cmd
```

Entered an intentionally incorrect password once.

Observed:

```text
RUNAS ERROR: Unable to run - cmd
1326: The user name or password is incorrect.
```

Only one intentional failure was generated to avoid unnecessarily triggering the account lockout policy.

## 9. Kerberos Event Investigation
Queried recent Windows Security events on DC01:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4771,4776,4625
    StartTime = (Get-Date).AddMinutes(-15)
} -ErrorAction SilentlyContinue |
Select-Object TimeCreated, Id, Message |
Format-List
```

Observed Event ID 4771.

| Field | Finding |
|---|---|
| Timestamp | November 9, 2026, 10:47:57 AM |
| Event ID | 4771 |
| Account | test.lockout |
| Client Address | ::ffff:10.0.2.100 |
| Failure Code | 0x18 |
| Authentication | Kerberos pre-authentication |
| Assessment | Authorized incorrect-password test |

Failure code 0x18 is consistent with an incorrect password. The event matched the controlled test performed from CLIENT01.

## 10. Repeated-Failure Detection Rule
Queried Event ID 4771 for the previous hour and grouped events by target account.

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4771
    StartTime = (Get-Date).AddHours(-1)
} -ErrorAction SilentlyContinue |
ForEach-Object {
    $xml = [xml]$_.ToXml()
    $account = ($xml.Event.EventData.Data |
        Where-Object Name -eq 'TargetUserName').'#text'
    [PSCustomObject]@{
        Account = $account
        Time = $_.TimeCreated
    }
} |
Group-Object Account |
Where-Object Count -ge 3 |
Select-Object Name,Count
```

**Detection threshold:** Three or more Event ID 4771 records for the same account within one hour.

**Observed result:** One Event ID 4771 in the hour; no accounts exceeded the threshold.

This was a basic training detection rule, not a comprehensive brute-force detection system.

## 11. Evidence Preservation
Created a dedicated evidence directory and exported recent Kerberos authentication events.

```powershell
New-Item -Path 'C:\SecurityLab\Day8' -ItemType Directory -Force

Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4771
    StartTime = (Get-Date).AddHours(-1)
} -ErrorAction SilentlyContinue |
Select-Object TimeCreated, Id, Message |
Format-List |
Out-File 'C:\SecurityLab\Day8\Kerberos-Investigation.txt'
```

Verified the exported file using Get-Item.

| Evidence property | Result |
|---|---|
| Filename | Kerberos-Investigation.txt |
| File size | 2,620 bytes |
| Last modified | November 9, 2026, 10:52:26 AM |
| Location | C:\SecurityLab\Day8 |

The file existed and was nonempty. No cryptographic hash or chain-of-custody procedure was performed.

## 12. Incident Assessment

**Classification:** Authorized security testing / expected authentication failure.

**Severity:** Low concern in the context of this isolated lab.

**Evidence:** One Kerberos pre-authentication failure with code 0x18, associated with test.lockout and CLIENT01.

**Disposition:** No evidence in the reviewed records that the controlled test represented an unauthorized attack. Continue routine monitoring.

## 13. Skills Demonstrated
- Windows Security Event Log investigation
- Active Directory authentication troubleshooting
- Kerberos failure-code analysis
- Source IP attribution
- Windows time synchronization verification
- PowerShell event parsing
- Basic threshold-based detection
- Security evidence export and validation
- SOC investigation documentation

## 14. Conclusion
This exercise demonstrated a practical SOC investigation workflow: establish a baseline, identify an authentication anomaly, validate supporting system conditions, reproduce a controlled failure, inspect the resulting event, evaluate a detection threshold, and preserve evidence.
## Screenshot Evidence

| # | Investigation Evidence | Screenshot |
|---|---|---|
| 1 | Security event baseline | [View](screenshots/01-security-event-baseline.png) |
| 2 | Failed logon source analysis | [View](screenshots/02-failed-logon-source-analysis.png) |
| 3 | CLIENT01 time synchronization | [View](screenshots/03-client01-time-synchronization.png) |
| 4 | DC01 time synchronization | [View](screenshots/04-dc01-time-synchronization.png) |
| 5 | Recent failed logon check | [View](screenshots/05-recent-failed-logon-check.png) |
| 6 | Controlled failed login | [View](screenshots/06-controlled-failed-login.png) |
| 7 | Kerberos Event 4771 investigation | [View](screenshots/07-kerberos-event-4771-investigation.png) |
| 8 | Kerberos failure count | [View](screenshots/08-kerberos-failure-count.png) |
| 9 | Threshold detection rule | [View](screenshots/09-threshold-detection-rule.png) |
| 10 | Security evidence export | [View](screenshots/10-security-evidence-export.png) |
| 11 | Evidence file verification | [View](screenshots/11-evidence-file-verification.png) |

**Environment:** Isolated personal cybersecurity lab. No production systems or real-world security incidents were involved.
