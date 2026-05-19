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

### Step 1: Installed and Configured Windows Server

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

### Step 2: Renamed the Server and Set a Static IP

**What I did:**  
I renamed the server to `DC01` and assigned it a static IP address.

**Why I did it:**  
A domain controller should have a consistent hostname and static IP address so clients can reliably find DNS and Active Directory services.

**Important lesson:**  
The server should be renamed before installing and promoting Active Directory Domain Services.

**Commands or settings used:**

```powershell
Rename-Computer -NewName DC01 -Force -Restart
```

Static IP settings used:

```text
IP Address: 192.168.58.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.58.2
Preferred DNS: 192.168.58.10
```

**Verification:**  

- Server name changed to `DC01`
- Server restarted successfully
- Static IP address stayed assigned after reboot
- Server used itself as DNS

---

### Step 3: Installed Active Directory Domain Services, DNS, and DHCP Roles

**What I did:**  
I used Server Manager to install the following roles:

- Active Directory Domain Services
- DNS Server
- DHCP Server

**Why I did it:**  

- Active Directory Domain Services provides centralized identity and authentication.
- DNS is required so domain clients can locate the domain controller.
- DHCP was installed for practice, but no DHCP scope was configured in this lab.

**Settings used:**

```text
Server Manager
→ Manage
→ Add Roles and Features
→ Server Roles
```

Selected roles:

```text
Active Directory Domain Services
DNS Server
DHCP Server
```

Optional tools included:

```text
Group Policy Management
RSAT / AD DS Tools
```

**Verification:**  

- AD DS role installed
- DNS role installed
- DHCP role installed
- Server Manager showed the roles as available

---

### Step 4: Promoted the Server to a Domain Controller

**What I did:**  
After installing the AD DS role, I promoted `DC01` to a domain controller and created a new forest named `corp.local`.

**Why I did it:**  
Promoting the server to a domain controller allowed it to manage the Active Directory domain, authenticate users, and provide domain services.

**Settings used:**

```text
Deployment Operation: Add a new forest
Root Domain Name: corp.local
DNS Server: Enabled
Global Catalog: Enabled
Read-only Domain Controller: Disabled
```

**Important security note:**  
The Directory Services Restore Mode password was used during setup, but it should never be documented in a public GitHub repository.

**Verification:**  

- Server was promoted to a domain controller
- Domain `corp.local` was created
- Server restarted after promotion
- Active Directory tools became available

---

### Step 5: Configured DNS and DHCP

**What I did:**  
I configured the domain controller to use itself as the preferred DNS server.

I also installed the DHCP role for practice, but I did not configure a DHCP scope.

**Why I did it:**  
DNS is required for Active Directory. Domain-joined clients must use the domain controller as their DNS server so they can locate the domain.

**Settings used:**

DNS setting on `DC01`:

```text
Preferred DNS: 192.168.58.10
```

Manual DNS setting used on `CLIENT02` during troubleshooting:

```text
Preferred DNS: 192.168.58.10
Alternate DNS: Leave blank
```

**Important lesson:**  
Do not use the NAT gateway as DNS for domain-joined clients. The client should point to the domain controller for DNS.

**Verification:**  

- `CLIENT01` and `CLIENT02` were able to find the domain
- Domain join issue was fixed after correcting DNS
- DHCP role was installed, but no DHCP scope was configured

---

### Step 6: Created Organizational Units

**What I did:**  
I created an Organizational Unit structure to organize domain objects.

**Why I did it:**  
OUs make it easier to organize users, computers, servers, groups, and service accounts. They also allow Group Policy to be applied more cleanly.

**OU structure used:**

```text
corp.local
│
├── _Admins
├── _Servers
├── _Workstations
├── _Users
│   ├── IT
│   ├── HR
│   ├── Sales
│   └── Executives
├── _ServiceAccounts
└── _Groups
```

**Verification:**  

- OUs were visible in Active Directory Users and Computers
- Users and groups were placed into the correct OUs
- The structure helped organize users, computers, and security groups

---

### Step 7: Created Users

**What I did:**  
I created test users for different departments.

**Why I did it:**  
Creating users helped me practice common Active Directory administration tasks such as account creation, department organization, and user placement inside OUs.

**Users created:**

| Name | Username | Department / OU |
|---|---|---|
| Walter Calderon | `wcalderon` | IT |
| Mike Jones | `mjones` | HR |
| Ralph Stone | `rstone` | Sales |
| Walter Calderon Admin | `wcalderon-adm` | IT Admin |
| Norah Calderon | `ncalderon` | Executives |

> Passwords were intentionally removed from this README.

**Verification:**  

- Users appeared in Active Directory Users and Computers
- Users were placed in the correct OUs
- Domain user accounts were available for client login testing

---

### Step 8: Created and Tested Security Groups

**What I did:**  
I created security groups to manage administrative access.

**Why I did it:**  
Using groups is better than assigning permissions directly to individual users. This is easier to manage and follows common enterprise practice.

**Groups created and tested:**

| Group Name | Scope | Type |
|---|---|---|
| `GRP_Domain_Admins` | Global | Security |
| `GRP_Server_Admins` | Global | Security |
| `GRP_Workstation_Admins` | Global | Security |
| `GRP_Helpdesk_Admins` | Global | Security |

**Security design:**  

Instead of adding users directly to built-in privileged groups, I created custom groups and managed access through group membership.

Example:

```text
GRP_Domain_Admins
```

was nested into:

```text
Domain Admins
```

**Verification:**  

- Groups were created in Active Directory
- Group type was set to Security
- Group scope was set to Global
- Admin groups were tested
- Group-based access was used instead of direct user permissions

---

### Step 9: Configured and Tested Group Policy

**What I did:**  
I created and tested Group Policy Objects for security and desktop management.

**Why I did it:**  
Group Policy allows administrators to centrally enforce settings across users and computers.

#### Password Policy

**GPO Name:**

```text
DomainPasswordPolicy
```

**Linked to:**

```text
corp.local
```

**Policy path:**

```text
Computer Configuration
→ Policies
→ Windows Settings
→ Security Settings
→ Account Policies
→ Password Policy
```

**Settings configured:**

```text
Minimum password length: 12
Password must meet complexity requirements: Enabled
Maximum password age: Not configured
```

**Verification:**  

- Password policy was configured at the domain level
- Policy was tested successfully

#### Disable USB Storage

**GPO Name:**

```text
Disable USBStorage
```

**Linked to:**

```text
_Workstations
```

**Policy path:**

```text
Computer Configuration
→ Policies
→ Administrative Templates
→ System
→ Removable Storage Access
```

**Setting configured:**

```text
All Removable Storage classes: Deny all access
```

**Verification:**  

- USB restriction GPO was linked to `_Workstations`
- Policy was tested successfully on domain-joined clients

#### Lock Screen After 10 Minutes

**Linked to:**

```text
_Users
```

**Policy path:**

```text
User Configuration
→ Policies
→ Administrative Templates
→ Control Panel
→ Personalization
```

**Settings configured:**

```text
Enable screen saver
Screen saver timeout: 600 seconds
Password protect the screen saver
```

**Verification:**  

- Lock screen settings were configured under User Configuration
- GPO was linked to `_Users`
- Policy was tested successfully

#### Desktop Wallpaper Enforcement

**Linked to:**

```text
_Users
```

**Policy path:**

```text
User Configuration
→ Policies
→ Administrative Templates
→ Desktop
→ Desktop
→ Desktop Wallpaper
```

**Wallpaper path used for testing:**

```text
\\corp.local\SYSVOL\corp.local\scripts\dragonball.jpg
```

**Best practice learned:**  
Use the domain path instead of an IP address when referencing SYSVOL resources.

**Verification:**  

- Wallpaper policy was created
- Wallpaper policy was linked to `_Users`
- Wallpaper policy was tested successfully after fixing the file extension issue

---

### Step 10: Joined Windows 11 Clients to the Domain

**What I did:**  
I renamed both client machines and joined them to the `corp.local` domain.

**Why I did it:**  
Domain joining allows client machines to authenticate with Active Directory and receive centralized policies.

**Client names:**

```text
CLIENT01
CLIENT02
```

**Domain joined:**

```text
corp.local
```

**Example domain login format:**

```text
corp\username
```

**Verification:**  

- `CLIENT01` joined the domain successfully
- `CLIENT02` joined the domain successfully
- Domain users could log in
- DNS issues were resolved by pointing the clients to the domain controller
- Group Policy was tested on the clients

---

### Step 11: Configured Local Administrator Access with Groups

**What I did:**  
I configured local administrator access using AD security groups and Group Policy Preferences.

**Why I did it:**  
This follows a more enterprise-style model where access is granted through groups instead of manually adding individual users to local administrator groups.

#### Server Local Admins

**GPO Name:**

```text
GPO_Server_Local_Admins
```

**Linked to:**

```text
_Servers
```

**GPO path:**

```text
Computer Configuration
→ Preferences
→ Control Panel Settings
→ Local Users and Groups
```

**Local group updated:**

```text
Administrators
```

**Group added:**

```text
GRP_Server_Admins
```

**Verification:**  

- `GRP_Server_Admins` was created
- GPO was created for server local administrator access
- Group-based admin access was tested

#### Workstation Local Admins

**GPO Name:**

```text
GPO_Workstation_Local_Admins
```

**Linked to:**

```text
_Workstations
```

**GPO path:**

```text
Computer Configuration
→ Preferences
→ Control Panel Settings
→ Local Users and Groups
```

**Group added:**

```text
GRP_Workstation_Admins
```

**Verification:**  

- `GRP_Workstation_Admins` was created
- GPO was created for workstation local administrator access
- Group-based admin access was tested

---

## Troubleshooting

### Issue: Server Was Not Renamed Before Installing Roles

**Problem:**  
I realized that I did not rename the server before giving it Active Directory roles.

**Cause:**  
The server should have been renamed to `DC01` before installing and promoting Active Directory Domain Services.

**Fix:**  

```powershell
Rename-Computer -NewName DC01 -Force -Restart
```

**Verification:**  

- Server restarted
- Server name changed to `DC01`

**Lesson learned:**  
Always rename the server before installing AD DS and promoting it to a domain controller.

---

### Issue: CLIENT02 Could Not Join the Domain

**Problem:**  
`CLIENT02` had issues joining the `corp.local` domain.

**Cause:**  
The client DNS was not set to the domain controller. It was likely using the NAT gateway or another DNS server instead of `DC01`.

**Fix:**  

```text
Preferred DNS: 192.168.58.10
Alternate DNS: Leave blank
```

**Verification:**  

- Client could find the domain
- `CLIENT02` successfully joined `corp.local`

**Lesson learned:**  
Active Directory depends heavily on DNS. Domain clients should use the domain controller as their DNS server.

---

### Issue: Wallpaper Appeared Black Instead of Showing the Image

**Problem:**  
The wallpaper policy applied, but the workstation showed a black background instead of the image.

**Cause:**  
The wallpaper file likely had the wrong file extension, such as:

```text
dragonball.jpg.jpg
```

**Fix:**  

On the domain controller, opened:

```text
C:\Windows\SYSVOL\sysvol\corp.local\scripts
```

Enabled file extensions in File Explorer:

```text
View
→ Show
→ File name extensions
```

Renamed the file to:

```text
dragonball.jpg
```

**Verification:**  

- File extension was corrected
- Wallpaper path matched the actual file name
- Wallpaper policy applied successfully after the fix

**Lesson learned:**  
Always show file extensions on servers to avoid mistakes with scripts, images, installers, certificates, and GPO files.

---

## Verification and Testing

The lab was verified using the following checks:

- Domain controller was renamed to `DC01`
- Static IP was configured on the domain controller
- DNS was set to the domain controller IP address
- Active Directory Domain Services was installed
- DNS Server role was installed
- DHCP Server role was installed
- DHCP scope was not configured
- Server was promoted to a domain controller
- New forest `corp.local` was created
- Organizational Units were created
- Test users were created
- Security groups were created and tested
- `CLIENT01` was renamed and joined to the domain
- `CLIENT02` was renamed and joined to the domain
- DNS issue on `CLIENT02` was identified and fixed
- Password policy GPO was tested
- Lock screen GPO was tested
- Disable USB storage GPO was tested
- Wallpaper enforcement GPO was tested
- Wallpaper GPO issue was troubleshot and fixed
- Admin groups were created and tested

---

## Security and Best Practices

- Do not document passwords in GitHub
- Do not document DSRM passwords publicly
- Use a static IP address for domain controllers
- Point domain clients to the domain controller for DNS
- Rename servers before installing AD DS
- Use Organizational Units to keep Active Directory organized
- Use security groups for permissions instead of assigning permissions directly to users
- Avoid adding users directly to built-in privileged groups when possible
- Use separate admin and standard accounts
- Link user policies to user OUs
- Link computer policies to computer OUs
- Keep file extensions visible on servers
- Use domain paths for SYSVOL resources instead of IP paths
- Apply password policies at the domain level
- Limit Domain Admin usage to domain controller administration

---

## Commands Used

### Rename Server

```powershell
Rename-Computer -NewName DC01 -Force -Restart
```

### Static IP Configuration

```text
IP Address: 192.168.58.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.58.2
Preferred DNS: 192.168.58.10
```

### Client DNS Fix

```text
Preferred DNS: 192.168.58.10
Alternate DNS: Leave blank
```

### Wallpaper Path

```text
\\corp.local\SYSVOL\corp.local\scripts\dragonball.jpg
```

---

## Folder Structure

```text
ActiveDirectory/
├── README.md
├── Configs/
├── Commands/
├── Diagrams/
└── Writeups/
```

### Configs

This folder contains cleaned configuration notes, such as GPO settings and domain configuration details.

### Commands

This folder contains commands used during the lab, such as PowerShell rename commands and useful verification commands.

### Diagrams

This folder contains network or Active Directory layout diagram notes.

### Writeups

This folder contains deeper explanations, troubleshooting notes, lessons learned, and reflections.

---

## What I Learned

This lab helped me understand how Active Directory depends on proper DNS configuration, organized OU structure, and careful planning before installing domain services. I learned why a domain controller should have a static IP address, why clients must use the domain controller for DNS, and why Group Policy needs to be linked to the correct domain or OU depending on whether the setting applies to users or computers.

I also learned practical troubleshooting skills, such as fixing domain join problems caused by incorrect DNS settings and fixing a wallpaper GPO issue caused by a hidden file extension problem.

This project gave me hands-on practice with real tasks that Junior System Administrators commonly perform in Windows environments.

---

## Resume Bullets

- Built a Windows Server 2022 Active Directory home lab with a domain controller, DNS, Organizational Units, users, groups, and Windows 11 domain-joined clients to practice core Junior System Administrator tasks.

- Configured and tested Group Policy Objects for password policy, workstation lock screen settings, USB storage restrictions, desktop wallpaper enforcement, and local administrator group management.

- Troubleshot Active Directory issues including client domain join failures caused by incorrect DNS settings and Group Policy wallpaper deployment problems caused by incorrect file extensions.

---

## Interview Explanation

In my Active Directory home lab, I built a small Windows domain environment using Windows Server 2022 Evaluation as a domain controller for the `corp.local` domain. I configured the server with a static IP address, installed Active Directory Domain Services, DNS, and DHCP roles, and promoted the server to a domain controller.

I created an organized OU structure for users, workstations, servers, admin accounts, service accounts, and groups. I also created test users and security groups to practice managing access the way a small business might.

I joined two Windows 11 Pro client machines, `CLIENT01` and `CLIENT02`, to the domain and configured Group Policy Objects for password requirements, lock screen timeout, USB storage restrictions, wallpaper enforcement, and local administrator access through security groups.

During the lab, I also troubleshot real issues, including a client that could not join the domain because DNS was not pointing to the domain controller, and a wallpaper GPO issue caused by a file extension mistake.

This project helped me practice core skills used in help desk, desktop support, and junior system administrator roles.
