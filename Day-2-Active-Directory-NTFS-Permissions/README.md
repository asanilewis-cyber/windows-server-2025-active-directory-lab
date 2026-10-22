# Day 2 — Active Directory & NTFS Security Lab

## Project Overview
Configured a Windows Server 2025 lab environment using Oracle VirtualBox to demonstrate Active Directory administration, identity and access management, security groups, and NTFS file permissions.

## Lab Environment
- **Operating System:** Windows Server 2025
- **Virtualization:** Oracle VirtualBox
- **Server:** DC01
- **Domain:** adlab.test
- **Internal Network:** ADLAB
- **Static IP Address:** 192.168.50.10/24
- **DNS Server:** 192.168.50.10

## Tasks Completed
- Configured an isolated virtual lab network.
- Assigned a static IPv4 address to the server's internal network adapter.
- Installed Active Directory Domain Services (AD DS) and DNS.
- Promoted DC01 to a domain controller for the new adlab.test forest.
- Created the Lab-Users organizational unit (OU).
- Created test user accounts for John Smith and Sarah Johnson.
- Created IT-Support and HR-Staff security groups.
- Assigned users to their respective security groups.
- Created the C:\HR-Documents folder.
- Configured NTFS permissions granting HR-Staff Modify access.
- Removed general Users permissions from the HR-Documents folder.
- Reviewed effective access permissions.
- Created a VirtualBox snapshot to preserve the Day 2 configuration.

## Security Concepts Demonstrated
- Active Directory administration
- Identity and Access Management (IAM)
- Role-Based Access Control (RBAC)
- Principle of Least Privilege
- NTFS Access Control Lists (ACLs)
- Security Group Management
- Windows Server administration

## Project Documentation
[View Day 2 Lab Documentation (PDF)](Hasani_Lewis_Active_Directory_Lab_Day_2_Documentation.pdf)

## Outcome
Established a functioning Active Directory domain and configured group-based NTFS access controls in a Windows Server 2025 virtual lab.
