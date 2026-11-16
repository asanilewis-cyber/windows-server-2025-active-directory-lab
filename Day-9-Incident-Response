# Day 9 — Incident Response & Account Containment

## Objective
Simulate a cybersecurity incident by disabling a test Active Directory account, verifying the security audit event, and restoring the account.

## Lab Environment
- Windows Server 2025
- Oracle VirtualBox
- Active Directory Domain Services
- Domain: adlab.test
- Domain Controller: DC01
- PowerShell and Windows Security Event Logs

## Incident Scenario
A test account was treated as potentially compromised for training purposes. The objective was to practice account containment, audit-log investigation, and account recovery.

This was a controlled simulation. No actual compromise was identified.

## Investigation and Response

### 1. Identify the Test Account
Used PowerShell to list Active Directory accounts and verify their enabled status.

Selected account: `test.lockout`

### 2. Contain the Account
Executed:

```powershell
Disable-ADAccount -Identity "test.lockout"
```

Verified the account status changed to `Enabled = False`.

### 3. Verify the Containment Event
Investigated Windows Security Event ID **4725**.

Findings:
- Date: November 16, 2026
- Time: 10:08:19 AM
- Event ID: 4725
- Action: User account disabled
- Target: ADLAB\test.lockout
- Initiated by: ADLAB\Administrator

### 4. Restore the Account
Executed:

```powershell
Enable-ADAccount -Identity "test.lockout"
```

Verified the account status changed to `Enabled = True`.

### 5. Verify the Recovery Event
Investigated Windows Security Event ID **4722**.

Findings:
- Date: November 16, 2026
- Time: 10:11:55 AM
- Event ID: 4722
- Action: User account enabled
- Target: ADLAB\test.lockout
- Initiated by: ADLAB\Administrator

## Results
Successfully completed the simulated account containment and recovery process. Confirmed both administrative actions through Windows Security Event Logs.

## Security Skills Demonstrated
- Incident response fundamentals
- Active Directory account management
- Identity and access management (IAM)
- PowerShell administration
- Windows Security Event Log analysis
- Security event documentation

## Evidence
Screenshots are stored in the `screenshots` folder:

1. `Day-9-01-account-containment.png`
2. `Day-9-02-event-4725.png`
3. `Day-9-03-account-recovery.png`
4. `Day-9-04-event-4722.png`

## Conclusion
This lab demonstrates the ability to perform controlled account containment, verify administrative actions using security logs, and restore access after a simulated incident.
