Day 05 — Active Directory Password Security & Account Lockout
Windows Server 2025 | Active Directory Domain Services | Windows 11 | PowerShell | Windows Security Logs
Lab date: October 28, 2026
Environment: Isolated Oracle VirtualBox training environment
Domain: adlab.test | Domain controller: DC01 | Client: CLIENT01
Overview
Configured and verified domain password and account-lockout policies, tested failed domain sign-ins, investigated Windows Security Event ID 4740, and manually unlocked a disposable Active Directory account. All activities were performed in a personal training lab, not a production environment.
Objectives
- Configure password security through the Default Domain Policy.
- Configure account-lockout controls and verify effective domain settings with PowerShell.
- Create a disposable domain test user.
- Investigate failed authentication and identify an account-lockout event.
- Restore access using Unlock-ADAccount and confirm the account is unlocked.
Environment & tools
- Windows Server 2025 domain controller (DC01)
- Domain-joined Windows 11 virtual machine (CLIENT01)
- Active Directory Users and Computers (ADUC)
- Group Policy Management Editor
- Active Directory PowerShell module
- Event Viewer → Windows Logs → Security
Configuration and validation
Setting	Lab value	Evidence
Minimum password length	12 characters	Password Policy + PowerShell
Password complexity	Enabled	Password Policy + PowerShell
Password history	10 remembered	Password Policy + PowerShell
Maximum password age	90 days	PowerShell .ToString() verification
Minimum password age	1 day	Group Policy Editor
Reversible password encryption	Disabled	Group Policy Editor
Account lockout threshold	5 failed attempts	Group Policy Editor + PowerShell
Account lockout duration	15 minutes	Group Policy Editor + PowerShell
Reset lockout counter	15 minutes	Group Policy Editor + PowerShell
Administrator account lockout	Enabled in lab	Group Policy Editor


These settings were chosen for demonstration in an isolated lab; they are not presented as universal production recommendations.
Procedure
1. Opened the Default Domain Policy in Group Policy Management Editor and configured the password and account lockout settings.
2. Verified the domain policy using the Active Directory PowerShell module:
   Get-ADDefaultDomainPasswordPolicy | Select-Object MinPasswordLength,ComplexityEnabled,PasswordHistoryCount,MaxPasswordAge,LockoutThreshold,LockoutDuration,LockoutObservationWindow
   (Get-ADDefaultDomainPasswordPolicy).MaxPasswordAge.ToString()
3. Created the disposable user test.lockout in Active Directory Users and Computers. An initial password was rejected by policy; account creation succeeded after choosing a compliant password.
4. Tested incorrect sign-ins from CLIENT01. An initial test was accidentally performed using jsmith, rather than test.lockout.
5. Filtered DC01 Security logs for 4771, 4776, and 4740. An Event ID 4740 record identified the locked account as ADLAB\jsmith, with caller computer CLIENT01. This event is evidence for the initial jsmith test, not for test.lockout.
6. Repeated the failed sign-in test with the intended disposable account test.lockout and verified its lockout status:
   Get-ADUser -Identity 'test.lockout' -Properties LockedOut | Select-Object Name,SamAccountName,LockedOut
   The output showed LockedOut = True.
7. Manually restored access:
   Unlock-ADAccount -Identity 'test.lockout'
   Get-ADUser -Identity 'test.lockout' -Properties LockedOut | Select-Object Name,SamAccountName,LockedOut
   The final output showed LockedOut = False.
Troubleshooting notes
- Incorrect account during initial lockout test: Event ID 4740 revealed jsmith as the affected user, helping identify the test-account mix-up.
- PowerShell input errors: Corrected mistyped cmdlets, property names, account identities, and accidental copying of the PS C:\...> prompt.
- PowerShell display formatting: MaxPasswordAge initially appeared as {} in a selected-object view. Using .ToString() confirmed 90.00:00:00.
- Status versus history: A later LockedOut = False check on jsmith did not invalidate the earlier 4740 event; the account could have unlocked automatically under the 15-minute policy.
- Separate evidence: No Event ID 4740 screenshot specifically identifying test.lockout was captured in the supplied evidence. The test.lockout lockout itself was verified with PowerShell (True) and its recovery with PowerShell (False).
Results
PASS: Domain policy values verified; disposable test account created; account lockout reproduced and confirmed for test.lockout; manual unlock command completed; post-remediation state verified as False. A separate Event ID 4740 was analyzed for jsmith, with CLIENT01 as the caller computer.
Screenshot evidence
File	What it demonstrates
01-group-policy.png	Default Domain Policy editor
02-password-policy.png	Password Policy configuration
03-account-lockout-policy.png	Lockout threshold and duration
04-powershell-verification.png	PowerShell domain policy verification
05-test-user.png	Created test.lockout AD user
06-failed-login-attempts.png	Initial Windows 11 authentication failure
07-lockout-status-investigation.png	test.lockout initially reported unlocked
08-failed-authentication-investigation.png	Investigation of last bad password attempt
09-security-event-investigation.png	Filtered Windows Security log
10-event-4740-jsmith-investigation.png	Event 4740: jsmith / CLIENT01
11-jsmith-status-verification.png	jsmith status checked later (unlocked)
12-test-account-login-failure.png	Repeat test with disposable account
13-test-account-locked-true.png	test.lockout verified locked (True)
14-manual-account-unlock.png	Unlock-ADAccount and verified False


All screenshots were taken in an isolated personal lab. Review images before public posting to ensure no sensitive information is present.
Skills demonstrated
Active Directory administration · Group Policy · Account security · Windows authentication · PowerShell · Security event investigation · Troubleshooting · Help desk account recovery · IAM fundamentals · Technical documentation
Portfolio context
This is a hands-on personal lab, not a claim of production-domain administration. The work demonstrates configuration, troubleshooting, investigation, and evidence-based remediation using Microsoft Windows Server tools.
