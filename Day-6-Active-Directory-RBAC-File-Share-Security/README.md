# Day 6 — Active Directory RBAC & Network File Share Security

## Project Overview

This lab demonstrates how Active Directory security groups and NTFS permissions control access to shared folders in a Windows Server 2025 environment.

The exercise simulates a healthcare organization where employees need access to specific departmental files while unauthorized users must be restricted.

I investigated a file-share access issue, verified domain authentication and security group membership, reviewed folder permissions, and tested access using two separate domain accounts.

**Lab environment:** Personal cybersecurity and Windows Server administration training lab.

## 1. Lab Environment

| Component | Configuration |
|---|---|
| Domain Controller | DC01 |
| Server Operating System | Windows Server 2025 |
| Client Computer | CLIENT01 |
| Client Operating System | Windows 11 |
| Active Directory Domain | adlab.test |
| Virtualization Platform | Oracle VirtualBox |
| File Server | DC01 |
| Shared Folder | `\\DC01\Nursing` |
| Server Folder Path | `C:\Hospital-lab\Nursing` |
| Administrative Tools | Active Directory Users and Computers, PowerShell, File Explorer |

## 2. Project Objectives

Apply role-based access control (RBAC) using Active Directory security groups.

Review NTFS permissions on a departmental shared folder.

Verify domain authentication and security group membership.

Investigate why a domain user cannot access a network share.

Confirm authorized users can create and modify files.

Test that unauthorized users are denied access.

Document troubleshooting steps and results.

## 3. Security Configuration

The Nursing folder was configured with NTFS permissions to restrict access based on departmental roles.

Verified NTFS permissions

Security Principal

Permission

Administrators

Full Control

SYSTEM

Full Control

Nursing-Staff

Modify

CREATOR OWNER

Full Control on subfolders and files only

The permissions window showed that inheritance was disabled and the listed permissions were explicitly assigned.

The Nursing-Staff group provides department-based access rather than assigning permissions separately to every employee.

This supports the principle of least privilege: users should receive only the access necessary for their responsibilities.

Note: The screenshots verify NTFS permissions. Share-level permissions were not separately captured, so this report does not claim their exact configuration.

## 4. Initial Problem — Network Share Access Denied

During testing, domain user David Wilson attempted to access the Nursing shared folder from CLIENT01.

Windows displayed the following error:

Windows cannot access \DC01\Nursing

You do not have permission to access \DC01\Nursing. Contact your network administrator to request access.

Initial investigation

The error could have been caused by several issues:

Incorrect domain authentication

DNS resolution problems

Network connectivity problems

Missing Active Directory group membership

Insufficient NTFS or share permissions

The investigation focused on determining whether the problem was related to connectivity, authentication, or authorization.

## 5. DNS and Network Connectivity Verification

Before investigating file permissions, I verified that CLIENT01 could communicate with the domain controller.

DNS cache refresh

ipconfig /flushdns

The command completed successfully.

DNS name resolution

nslookup DC01.adlab.test 10.0.2.3

Verified result:

Name:    DC01.adlab.test
Address: 10.0.2.3

SMB network connectivity

Test-NetConnection DC01.adlab.test -Port 445

Verified result:

ComputerName     : DC01.adlab.test
RemoteAddress    : 10.0.2.3
RemotePort       : 445
SourceAddress    : 10.0.2.100
TcpTestSucceeded : True

Port 445 is used by SMB, the protocol Windows uses to access network file shares.

Finding: CLIENT01 could resolve the domain controller's hostname and establish a TCP connection to its SMB service.

The issue was therefore not explained by basic DNS resolution or TCP connectivity failure.

## 6. Active Directory Authentication Verification

On CLIENT01, I verified the identity of the signed-in user.

whoami

Result:

adlab\dwilson

I also checked the user's domain:

(Get-CimInstance Win32_ComputerSystem).Domain

Result:

adlab.test

The domain controller used for the logon session was identified with:

$env:LOGONSERVER

Result:

\\DC01

I also ran:

nltest /sc_query:adlab.test

Result:

Trusted DC Name \\DC01.adlab.test
Trusted DC Connection Status Status = 0 0x0 NERR_Success
The command completed successfully

These results confirmed that CLIENT01 was domain-joined and could communicate successfully with its domain controller.

## 7. Kerberos Authentication Investigation

I used the following command on CLIENT01:

klist

The output displayed eight cached Kerberos tickets for:

dwilson @ ADLAB.TEST

Observed service tickets included:

krbtgt/ADLAB.TEST

ldap/DC01.adlab.test

cifs/DC01.adlab.test

ProtectedStorage/DC01.adlab.test

The tickets used AES-256 encryption.

The CIFS ticket was particularly relevant because CIFS/SMB is used for Windows network file sharing.

Finding: The user had Kerberos authentication tickets associated with the domain controller and its file-sharing service.

This supported the conclusion that the access issue required further authorization investigation.

## 8. Active Directory Group Membership Investigation

On DC01, I checked David Wilson's security group membership.

Get-ADPrincipalGroupMembership -Identity dwilson |
    Select-Object Name

Result:

Name
----
Domain Users

I then checked the user's primary group:

Get-ADUser -Identity dwilson -Properties PrimaryGroupID |
    Select-Object Name,SamAccountName,PrimaryGroupID

Result:

Name         SamAccountName PrimaryGroupID
----         -------------- --------------
David Wilson dwilson        513

I verified that primary group ID 513 represented Domain Users.

Get-ADGroup -Identity "Domain Users" |
    Select-Object Name,SID

I also reviewed the user's explicitly assigned groups:

Get-ADUser -Identity dwilson -Properties MemberOf,PrimaryGroupID |
    Select-Object Name,PrimaryGroupID,MemberOf

Result:

Name         PrimaryGroupID MemberOf
----         -------------- --------
David Wilson 513            {}

This showed that David Wilson's primary group was Domain Users and no additional direct group memberships were listed.

He was not shown as a member of the Nursing-Staff security group.

Important troubleshooting lesson

I initially investigated why Domain Users did not appear in the output of whoami /groups.

The Active Directory investigation confirmed David Wilson's primary group ID was 513. However, the client-side token output did not display that domain group.

The screenshots establish his Active Directory membership and the missing Nursing-Staff authorization; they do not establish why Domain Users was absent from the client token display.

## 9. Nursing-Staff Group Verification

On DC01, I ran:

Get-ADGroupMember -Identity "Nursing-Staff" |
    Select-Object Name,SamAccountName

Result:

Name           SamAccountName
----           --------------
Michael Carter mcarter

This confirmed that Michael Carter was a member of the Nursing-Staff group.

David Wilson was not listed.

Authorization comparison

User

Nursing-Staff Membership

Observed Access

Michael Carter (mcarter)

Yes

Write access successful

David Wilson (dwilson)

No

Access denied

This matched the intended departmental access restrictions.

## 10. Authorized User Access Test

On CLIENT01, I signed in as:

adlab\mcarter

I verified the identity using:

whoami

Result:

adlab\mcarter

I then accessed:

\\DC01\Nursing

The folder opened successfully.

I created a test document named:

Nursing staff write access verified.

I entered the following text into the document:

Nursing staff write access verified.

The file was saved successfully in the shared folder.

Result: PASS

The test demonstrated that Michael Carter could create and save a file in the Nursing shared folder.

This was consistent with the Modify permission assigned to Nursing-Staff.

## 11. Unauthorized User Access Test

Using David Wilson's domain account, I attempted to access:

\\DC01\Nursing

Windows displayed an access-denied message.

The account was authenticated to the domain, and the network connection to DC01 was functional.

However, David Wilson was not a member of the Nursing-Staff security group.

Result: PASS — access restriction enforced

The denied access was the expected outcome for a user without the required departmental authorization.

## 12. Troubleshooting Summary

Investigation

Result

DNS cache refresh

Successful

DC01 hostname resolution

Successful

SMB TCP port 445

Reachable

CLIENT01 domain membership

Confirmed

Domain controller connection

Successful

Kerberos service tickets

Present

David Wilson's primary group

Domain Users

David Wilson's Nursing-Staff membership

Not present

Michael Carter's Nursing-Staff membership

Confirmed

Nursing folder NTFS permissions

Verified

Michael Carter's file creation

Successful

David Wilson's share access

Denied

Root Cause

The access restriction was consistent with the intended role-based security configuration.

The Nursing folder granted Modify access to Nursing-Staff, and David Wilson was not a member of that group.

The evidence supports an authorization restriction rather than a basic network connectivity failure.

## 13. Security Principles Demonstrated

Role-Based Access Control (RBAC)

Permissions are assigned to a group representing a job function rather than individually to each user.

Least Privilege

Only users requiring Nursing department access should receive the appropriate permissions.

Authentication vs. Authorization

Authentication verifies who a user is. Authorization determines what that user is allowed to access.

David Wilson successfully authenticated to the domain but was denied access to a restricted departmental folder.

Defense in Depth

Active Directory group membership and NTFS permissions work together to protect shared resources.

Security Validation

Access controls should be tested using both authorized and unauthorized accounts.

## 14. Final Results

Test

Outcome

Domain authentication

PASS

DNS resolution

PASS

SMB connectivity

PASS

Kerberos authentication evidence

PASS

Active Directory group membership verification

PASS

Nursing folder NTFS configuration review

PASS

Authorized user file creation

PASS

Unauthorized user access restriction

PASS

## 15. Skills Demonstrated

Windows Server 2025 administration

Active Directory Domain Services

Role-Based Access Control (RBAC)

Identity and Access Management (IAM)

NTFS file permissions

Windows SMB network shares

Active Directory security group administration

PowerShell troubleshooting

DNS troubleshooting

Kerberos authentication investigation

Windows domain authentication

Least-privilege security

Access control validation

Technical incident documentation

## 16. Conclusion

This lab demonstrated how Windows Server 2025, Active Directory security groups, and NTFS permissions can protect departmental network shares.

I investigated an access-denied issue by checking DNS, SMB connectivity, domain authentication, Kerberos tickets, Active Directory group membership, and folder permissions.

The final tests confirmed that Michael Carter could create and save files in the Nursing share while David Wilson was denied access.

The exercise reinforced the importance of separating authentication from authorization and verifying security controls through practical testing.

Project status: Completed — personal Windows Server 2025 Active Directory lab.
