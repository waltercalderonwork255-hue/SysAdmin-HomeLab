# Troubleshooting Notes - Active Directory Lab

## Overview

This file documents the troubleshooting issues I ran into during my Active Directory home lab.

The lab was built using:

| Component | Details |
|---|---|
| Domain | `corp.local` |
| Domain Controller | `DC01` |
| Server OS | Windows Server 2022 Evaluation |
| Client OS | Windows 11 Pro |
| Network Type | VMware NAT |
| Clients | `CLIENT01`, `CLIENT02` |

The goal of this writeup is to document what went wrong, how I fixed it, how I verified the fix, and what I learned.

---

## Issue 1: Server Was Not Renamed Before Installing Roles

### Problem

I realized that I did not rename the server before giving it Active Directory roles.

The server should have been renamed to:

```text
DC01
```

before installing and promoting Active Directory Domain Services.

### Cause

I installed or started configuring roles before making sure the server had the correct hostname.

In an Active Directory environment, naming the domain controller clearly is important because the server name appears in tools, logs, DNS records, and documentation.

### Fix

I used PowerShell as Administrator to rename the server:

```powershell
Rename-Computer -NewName DC01 -Force -Restart
```

### Verification

After the server restarted, I verified that the computer name changed to:

```text
DC01
```

### Lesson Learned

Rename the server before installing Active Directory Domain Services and promoting it to a domain controller.

A clean naming standard makes the lab easier to manage, troubleshoot, and document.

---

## Issue 2: CLIENT02 Could Not Join the Domain

### Problem

`CLIENT02` had trouble joining the `corp.local` domain.

### Cause

The DNS server on `CLIENT02` was not pointing to the domain controller.

For Active Directory domain joins, the client must be able to find the domain controller through DNS. If the client uses the VMware NAT gateway, home router, or public DNS instead of the domain controller, it may not be able to locate `corp.local`.

### Fix

I manually set the DNS server on `CLIENT02` to the domain controller IP address:

```text
Preferred DNS: 192.168.58.10
Alternate DNS: Leave blank
```

The domain controller was:

```text
DC01 - 192.168.58.10
```

### Verification

After correcting the DNS settings:

- `CLIENT02` was able to find the `corp.local` domain
- `CLIENT02` successfully joined the domain
- The client was able to use domain authentication
- Group Policy testing could continue

### Lesson Learned

Active Directory depends heavily on DNS.

Domain-joined clients should use the domain controller as their DNS server. The NAT gateway should not be used as DNS for domain-joined clients.

---

## Issue 3: Wallpaper Appeared Black Instead of Showing the Image

### Problem

The wallpaper Group Policy applied, but the workstation showed a black background instead of the image.

### Cause

The wallpaper file likely had the wrong file extension.

The intended file name was:

```text
dragonball.jpg
```

but it may have been saved as:

```text
dragonball.jpg.jpg
```

Because file extensions were hidden in File Explorer, the mistake was not obvious at first.

### Fix

On the domain controller, I opened the SYSVOL scripts folder:

```text
C:\Windows\SYSVOL\sysvol\corp.local\scripts
```

Then I enabled file extensions in File Explorer:

```text
View
→ Show
→ File name extensions
```

After that, I corrected the wallpaper file name to:

```text
dragonball.jpg
```

The Group Policy wallpaper path used was:

```text
\\corp.local\SYSVOL\corp.local\scripts\dragonball.jpg
```

### Verification

After fixing the file extension:

- The wallpaper file name matched the GPO path
- The wallpaper policy applied successfully
- The workstation displayed the wallpaper correctly

### Lesson Learned

Always show file extensions on servers.

This helps avoid mistakes with:

- Images
- PowerShell scripts
- Installers
- Certificates
- GPO-related files

---

## Issue 4: Understanding Where to Link User and Computer GPOs

### Problem

Some Group Policy settings apply to users, while others apply to computers. At first, it was not obvious which OU each policy should be linked to.

### Cause

Group Policy has two main sections:

```text
Computer Configuration
User Configuration
```

Computer Configuration policies apply to computer objects.

User Configuration policies apply to user accounts.

### Fix

I linked policies based on what they affected.

User-based policies were linked to:

```text
_Users
```

Computer-based policies were linked to:

```text
_Workstations
```

or:

```text
_Servers
```

Examples:

| Policy | Configuration Type | Linked To |
|---|---|---|
| Lock screen after 10 minutes | User Configuration | `_Users` |
| Desktop wallpaper | User Configuration | `_Users` |
| Disable USB storage | Computer Configuration | `_Workstations` |
| Workstation local admins | Computer Configuration | `_Workstations` |
| Server local admins | Computer Configuration | `_Servers` |
| Domain password policy | Computer Configuration | `corp.local` |

### Verification

I tested the Group Policy Objects on the domain-joined clients and confirmed they applied successfully.

### Lesson Learned

Link GPOs based on whether the settings affect users or computers.

User Configuration policies should follow the user account.

Computer Configuration policies should follow the computer object.

---

## Issue 5: DHCP Role Installed but No Scope Configured

### Problem

The DHCP role was installed on the domain controller, but no DHCP scope was configured.

### Cause

The lab was using VMware NAT networking, and the lab did not require a custom DHCP scope at the time.

### Fix

No DHCP scope was configured.

The DHCP role was left installed for practice and documentation.

### Verification

The lab still worked because the important Active Directory requirement was DNS, not DHCP.

The clients were able to join the domain after using the correct DNS server:

```text
192.168.58.10
```

### Lesson Learned

DHCP and DNS are different services.

For Active Directory, DNS is critical. DHCP is useful for assigning IP addresses automatically, but the clients must still use the domain controller as DNS.

---

## Verification Commands for Troubleshooting

These commands are useful for troubleshooting Active Directory, DNS, and Group Policy issues.

### Check IP and DNS Settings

```powershell
ipconfig /all
```

Use this to confirm the client is using:

```text
DNS Server: 192.168.58.10
```

---

### Force Group Policy Update

```powershell
gpupdate /force
```

Use this after making Group Policy changes.

---

### Check Applied Group Policy

```powershell
gpresult /r
```

Use this to see which GPOs applied to the user and computer.

---

### Check Current User

```powershell
whoami
```

Use this to confirm whether the current login is a domain user or local user.

---

### Check Group Membership

```powershell
whoami /groups
```

Use this to confirm group membership for admin or access testing.

---

### Find the Domain Controller

```powershell
nltest /dsgetdc:corp.local
```

Use this to confirm the client can locate the domain controller.

---

### Test DNS Resolution

```powershell
nslookup corp.local
```

Use this to confirm the domain name resolves through DNS.

---

## Final Lessons Learned

This lab helped me understand that many Active Directory problems are connected to DNS, OU structure, or incorrect Group Policy linking.

The most important lessons I learned were:

- Rename servers before installing roles
- Use a static IP for the domain controller
- Point clients to the domain controller for DNS
- Do not use the NAT gateway as DNS for domain clients
- Link User Configuration GPOs to user OUs
- Link Computer Configuration GPOs to computer OUs
- Test GPOs on real domain-joined clients
- Keep file extensions visible on servers
- Do not document passwords or recovery passwords in GitHub

These troubleshooting notes show the mistakes I ran into, how I fixed them, and what I learned from the process.
