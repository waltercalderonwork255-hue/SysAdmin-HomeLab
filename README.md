# SysAdmin Home Lab

## Overview

This repository documents my personal SysAdmin home lab built to practice hands-on IT administration skills for a Junior System Administrator role.

The lab was designed to simulate common small business and enterprise IT tasks in a safe learning environment. It includes Windows Server, Active Directory, DNS, DHCP, Group Policy, file sharing, backups, Linux administration, VPN access, monitoring, and automation.

This is not a production environment. It is a learning and portfolio lab used to practice real sysadmin concepts, troubleshoot issues, document configurations, and build resume-ready experience.

## Target Skills

This home lab demonstrates beginner-to-intermediate Junior System Administrator skills, including:

- Windows Server administration
- Active Directory Domain Services
- DNS and DHCP configuration
- Group Policy management
- User, group, and permission management
- Windows client domain joining
- Linux server administration
- SMB/CIFS file sharing
- Access-based permissions and share testing
- Backup and recovery concepts
- Scheduled backups and restore verification
- VPN configuration and remote access testing
- Basic monitoring and log review
- PowerShell and Bash automation
- Troubleshooting and documentation

## Lab Environment

| Component | Details |
|---|---|
| Lab Type | Personal home lab |
| Target Role | Junior System Administrator |
| Virtualization | VMware / virtual machines |
| Windows Server | Used for domain, Active Directory, DNS, DHCP, file services, and Group Policy practice |
| Windows Clients | Domain-joined Windows client machines used for testing access and policies |
| Linux Server | Used for Linux administration, rsync backups, VPN, Jellyfin/media access testing, and other services |
| Domain | `corp.local` |
| Domain Controller | `DC01` |
| Network Type | NAT network |
| Example DC IP | `192.168.58.10` |
| Services Practiced | AD DS, DNS, DHCP, SMB/CIFS, backups, VPN, monitoring, automation |
| Security Note | Passwords, tokens, private keys, and sensitive information should not be committed to this repository |

Some details may vary by project folder. Any unclear or unfinished information is marked as **Needs clarification** inside the individual project README files.

## Project Structure

```text
SysAdmin-HomeLab/
│
├── README.md
├── LabSetup/
│   └── README.md
├── ActiveDirectory/
│   └── README.md
├── Backups/
│   └── README.md
├── FileServer/
│   └── README.md
├── VPN/
│   └── README.md
├── Monitoring/
│   └── README.md
└── Automation/
    └── README.md
```

## Folder Overview

### LabSetup

Documents the full home lab setup, including virtual machines, network layout, domain information, IP addressing, and the purpose of each system in the lab.

### ActiveDirectory

Documents the Windows domain environment, including Active Directory Domain Services, DNS, DHCP, organizational units, users, groups, Group Policy, and domain-joined client testing.

### Backups

Documents backup and recovery practice, including Windows Server Backup concepts, scheduled backups, Linux rsync backup ideas, backup verification, and restore testing.

### FileServer

Documents SMB/CIFS file sharing, department shares, permissions, access-based enumeration, and client access testing.

### VPN

Documents WireGuard VPN setup, client configuration, remote access testing, NAT/bridged networking considerations, firewall rules, and troubleshooting.

### Monitoring

Documents system monitoring, service checks, disk/CPU/RAM checks, uptime checks, Windows Event Viewer, Linux logs, and troubleshooting.

### Automation

Documents PowerShell scripts, Bash scripts, Windows Task Scheduler, cron jobs, backup automation, system checks, and documentation automation.

## Projects

### LabSetup

The LabSetup project explains how the home lab was built, including the virtual machines, network type, domain name, IP addressing, and the purpose of each server/client system.

[View LabSetup Project](./LabSetup/README.md)

### ActiveDirectory

The ActiveDirectory project focuses on building and managing a Windows domain environment. It includes Active Directory Domain Services, DNS, DHCP, organizational units, users, groups, Group Policy, and Windows client domain testing.

[View ActiveDirectory Project](./ActiveDirectory/README.md)

### Backups

The Backups project focuses on backup and recovery practice. It includes Windows Server Backup concepts, backup disk setup, scheduled backups, Linux rsync backup ideas, restore testing, and backup verification.

[View Backups Project](./Backups/README.md)

### FileServer

The FileServer project focuses on shared folder access, department-based permissions, and SMB/CIFS file sharing. It includes HR, Sales, Admin, and Executive share testing, access-based enumeration, read/write permissions, and client access verification.

[View FileServer Project](./FileServer/README.md)

### VPN

The VPN project focuses on WireGuard VPN configuration and remote access testing. It documents VPN server/client setup, firewall and network considerations, NAT vs bridged networking issues, and connection troubleshooting.

[View VPN Project](./VPN/README.md)

### Monitoring

The Monitoring project focuses on basic sysadmin monitoring tasks. It includes service monitoring, log review, uptime checks, disk space checks, CPU/RAM checks, Windows Event Viewer, Linux logs, and troubleshooting.

[View Monitoring Project](./Monitoring/README.md)

### Automation

The Automation project focuses on using scripts and schedulers to reduce repetitive sysadmin work. It may include PowerShell scripts, Bash scripts, Windows Task Scheduler, cron jobs, backup automation, user/group automation, system checks, and documentation automation.

[View Automation Project](./Automation/README.md)

## Skills Demonstrated

This lab demonstrates the following skills based on the project notes:

- Windows Server administration
- Active Directory Domain Services
- Domain Controller configuration
- DNS configuration and troubleshooting
- DHCP configuration
- Group Policy management
- Organizational Unit structure
- User and group management
- Domain-joined Windows client testing
- File share creation and permission management
- SMB/CIFS file sharing
- Access-based enumeration testing
- Backup planning and scheduled backup concepts
- Restore testing and backup verification
- Linux administration
- rsync backup concepts
- Cron job scheduling
- WireGuard VPN configuration
- VPN client/server troubleshooting
- NAT networking troubleshooting
- Monitoring services and system health
- Reviewing Windows and Linux logs
- PowerShell scripting
- Bash scripting
- Windows Task Scheduler
- Troubleshooting common sysadmin issues
- Writing technical documentation
- Creating GitHub-ready project documentation

## What I Practiced

In this lab, I practiced common Junior SysAdmin tasks such as:

- Setting up a Windows Server lab environment
- Creating and managing a Windows domain
- Configuring DNS and DHCP services
- Creating users, groups, and organizational units
- Applying and testing Group Policy settings
- Joining Windows clients to a domain
- Creating file shares for different departments
- Testing read, write, modify, and deny permissions
- Verifying that users only see and access the folders they are allowed to use
- Planning backup methods and testing backup concepts
- Using Linux tools such as rsync and cron
- Setting up and troubleshooting VPN access
- Reviewing logs and system health
- Automating repeatable tasks with scripts
- Documenting issues, fixes, screenshots, and lessons learned

## Resume Summary

Built and documented a personal SysAdmin home lab using Windows Server, Active Directory, DNS, DHCP, Group Policy, Linux, SMB/CIFS file sharing, backups, VPN configuration, monitoring, and automation. Practiced user/group administration, file share permissions, backup verification, troubleshooting, and technical documentation in a virtual lab environment.

## Resume Bullet Examples

- Built a personal Windows Server and Linux home lab to practice Junior System Administrator tasks including Active Directory, DNS, DHCP, Group Policy, backups, file sharing, VPN access, monitoring, and automation.
- Configured and tested domain users, security groups, organizational units, Group Policy settings, and domain-joined Windows clients in a virtual lab environment.
- Created SMB/CIFS file shares with department-based permissions and tested access control using HR, Sales, Admin, and Executive user scenarios.
- Practiced backup and recovery concepts using Windows Server Backup and Linux rsync backup workflows.
- Documented troubleshooting steps, verification results, screenshots, and lessons learned in a GitHub portfolio repository.

## Interview Summary

**“Tell me about your home lab.”**

I built a personal SysAdmin home lab to practice the skills needed for a Junior System Administrator role. The lab includes Windows Server, Active Directory, DNS, DHCP, Group Policy, Windows clients, Linux servers, file sharing, backups, VPN access, monitoring, and automation.

I used the lab to practice common sysadmin tasks like creating users and groups, joining clients to a domain, applying Group Policy, setting up file share permissions, testing access control, configuring backups, reviewing logs, troubleshooting network issues, and documenting my work. It is not a production environment, but it helped me build hands-on experience with real tools and workflows used in IT support and system administration.

## Screenshots and Diagrams To Add

Recommended screenshots and diagrams for this repository:

### LabSetup

- Overall lab network diagram
- VM list showing server/client roles
- NAT network settings
- IP address plan
- Folder structure of the GitHub repo

### ActiveDirectory

- Server Manager showing AD DS installed
- Active Directory Users and Computers
- Organizational Units
- Users and groups
- DNS Manager
- DHCP Manager
- Group Policy Management
- Windows client joined to `corp.local`
- `gpresult` or `whoami /groups` verification output

### Backups

- Backup disk or destination configuration
- Windows Server Backup schedule
- Successful backup result
- Restore test result
- Linux rsync command output
- Cron job schedule, if used

### FileServer

- Shared folders
- NTFS permissions
- Share permissions
- Access-based enumeration settings
- HR user only seeing HR folder
- Sales user only seeing Sales folder
- Executive user access test
- Access denied test
- Successful create/edit/delete test inside allowed share

### VPN

- WireGuard server configuration with private keys removed
- WireGuard client configuration with private keys removed
- VPN connection status
- Firewall or port forwarding settings, if used
- Ping or remote access test over VPN
- Network diagram showing VPN path

### Monitoring

- Windows Event Viewer example
- Linux log review example
- Disk space check
- CPU/RAM check
- Service status check
- Monitoring dashboard, if used
- Alert or notification test, if used

### Automation

- PowerShell script examples
- Bash script examples
- Windows Task Scheduler job
- Cron job schedule
- Script output showing success
- Before/after example of a task being automated

## Sensitive Information Warning

Before uploading anything to GitHub, I should make sure the repository does not include sensitive information.

Do not upload:

- Passwords
- API keys
- Private keys
- VPN private keys
- Tokens
- Recovery codes
- Real personal information
- Public IP addresses, unless intentionally shared
- Screenshots showing usernames/passwords
- Screenshots showing private keys or secrets
- Configuration files containing real credentials

Use placeholders instead, such as:

```text
PASSWORD_REMOVED
TOKEN_REMOVED
PRIVATE_KEY_REMOVED
API_KEY_REMOVED
IP_ADDRESS_EXAMPLE
USERNAME_EXAMPLE
```

## Notes

This repository is for learning, documentation, and resume portfolio purposes.

Sensitive information has been removed or should be sanitized before being uploaded to GitHub. Any passwords, tokens, API keys, private keys, or personal information should be replaced with safe placeholder values.

This lab is intentionally documented in a beginner-friendly way to show what I built, what I tested, what I learned, and how I troubleshot issues while practicing Junior System Administrator skills.
