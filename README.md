
# Windows Server 2025 | Active Directory & IT Security Lab

## Project Overview

A hands-on IT infrastructure and cybersecurity home lab built using Windows Server 2025 and Oracle VirtualBox.

This project demonstrates practical skills in Windows Server administration, network configuration, Active Directory Domain Services (AD DS), DNS, user and group management, and NTFS access controls.

The lab simulates a small business environment with separate IT Support and Human Resources security groups.

## Technologies Used

- Microsoft Windows Server 2025
- Oracle VirtualBox
- Active Directory Domain Services (AD DS)
- Windows DNS Server
- PowerShell
- TCP/IP Networking
- NTFS Permissions
- Active Directory Users and Computers (ADUC)

## Day 1 — Windows Server Installation & Configuration

**Objective:** Build a working Windows Server environment.

### Tasks Completed
- Installed Windows Server 2025 Standard Evaluation in VirtualBox.
- Configured the virtual machine and server hostname `DC01`.
- Verified network connectivity using PowerShell.
- Reviewed Server Manager and server configuration.
- Created a VirtualBox recovery snapshot.

**Outcome:** Established a functioning Windows Server environment for future Active Directory exercises.

## Day 2 — Active Directory & NTFS Security

**Objective:** Configure a Windows domain and implement role-based access controls.

### Tasks Completed
- Configured a VirtualBox internal network named `ADLAB`.
- Assigned static IP address `192.168.50.10/24`.
- Installed Active Directory Domain Services and DNS.
- Promoted `DC01` to a domain controller.
- Created the `adlab.test` domain.
- Created an organizational unit named `Lab-Users`.
- Provisioned test user accounts for John Smith and Sarah Johnson.
- Created `IT-Support` and `HR-Staff` security groups.
- Assigned users to their respective groups.
- Created `C:\HR-Documents`.
- Configured NTFS permissions to grant HR-Staff Modify access.
- Disabled permission inheritance and removed general Users access.
- Reviewed Effective Access results.
- Created a Day 2 VirtualBox snapshot.

**Outcome:** Established an Active Directory domain with organized user accounts, security groups, and folder-level access controls.

## Security Concepts Demonstrated

- Principle of Least Privilege
- Role-Based Access Control (RBAC)
- Identity and Access Management (IAM)
- NTFS Access Control Lists (ACLs)
- Network Segmentation
- Domain Administration
- Security Group Management

## Project Documentation

Detailed PDF reports and screenshots for Day 1 and Day 2 will be added to this repository.

## Upcoming Lab Activities

- Group Policy Objects (GPOs)
- Password and account lockout policies
- Windows security hardening
- Additional access control verification
- Windows event log monitoring

## Professional Development

This project is part of my ongoing hands-on development in IT support, Windows system administration, networking, and cybersecurity.

**Related project:** [Secure AWS Healthcare Document Portal](https://github.com/asanilewis-cyber/secure-aws-healthcare-document-portal)
