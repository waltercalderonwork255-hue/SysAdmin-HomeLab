# GPO Settings - Active Directory Lab

## Overview

This file documents the Group Policy Objects and configuration settings used in my Active Directory home lab.

The lab environment used:

| Component | Details |
|---|---|
| Domain | `corp.local` |
| Domain Controller | `DC01` |
| Server OS | Windows Server 2022 Evaluation |
| Client OS | Windows 11 Pro |
| Network Type | VMware NAT |

---

## Domain Password Policy

**Purpose:**  
Enforce stronger password requirements for domain users.

**GPO Name:**  
`DomainPasswordPolicy`

**Linked To:**  
`corp.local`

**Configuration Path:**

```text
Computer Configuration
→ Policies
→ Windows Settings
→ Security Settings
→ Account Policies
→ Password Policy
```

**Settings Configured:**

```text
Minimum password length: 12
Password must meet complexity requirements: Enabled
Maximum password age: Not configured
```

**Verification:**  

- Password policy was configured at the domain level
- Policy was tested successfully

---

## Disable USB Storage

**Purpose:**  
Prevent removable USB storage access on domain-joined workstations.

**GPO Name:**  
`Disable USBStorage`

**Linked To:**  
`_Workstations`

**Configuration Path:**

```text
Computer Configuration
→ Policies
→ Administrative Templates
→ System
→ Removable Storage Access
```

**Setting Configured:**

```text
All Removable Storage classes: Deny all access
```

**Verification:**  

- GPO was linked to `_Workstations`
- Policy was tested successfully on domain-joined clients

---

## Lock Screen After 10 Minutes

**Purpose:**  
Automatically lock user sessions after 10 minutes of inactivity.

**Linked To:**  
`_Users`

**Configuration Path:**

```text
User Configuration
→ Policies
→ Administrative Templates
→ Control Panel
→ Personalization
```

**Settings Configured:**

```text
Enable screen saver
Screen saver timeout: 600 seconds
Password protect the screen saver
```

**Verification:**  

- Policy was configured under User Configuration
- GPO was linked to `_Users`
- Lock screen behavior was tested successfully

---

## Desktop Wallpaper Enforcement

**Purpose:**  
Apply a standard desktop wallpaper to domain users.

**Linked To:**  
`_Users`

**Configuration Path:**

```text
User Configuration
→ Policies
→ Administrative Templates
→ Desktop
→ Desktop
→ Desktop Wallpaper
```

**Wallpaper Path Used:**

```text
\\corp.local\SYSVOL\corp.local\scripts\dragonball.jpg
```

**Wallpaper Style:**  
Fill / Stretch

**Verification:**  

- Wallpaper policy was linked to `_Users`
- Wallpaper path was corrected
- Wallpaper policy was tested successfully after fixing the file extension issue

**Note:**  
The wallpaper image was only used for lab testing.

---

## Server Local Administrator Access

**Purpose:**  
Grant server local administrator access through an Active Directory security group.

**GPO Name:**  
`GPO_Server_Local_Admins`

**Linked To:**  
`_Servers`

**Configuration Path:**

```text
Computer Configuration
→ Preferences
→ Control Panel Settings
→ Local Users and Groups
```

**Local Group Updated:**

```text
Administrators
```

**Group Added:**

```text
GRP_Server_Admins
```

**Verification:**  

- `GRP_Server_Admins` was created
- GPO was linked to `_Servers`
- Group-based admin access was tested

---

## Workstation Local Administrator Access

**Purpose:**  
Grant workstation local administrator access through an Active Directory security group.

**GPO Name:**  
`GPO_Workstation_Local_Admins`

**Linked To:**  
`_Workstations`

**Configuration Path:**

```text
Computer Configuration
→ Preferences
→ Control Panel Settings
→ Local Users and Groups
```

**Local Group Updated:**

```text
Administrators
```

**Group Added:**

```text
GRP_Workstation_Admins
```

**Verification:**  

- `GRP_Workstation_Admins` was created
- GPO was linked to `_Workstations`
- Group-based admin access was tested

---

## Security Groups Used

| Group Name | Purpose |
|---|---|
| `GRP_Domain_Admins` | Custom group for domain admin access |
| `GRP_Server_Admins` | Local admin access on servers |
| `GRP_Workstation_Admins` | Local admin access on workstations |
| `GRP_Helpdesk_Admins` | Help desk admin group for practice |

---

## Best Practices Practiced

- Used groups instead of direct user permissions
- Linked computer policies to computer OUs
- Linked user policies to user OUs
- Applied password policy at the domain level
- Used domain paths instead of IP paths for SYSVOL resources
- Avoided documenting passwords or recovery passwords
- Tested GPO behavior on domain-joined Windows 11 clients
