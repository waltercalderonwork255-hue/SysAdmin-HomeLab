# Commands Used - Active Directory Lab

## Overview

This file documents the main commands and command-line settings used during the Active Directory lab.

Lab environment:

| Component | Details |
|---|---|
| Domain | `corp.local` |
| Domain Controller | `DC01` |
| Server OS | Windows Server 2022 Evaluation |
| Client OS | Windows 11 Pro |
| Network Type | VMware NAT |

---

## Rename Server

Used on the Windows Server before/after configuring Active Directory roles.

```powershell
Rename-Computer -NewName DC01 -Force -Restart
```

**Purpose:**  
Renamed the server to `DC01` so it had a clear domain controller hostname.

---

## Static IP Configuration for DC01

Configured manually in Windows network settings.

```text
IP Address: 192.168.58.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.58.2
Preferred DNS: 192.168.58.10
```

**Purpose:**  
The domain controller needs a static IP address so clients can consistently find DNS and Active Directory services.

---

## Client DNS Fix

Used on `CLIENT02` when it had trouble joining the domain.

```text
Preferred DNS: 192.168.58.10
Alternate DNS: Leave blank
```

**Purpose:**  
Domain-joined clients must use the domain controller as DNS. The NAT gateway should not be used as DNS for Active Directory clients.

---

## Domain Name Used

```text
corp.local
```

**Purpose:**  
This was the Active Directory domain name used for the lab.

---

## Domain Login Format

```text
corp\username
```

Example:

```text
corp\mjones
```

**Purpose:**  
Used to log into domain-joined Windows 11 clients with Active Directory user accounts.

---

## SYSVOL Wallpaper Path

```text
\\corp.local\SYSVOL\corp.local\scripts\dragonball.jpg
```

**Purpose:**  
Used for the desktop wallpaper Group Policy test.

**Note:**  
The image was only used for lab testing.

---

## Local SYSVOL Path on DC01

```text
C:\Windows\SYSVOL\sysvol\corp.local\scripts
```

**Purpose:**  
Used to store the wallpaper file referenced by Group Policy.

---

## Useful Verification Commands

These commands are useful for checking domain, DNS, user, and Group Policy status.

### Check Computer Name

```powershell
hostname
```

**Purpose:**  
Confirms the computer name, such as `DC01`, `CLIENT01`, or `CLIENT02`.

---

### Check IP and DNS Settings

```powershell
ipconfig /all
```

**Purpose:**  
Confirms IP address, gateway, and DNS settings.

---

### Force Group Policy Update

```powershell
gpupdate /force
```

**Purpose:**  
Forces the client to refresh Group Policy instead of waiting for the normal refresh interval.

---

### Check Applied Group Policy

```powershell
gpresult /r
```

**Purpose:**  
Shows which Group Policy Objects applied to the current user and computer.

---

### Check Current Logged-In User

```powershell
whoami
```

**Purpose:**  
Confirms which domain or local user is currently logged in.

---

### Check User Group Membership

```powershell
whoami /groups
```

**Purpose:**  
Shows the groups assigned to the logged-in user.

---

### Find Domain Controller

```powershell
nltest /dsgetdc:corp.local
```

**Purpose:**  
Checks whether the client can locate a domain controller for `corp.local`.

---

## Commands I Would Use for Future Testing

These commands are useful for future Active Directory troubleshooting and verification.

### Test DNS Resolution

```powershell
nslookup corp.local
```

**Purpose:**  
Checks whether the client can resolve the domain name.

---

### Test Domain Controller DNS Record

```powershell
nslookup dc01.corp.local
```

**Purpose:**  
Checks whether the client can resolve the domain controller hostname.

---

### Check Domain Trust / Secure Channel

```powershell
Test-ComputerSecureChannel
```

**Purpose:**  
Checks whether the domain-joined computer still has a working trust relationship with the domain.

---

### Repair Domain Trust Relationship

```powershell
Test-ComputerSecureChannel -Repair
```

**Purpose:**  
Can be used to repair a broken trust relationship between a client and the domain.

> Note: This should be run carefully and may require domain admin credentials.

---

## Notes

- Passwords are not documented in this file.
- DSRM recovery passwords are not documented in this file.
- The domain controller should use itself as DNS.
- Domain clients should use the domain controller as DNS.
- The NAT gateway should not be used as DNS for domain-joined clients.
- `gpupdate /force` and `gpresult /r` are useful when testing Group Policy.
