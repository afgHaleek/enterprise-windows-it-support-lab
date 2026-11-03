# Enterprise Windows IT Support Lab

**Windows Server 2025 | Active Directory | DNS | DHCP | Group Policy | PowerShell | Windows 11**

A hands-on enterprise Windows infrastructure lab built in VirtualBox to develop and demonstrate practical skills in IT support, Windows administration, identity management, network services, access control, remote support, and troubleshooting.

The project simulates a small organization's Windows domain environment, with a domain controller providing centralized authentication and network services to a domain-joined workstation.

Each completed lab documents the configuration, testing, verification, and technical evidence collected during implementation.

## 1. Infrastructure Overview

| Component | Configuration |
|---|---|
| Virtualization | Oracle VirtualBox |
| Domain Controller | DC01 — Windows Server 2025 |
| Client Workstation | CLIENT01 — Windows 11 |
| Active Directory Domain | `corp.novatech.test` |
| NetBIOS Domain | `NOVATECH` |
| Domain Controller IP | `192.168.10.10` |
| Client IP | `192.168.10.100` |
| Internal Network | `NovaTech-LAN` |
| Directory Services | Active Directory Domain Services (AD DS) |
| Network Services | DNS and DHCP |
| Centralized Management | Group Policy and PowerShell |
| File Services | SMB shares with NTFS permissions |
| Remote Administration | RDP and PowerShell Remoting (WinRM/Kerberos) |

### Environment Architecture

```text
                 Fedora Workstation
                   VirtualBox Host
                          |
          +---------------+---------------+
          |                               |
     Windows Server 2025             Windows 11
           DC01                       CLIENT01
     192.168.10.10                192.168.10.100
          |                               |
          +---------- NovaTech-LAN -------+
                          |
                  corp.novatech.test

DC01:
  - Active Directory Domain Services
  - DNS Server
  - DHCP Server
  - Group Policy
  - SMB File Services
  - RDP / WinRM

CLIENT01:
  - Domain-joined workstation
  - Domain user authentication
  - Group Policy application
  - File share access
  - Remote administration testing
```

## 2. Implemented Infrastructure

### Active Directory and Identity Management

- Deployed a Windows Server 2025 Active Directory domain.
- Joined a Windows 11 workstation to `corp.novatech.test`.
- Created department-based Organizational Units (OUs).
- Provisioned and managed employee accounts using ADUC and PowerShell.
- Configured security groups for role-based resource access.
- Practiced user onboarding, department transfers, password resets, account lockouts, and offboarding.

### Network Services

- Configured Active Directory-integrated DNS for internal name resolution.
- Configured DHCP services for client IP address allocation.
- Verified domain connectivity, name resolution, and workstation registration.

### Group Policy Management

- Applied user and computer configuration policies.
- Configured Control Panel restrictions and mapped network drives.
- Implemented workstation inactivity locking.
- Tested Windows Firewall ICMP rules and removable storage restrictions.
- Verified Group Policy security filtering and policy application.

### File Services and Access Control

- Configured SMB file shares.
- Applied NTFS and share permissions.
- Implemented Active Directory group-based access using the AGDLP model.
- Tested authorized and unauthorized access to shared resources.

### Remote Support and Administration

- Configured Remote Desktop Protocol (RDP) with Network Level Authentication.
- Tested RDP access and firewall restrictions.
- Configured and verified Windows Remote Management (WinRM).
- Used Kerberos-authenticated PowerShell Remoting.
- Executed remote administration and system health checks.
- Investigated Windows event logs.

## 3. Lab Documentation

The following completed exercises include configuration steps, commands, verification results, and supporting evidence.

| Lab | Technical Focus | Documentation |
|---|---|---|
| Domain Join | Windows 11 domain membership, DNS configuration, secure channel verification | [View documentation](docs/domain-join.md) |
| AD User Provisioning | Organizational Units, user attributes, PowerShell user management, authentication testing | [View documentation](docs/ad-user-management.md) |
| LAB-013 | Active Directory group-based file access, AGDLP, SMB and NTFS permissions | [View lab](docs/labs/LAB-013-AD-Group-Based-File-Access.md) |
| LAB-014 | Group Policy deployment, security filtering, drive mapping, firewall and workstation restrictions | [View lab](docs/labs/LAB-014-Group-Policy-Management.md) |
| LAB-015 | Active Directory user lifecycle: onboarding, transfer, offboarding, account lockouts and auditing | [View lab](docs/labs/LAB-015-Active-Directory-User-Lifecycle.md) |
| LAB-016 | RDP, NLA, firewall access control, WinRM, Kerberos PowerShell Remoting and remote diagnostics | [View lab](docs/labs/LAB-016-Remote-Support-and-Administration.md) |

## 4. Technical Skills Demonstrated

| Area | Practical Experience |
|---|---|
| Windows Server Administration | Server configuration, role management, service verification |
| Active Directory | Domain administration, OUs, users, groups, account lifecycle |
| Windows Client Support | Domain joining, user authentication, policy verification |
| DNS and DHCP | Internal DNS configuration, DHCP administration, connectivity testing |
| Group Policy | Policy deployment, scope, security filtering and troubleshooting |
| Access Management | AGDLP, NTFS permissions, SMB share permissions |
| Remote Support | RDP, NLA, Windows Firewall and PowerShell Remoting |
| PowerShell | AD management, system administration, diagnostics and audit exports |
| Troubleshooting | Connectivity testing, access failures, event log investigation |
| Documentation | Technical procedures, verification records and screenshot evidence |

## 5. Tools and Technologies

- **Operating Systems:** Windows Server 2025, Windows 11, Fedora Linux
- **Virtualization:** Oracle VirtualBox
- **Identity:** Active Directory Domain Services, Kerberos
- **Networking:** TCP/IP, DNS, DHCP
- **Administration:** PowerShell, Windows Admin Tools, Group Policy Management
- **File Services:** SMB, NTFS, AGDLP
- **Remote Management:** RDP, WinRM, PowerShell Remoting
- **Version Control:** Git and GitHub

## 6. Upcoming Labs

| Lab | Planned Focus |
|---|---|
| LAB-017 | Windows troubleshooting: DNS, DHCP, Group Policy and authentication failures |
| LAB-018 | Windows Server backup, file recovery and recovery verification |
| LAB-019 | Real-world helpdesk incident scenarios, diagnosis, resolution and ticket documentation |

Additional exercises may be introduced based on troubleshooting findings and practical IT support requirements.

## 7. Project Objectives

This lab is designed to strengthen practical skills relevant to:

- IT Support Technician
- Service Desk Analyst (L1/L2)
- Desktop Support Technician
- Junior Windows System Administrator

The focus is on understanding how Windows infrastructure operates, diagnosing common support issues, applying appropriate fixes, verifying outcomes, and maintaining clear technical documentation.

## Disclaimer

This is an independently maintained home lab for technical learning and portfolio development. All organizations, users, incidents, and company information are fictional. The environment does not represent a production business network.
