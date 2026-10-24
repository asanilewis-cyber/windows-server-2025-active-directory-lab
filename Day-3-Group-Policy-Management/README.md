# Day 3 — Active Directory Group Policy Management

## Project Overview
Configured and verified a Group Policy Object (GPO) in a Windows Server 2025 Active Directory lab environment.

This exercise demonstrates foundational Windows Server administration, centralized policy management, security configuration, and PowerShell verification.

## Lab Environment
- Operating System: Windows Server 2025
- Virtualization: Oracle VirtualBox
- Domain: adlab.test
- Domain Controller: DC01
- Organizational Unit: Lab-Users
- GPO: Lab-User security policy

## Objectives
- Understand Group Policy management.
- Create and link a custom Group Policy Object.
- Configure a user-based security restriction.
- Verify GPO settings using Group Policy Management.
- Validate GPO existence and inheritance using PowerShell.
- Preserve the lab configuration using a VirtualBox snapshot.

## Tasks Completed

### 1. Created and Linked a Group Policy Object
- Opened Group Policy Management.
- Created a custom GPO named Lab-User security policy.
- Linked the GPO to the Lab-Users organizational unit.
- Confirmed the GPO link was enabled.

### 2. Configured Command Prompt Restrictions
Navigation:

User Configuration > Policies > Administrative Templates > System

Configured:
- Prevent access to the command prompt: Enabled
- Disable the command prompt script processing also?: No

### 3. Verified Group Policy Configuration
Reviewed the GPO Settings report and confirmed the configured restriction was enabled.

### 4. Verified GPO Using PowerShell
Executed:

    Get-GPO -Name "Lab-User security policy"

Confirmed:
- Domain: adlab.test
- GPO Status: AllSettingsEnabled
- User configuration version: 1

### 5. Verified Organizational Unit Link
Executed:

    Get-GPInheritance -Target "OU=Lab-Users,DC=adlab,DC=test"

Confirmed:
- Lab-Users OU exists.
- Custom GPO is linked.
- Group Policy inheritance is not blocked.
- Default Domain Policy is inherited.

### 6. Created Recovery Snapshot
Saved a VirtualBox snapshot named:

Day 3 - Group Policy Configuration and Verification

## Verification Status
GPO creation, configuration, and OU linking were verified.

Client-side policy application testing is pending until a Windows client is joined to the domain.

## Skills Demonstrated
- Windows Server Administration
- Active Directory Group Policy
- Organizational Unit Management
- Security Policy Configuration
- PowerShell Administration
- GPO Troubleshooting and Verification
- Virtual Machine Snapshot Management

## Next Steps
Day 4 — Configure a Windows client, join it to adlab.test, and test Group Policy application using a standard domain user.
