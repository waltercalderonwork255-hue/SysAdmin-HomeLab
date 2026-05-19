# Active Directory Lab

## Overview

This project documents my Active Directory home lab built with Windows Server 2022 Evaluation and Windows 11 Pro client machines. The lab was designed to practice core Junior System Administrator skills such as configuring a domain controller, setting up Active Directory Domain Services, managing DNS, organizing users and groups, applying Group Policy, and joining Windows clients to a domain.

The lab uses a VMware NAT network and is not a production environment. It was built for hands-on learning, troubleshooting practice, and GitHub portfolio documentation.

> Sensitive note: Passwords, recovery passwords, private keys, and secrets are not included in this public documentation.

---

## Goal

The goal of this project was to build a small Windows domain environment and practice common Active Directory administration tasks used in real IT environments.

I wanted to practice:

- Installing and configuring Windows Server
- Creating a new Active Directory forest
- Promoting a server to a domain controller
- Configuring DNS for domain authentication
- Installing the DHCP role for practice
- Creating Organizational Units
- Creating users and security groups
- Applying Group Policy Objects
- Joining Windows 11 clients to the domain
- Troubleshooting DNS, domain join, and Group Policy issues

---

## Skills Demonstrated

- Windows Server 2022 administration
- Active Directory Domain Services
- Domain Controller setup
- DNS configuration
- DHCP role installation
- Static IP configuration
- Organizational Unit design
- User account management
- Security group management
- Group Policy Management
- Password policy configuration
- Workstation domain join
- Basic PowerShell administration
- Troubleshooting DNS and GPO issues
- Applying basic Windows domain security practices

---

## Environment

| Component | Details |
|---|---|
| Domain Controller | `DC01` |
| Server OS | Windows Server 2022 Evaluation |
| Domain Name | `corp.local` |
| Client Machines | `CLIENT01`, `CLIENT02` |
| Client OS | Windows 11 Pro |
| DNS Server | `192.168.58.10` / `DC01` |
| DHCP Server | DHCP role installed for practice; no scope configured |
| Network Type | VMware NAT |
| DC IP Address | `192.168.58.10` |
| Subnet Mask | `255.255.255.0` |
| Gateway | `192.168.58.2` |

---

## Project Steps

## Step 1: Installed and Configured Windows Server

**What I did:**  
I installed Windows Server 2022 Evaluation and logged in using the local Administrator account.

**Why I did it:**  
Windows Server was needed to host Active Directory Domain Services, DNS, DHCP, and domain administration tools.

**Commands or settings used:**  

No command was used for the initial installation.

**Verification:**  

- Windows Server 2022 Evaluation installed successfully
- Server was accessible through the console
- Administrator login worked

---

## Step 2: Renamed the Server and Set a Static IP

**What I did:**  
I renamed the server to `DC01` and assigned it a static IP address.

**Why I did it:**  
A domain controller should have a consistent hostname and static IP address so clients can reliably find DNS and Active Directory services.

**Important lesson:**  
The server should be renamed before installing and promoting Active Directory Domain Services.

**Commands or settings used:**

```powershell
Rename-Computer -NewName DC01 -Force -Restart
