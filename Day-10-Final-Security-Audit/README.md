# Day 10 — Final Security Audit & Verification

## Objective

Perform a final security configuration review of a Windows Server 2025 Active Directory environment. Verify domain password policies, account lockout protections, and Windows Security auditing configurations.

## Lab Environment

- Windows Server 2025
- Oracle VirtualBox
- Active Directory Domain Services (AD DS)
- Domain: adlab.test
- Domain Controller: DC01
- Windows PowerShell
- Windows Security Auditing (auditpol)

## 1. Domain Password Policy Audit

Executed the following PowerShell command on DC01:

```powershell
Get-ADDefaultDomainPasswordPolicy |
Select-Object MinPasswordLength,ComplexityEnabled,PasswordHistoryCount,MaxPasswordAge,LockoutThreshold,LockoutDuration,LockoutObservationWindow |
Format-List
```

### Verified Results

| Security Setting | Verified Configuration |
|---|---|
| Minimum Password Length | 12 characters |
| Password Complexity | Enabled |
| Password History | 10 passwords |
| Maximum Password Age | 90 days |
| Account Lockout Threshold | 5 failed attempts |
| Account Lockout Duration | 15 minutes |
| Lockout Observation Window | 15 minutes |

**Finding:** The configured domain password and lockout settings matched the policies established earlier in the lab.

## 2. Logon and Authentication Auditing

Executed:

```powershell
auditpol /get /category:"Logon/Logoff"
```

### Verified Results

| Audit Subcategory | Configuration |
|---|---|
| Logon | Success and Failure |
| Logoff | Success |
| Account Lockout | Success |
| Special Logon | Success |
| Network Policy Server | Success and Failure |
| IPsec Main Mode | No Auditing |
| IPsec Quick Mode | No Auditing |
| IPsec Extended Mode | No Auditing |
| Other Logon/Logoff Events | No Auditing |
| User / Device Claims | No Auditing |
| Group Membership | No Auditing |
| Access Rights | No Auditing |

**Finding:** Successful and failed logon auditing was enabled, supporting authentication monitoring and investigation.

Not all auditing subcategories were enabled. Additional auditing should be evaluated based on organizational security requirements.

## 3. User Account Management Auditing

Executed:

```powershell
auditpol /get /subcategory:"User Account Management"
```

### Verified Result

**User Account Management: Success**

**Finding:** Successful user account management events were configured for auditing.

This supports the security event investigation performed during Day 9.

## 4. Incident Response Evidence Review

Reviewed the account containment and recovery evidence collected during Day 9.

| Event ID | Description | Verified Result |
|---|---|---|
| 4725 | User account disabled | test.lockout disabled |
| 4722 | User account enabled | test.lockout restored |

Both events were recorded on DC01 on November 16, 2026, and identified ADLAB\Administrator as the account performing the actions.

These events were generated during a controlled incident response simulation, not an actual security breach.

## 5. Security Assessment

### Verified Controls

- Active Directory domain password policy configured
- Password complexity enabled
- Account lockout threshold configured
- Successful and failed logon auditing enabled
- Successful user account management auditing enabled
- Administrative account changes recorded in Windows Security logs
- Test account successfully restored after simulated containment

### Scope and Limitations

This audit verified selected Active Directory security settings and Windows auditing configurations.

It was not a comprehensive vulnerability assessment, penetration test, or production security audit.

The exercise did not validate every security control or determine whether the environment complies with a specific regulatory framework.

## 6. Skills Demonstrated

- Windows Server 2025 administration
- Active Directory security configuration
- Identity and Access Management (IAM)
- Windows Security Event Log analysis
- PowerShell security verification
- Account lockout policy analysis
- Security auditing with auditpol
- Incident response evidence review
- Technical documentation and reporting

## 7. Evidence

Screenshots captured during this lab:

1. `Day-10-01-password-policy-audit.png`
2. `Day-10-02-logon-audit.png`
3. `Day-10-03-account-management-audit.png`

These screenshots document the actual commands and results observed on DC01.

## Conclusion

Successfully verified selected Active Directory password security settings, account lockout configurations, and Windows Security auditing policies.

Combined with the earlier lab exercises, this project demonstrates practical experience building, administering, monitoring, and investigating a Windows Server 2025 Active Directory test environment.

**Project Status: 10 of 10 lab exercises completed.**
