# Enterprise Windows IT Support Lab

A simulated enterprise Windows environment built to demonstrate practical
IT support, system administration, networking, troubleshooting, and
PowerShell automation skills.

## Project Status

### Active Directory — Domain Join

Successfully joined a Windows 11 workstation to the `corp.novatech.test` domain hosted on Windows Server 2025.

- Configured the client to use internal Active Directory DNS.
- Joined CLIENT01 using domain administrator credentials.
- Verified domain membership and computer account registration.
- Confirmed the workstation's secure channel with the domain.

[View domain join documentation](docs/domain-join.md)

### Active Directory User Provisioning

Provisioned standard employee accounts in a Windows Server 2025 Active Directory environment, using both Active Directory Users and Computers (ADUC) and PowerShell.

- Organized employee accounts into department-specific OUs.
- Configured sign-in names and user account attributes.
- Practiced `Get-ADUser`, `Set-ADUser`, and `New-ADUser`.
- Successfully tested domain user authentication on a Windows 11 workstation.
- Documented account verification and the provisioning workflow.

[View implementation and screenshots](docs/ad-user-management.md)

## Planned Environment

- Windows Server 2025
- Windows 11 client
- Active Directory Domain Services
- DNS
- DHCP
- Group Policy
- File Services
- PowerShell
- VirtualBox


## Windows Server & Active Directory Labs

| Lab | Focus | Documentation |
|---|---|---|
| LAB-013 | AD group-based file access (AGDLP, NTFS and SMB permissions) | [Read lab](docs/labs/LAB-013-AD-Group-Based-File-Access.md) |
| LAB-014 | Group Policy deployment, computer and user policies, security filtering, verification and troubleshooting | [Read lab](docs/labs/LAB-014-Group-Policy-Management.md) |

**LAB-014 highlights:** configured a Finance Control Panel restriction and mapped drive; enforced a 60-second workstation inactivity lock; deployed a scoped ICMPv4 firewall rule; tested removable-disk write restrictions; and verified security filtering with two users in the same Finance OU. Screenshots and troubleshooting evidence are included in the lab report.

> WMI filtering and GPO inheritance/precedence are planned topics, not completed tests in LAB-014.

| LAB-015 | Active Directory User Lifecycle (Joiner–Mover–Leaver): onboarding, group access, password reset, lockout investigation, department transfer, offboarding, PowerShell audit | [Read lab](docs/labs/LAB-015-Active-Directory-User-Lifecycle.md) |

Optional project highlight:

> **LAB-015:** Completed a simulated employee lifecycle in a Windows Server 2025 domain: created and validated a new user, verified AGDLP resource access, handled password resets and account lockouts, transferred an employee between departments with access revocation, disabled and retained a departing employee account, and exported an Active Directory account audit to CSV.
r

| LAB-016 | Remote Support and Windows Administration | RDP/NLA, access auditing, source-IP firewall hardening, WinRM, Kerberos PowerShell Remoting, remote health and event logs | [View Lab](docs/labs/LAB-016-Remote-Support-and-Administration.md) |

## Planned Skills Demonstrated

- Active Directory administration
- Windows client administration
- DNS and DHCP configuration
- Group Policy management
- User and group management
- File and NTFS permissions
- Network troubleshooting
- IT incident troubleshooting
- PowerShell automation
- Technical documentation

## Lab Architecture

Architecture documentation will be added as the environment is built.

## Troubleshooting Scenarios

Realistic IT support incidents will be introduced, diagnosed, resolved,
and documented throughout the project.

## Disclaimer

This is a personal home lab created for learning and portfolio purposes.
All users, company names, systems, and incidents used in this project are fictional.# enterprise-windows-it-support-lab
Description: Enterprise Windows lab demonstrating Active Directory, DNS, DHCP, Group Policy, PowerShell, access control and real-world IT troubleshooting.
