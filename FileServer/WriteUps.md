# File Server Lab Writeups

## Overview

This folder contains the written explanations and reflections for my File Server and SMB/CIFS Lab.

The purpose of these writeups is to explain what I built, why I built it, what problems I ran into, what I learned, and how this project connects to real Junior System Administrator work.

This lab focused on:

- Windows file shares
- SMB/CIFS access
- Active Directory security groups
- Share permissions
- NTFS permissions
- Access-Based Enumeration
- Group Policy drive mapping
- Linux SMB/CIFS mounting
- Jellyfin media server access
- Permission testing and troubleshooting

---

# Writeup 1: Building Department-Based File Shares

## Summary

In this part of the lab, I created department-based file shares on `DC01` for HR, Sales, IT, and Executives. Each share was designed to simulate how a business might separate files by department.

The shares created were:

```text
HR
Sales
IT
Executives
Media
```

The department shares were used for business-style access control practice. The `Media` share was used for a personal Jellyfin media server setup.

---

## Why I Built This

File servers are common in business environments. Even though many companies use cloud storage, traditional file shares are still used for departments, shared drives, application data, and internal resources.

I built this lab to practice the type of file access control a Junior System Administrator may need to support, such as:

- Creating shared folders
- Controlling who can access each share
- Testing user permissions
- Troubleshooting access denied issues
- Mapping network drives for users
- Understanding how SMB/CIFS works

---

## What I Configured

The shared folders were created under:

```text
C:\Shares
```

Folder structure:

```text
C:\Shares
├── HR
├── Sales
├── IT
├── Executives
└── Media
```

Each department folder was shared over the network using SMB.

The shares could be accessed from Windows clients using:

```text
\\dc01
```

Direct share paths included:

```text
\\dc01\HR
\\dc01\Sales
\\dc01\IT
\\dc01\Executives
\\dc01\Media
```

---

## What I Learned

I learned that creating a shared folder is only one part of setting up a file server. The more important part is controlling access correctly.

A file server needs:

- A clear folder structure
- Proper share permissions
- Proper NTFS permissions
- Security groups
- Testing from client machines

I also learned that users should not all receive the same access. Each user should only receive the permissions they need for their role.

---

# Writeup 2: Using Security Groups for File Access

## Summary

In this part of the lab, I used Active Directory security groups to control access to file shares.

The groups created were:

```text
GRP_HR
GRP_Sales
GRP_IT
GRP_Executives
```

Users were added to the correct groups based on their department.

---

## Why Groups Matter

Permissions should be assigned to groups instead of directly to users.

For example, instead of giving one HR user access to the HR folder directly, the better method is:

```text
HR user → GRP_HR → HR share permissions
```

This makes administration easier because if a new HR user joins the company, the admin only needs to add that user to `GRP_HR`.

---

## Access Design

| User Type | Security Group | Share Access |
|---|---|---|
| HR user | `GRP_HR` | HR share |
| Sales user | `GRP_Sales` | Sales share |
| IT user | `GRP_IT` | IT share |
| Executive user | `GRP_Executives` | Executives share |
| Admin accounts | Admin group | Full Control |

---

## What I Learned

I learned that group-based access control is easier to manage and more realistic than assigning permissions directly to users.

This is related to Role-Based Access Control, also known as RBAC. In RBAC, access is assigned based on a user's role, not just their username.

Examples:

- HR users get HR access.
- Sales users get Sales access.
- IT users get IT access.
- Executive users get Executive access.
- Admin accounts manage the system.

This is a common concept in Windows Server, cloud platforms, and security systems.

---

# Writeup 3: Share Permissions vs NTFS Permissions

## Summary

One of the most important lessons from this lab was understanding the difference between share permissions and NTFS permissions.

Both permission types matter when users access folders over the network.

---

## Share Permissions

Share permissions apply when a user accesses a folder over the network.

Example network path:

```text
\\dc01\HR
```

Share permissions are configured from:

```text
Folder Properties → Sharing → Advanced Sharing → Permissions
```

---

## NTFS Permissions

NTFS permissions apply to the folder on the disk.

They are configured from:

```text
Folder Properties → Security
```

NTFS permissions apply whether the user accesses the folder locally or over the network.

---

## Most Restrictive Permission Wins

The key rule I learned is:

```text
The most restrictive permission wins.
```

Example:

| Share Permission | NTFS Permission | Effective Result |
|---|---|---|
| Full Control | Modify | Modify |
| Modify | Read Only | Read Only |
| Read Only | Modify | Read Only |
| Deny | Modify | Denied |

This means both permission areas need to be checked when troubleshooting access problems.

---

## How I Used Permissions

| Share | Regular User Permission | Admin Permission |
|---|---|---|
| HR | Modify | Full Control |
| Sales | Modify | Full Control |
| IT | Modify | Full Control |
| Executives | Read/Write | Full Control |
| Media | Read Only | Full Control |

---

## What I Learned

I learned that file permissions can be confusing because there are two layers of control.

When troubleshooting access denied issues, I should check:

1. Is the user in the correct security group?
2. Does the group have share permissions?
3. Does the group have NTFS permissions?
4. Is the user accessing the correct path?
5. Did Group Policy apply correctly?
6. Is the user logged in with the correct account?

---

# Writeup 4: Testing User Access

## Summary

After creating the shares and assigning permissions, I tested access from both `client01` and `client02`.

Testing was important because permissions need to be verified from the user's point of view.

---

## How I Tested

On the Windows clients, I opened Run:

```text
WIN + R
```

Then entered:

```text
\\dc01
```

I tested access using different users.

---

## Test Results

| Test | Result |
|---|---|
| Browse to `\\dc01` from `client01` | Successful |
| Browse to `\\dc01` from `client02` | Successful |
| HR user accessing HR | Successful |
| HR user accessing Sales | Denied |
| Sales user accessing Sales | Successful |
| Sales user accessing HR | Denied |
| Executive user accessing Executives | Successful |
| IT user accessing IT | Successful |
| Admin account access | Full Control |
| Media share access | Read-only access from Windows clients |

---

## File Action Testing

I also tested file actions inside the allowed shares.

| Action | Result |
|---|---|
| Create file | Successful in allowed shares |
| Edit file | Successful in allowed shares |
| Delete file | Successful in allowed shares |
| Access unauthorized share | Denied |

---

## What I Learned

I learned that it is not enough to only configure permissions on the server. I also need to test from a client machine using real user accounts.

This helped confirm that:

- Users could access only their assigned shares.
- Unauthorized access was blocked.
- Modify permissions allowed create, edit, and delete.
- Admin accounts had Full Control.
- Media access was read-only for Windows clients.

---

# Writeup 5: Access-Based Enumeration

## Summary

In this part of the lab, I enabled Access-Based Enumeration, also known as ABE.

ABE hides shares or folders that users do not have permission to access.

---

## Why ABE Is Useful

Without ABE, users may see shares they cannot open. This can cause confusion and may expose information about folder names.

With ABE enabled:

- HR users only see HR resources.
- Sales users only see Sales resources.
- Executive users only see Executive resources.

---

## Configuration Path

ABE was enabled from:

```text
Server Manager
→ File and Storage Services
→ Shares
→ Right-click share
→ Properties
→ Settings
→ Enable access-based enumeration
```

---

## Verification

| User Type | Result |
|---|---|
| HR user | Only saw HR resources |
| Sales user | Only saw Sales resources |
| Executive user | Only saw Executive resources |

---

## What I Learned

I learned that ABE is useful for both security and user experience.

It does not replace permissions, but it helps hide resources from users who do not need to see them.

This is useful in real environments because users only see the folders relevant to their role.

---

# Writeup 6: Group Policy Drive Mapping

## Summary

In this part of the lab, I used Group Policy to automatically map drives for users.

The drive mappings were based on Active Directory security group membership.

---

## Why Drive Mapping Matters

In many organizations, users do not manually type network paths like:

```text
\\dc01\HR
```

Instead, they receive mapped drives automatically when they log in.

Example:

```text
H: → HR share
S: → Sales share
```

This makes file access easier for users.

---

## Drive Mapping Design

| Department | Drive Letter | Network Path | Target Group |
|---|---|---|---|
| HR | `H:` | `\\dc01\HR` | `GRP_HR` |
| Sales | `S:` | `\\dc01\Sales` | `GRP_Sales` |
| IT | `I:` | `\\dc01\IT` | `GRP_IT` |
| Executives | `E:` | `\\dc01\Executives` | `GRP_Executives` |

---

## Configuration Path

The GPO was created in Group Policy Management.

GPO name:

```text
Drive Mapping Policy
```

GPO path:

```text
User Configuration
→ Preferences
→ Windows Settings
→ Drive Maps
```

Item-level targeting was used:

```text
Common
→ Item-level targeting
→ Security Group
```

---

## Verification

On the Windows clients, I ran:

```cmd
gpupdate /force
```

Expected results:

| User | Mapped Drive |
|---|---|
| HR user | `H:` |
| Sales user | `S:` |
| IT user | `I:` |
| Executive user | `E:` |

---

## What I Learned

I learned that Group Policy can automate common user settings, such as mapped drives.

I also learned that item-level targeting makes the drive mapping smarter because users only get the drives that match their group membership.

This is a realistic Junior SysAdmin skill because help desk and sysadmin teams often support mapped drive issues.

---

# Writeup 7: Linux SMB/CIFS Mount for Jellyfin

## Summary

In this part of the lab, I mounted a Windows SMB/CIFS share on Linux so Jellyfin could read media files from a Windows file share.

The Windows share was:

```text
\\dc01\Media
```

The Linux mount point was:

```text
/mnt/media
```

---

## Why I Built This

I wanted to connect a Windows file share to a Linux service. This helped me practice cross-platform file sharing.

This is useful because real environments often have Windows and Linux systems that need to interact.

---

## Mount Command

The Windows Media share was mounted using:

```bash
sudo mount -t cifs //192.168.58.10/Media /mnt/media -o username=YOUR_USERNAME,password=YOUR_PASSWORD
```

Real credentials should not be stored in GitHub. This documentation uses placeholders.

---

## Verification

After mounting the share, I verified it with:

```bash
ls /mnt/media
```

Expected folders:

```text
movies
tv
```

---

## What I Learned

I learned that Linux can mount Windows SMB shares using CIFS.

I also learned that Linux commands require exact syntax. Missing spaces caused errors during the lab.

Examples:

Incorrect:

```bash
sudomkdir-p /mnt/media
```

Correct:

```bash
sudo mkdir -p /mnt/media
```

Incorrect:

```bash
sudo mount-t cifs
```

Correct:

```bash
sudo mount -t cifs
```

---

# Writeup 8: Jellyfin Media Server

## Summary

This part of the lab used Jellyfin as a personal media server.

Jellyfin was installed on `LINUX01` and configured to read media from the mounted SMB/CIFS share.

---

## Jellyfin Access

Jellyfin was accessed from a web browser using:

```text
http://<LINUX_IP>:8096
```

Example lab URL:

```text
http://192.168.58.133:8096
```

The Jellyfin web page loaded successfully.

---

## Library Paths

The media libraries were configured as:

| Library Type | Path |
|---|---|
| Movies | `/mnt/media/movies` |
| TV | `/mnt/media/tv` |

---

## Client Testing

Jellyfin was tested from:

- `client01`
- `client02`

Phone and TV access were not tested.

---

## What I Learned

I learned how an application on Linux can use a mounted SMB/CIFS share as storage.

This helped me understand how file shares can support applications, not just users.

The Jellyfin part was mostly for personal use, but it still gave me practice with:

- Linux packages
- Repository setup
- Firewall ports
- Web access
- SMB/CIFS mounts
- Application storage paths

---

# Writeup 9: Troubleshooting Lessons

## Summary

This lab included several troubleshooting lessons.

The main issues were:

- Understanding Share vs NTFS permissions
- Verifying denied access
- Fixing Linux command syntax
- Correcting Jellyfin repository setup
- Remembering that DC01 should not be a production file server

---

## Issue 1: Share Permissions vs NTFS Permissions

**Problem:**  
Access can fail even when one permission setting looks correct.

**Cause:**  
Both share permissions and NTFS permissions apply.

**Fix:**  
Check both permission layers and test with the correct user.

**Lesson learned:**  
The most restrictive permission wins.

---

## Issue 2: Verifying Denied Access

**Problem:**  
It is easy to only test successful access and forget to test blocked access.

**Cause:**  
Permissions may appear correct, but unauthorized users must also be tested.

**Fix:**  
Test that HR is denied from Sales and Sales is denied from HR.

**Lesson learned:**  
A good permissions test includes both allowed access and denied access.

---

## Issue 3: Linux Command Syntax

**Problem:**  
Some Linux commands failed.

**Cause:**  
Spaces were missing in commands.

Examples:

```text
sudomkdir-p
sudo mount-t
sudo ufw allow8096
```

**Fix:**  
Corrected the syntax:

```bash
sudo mkdir -p /mnt/media
sudo mount -t cifs
sudo ufw allow 8096
```

**Lesson learned:**  
Linux commands require exact spacing.

---

## Issue 4: Jellyfin Repository Setup

**Problem:**  
There was an error adding the Jellyfin repository.

**Cause:**  
The repository line was typed incorrectly.

**Fix:**  
Edited the repo file:

```bash
sudo vim /etc/apt/sources.list.d/jellyfin.list
```

Corrected the repository line:

```text
deb [signed-by=/usr/share/keyrings/jellyfin.gpg] https://repo.jellyfin.org/ubuntu noble main
```

Then updated package lists:

```bash
sudo apt update
```

**Lesson learned:**  
Repository configuration files need exact syntax.

---

## Issue 5: File Server on Domain Controller

**Problem:**  
The file server was hosted on `DC01`.

**Cause:**  
This was done for lab simplicity.

**Fix:**  
Documented that this is okay for a home lab but not recommended in production.

**Lesson learned:**  
In real environments, Domain Controllers and file servers should usually be separate.

---

# Writeup 10: Real-World Best Practices

## Summary

This lab helped me understand several real-world file server best practices.

---

## Best Practices Practiced

| Best Practice | How I Practiced It |
|---|---|
| Use groups for permissions | Created `GRP_HR`, `GRP_Sales`, `GRP_IT`, and `GRP_Executives` |
| Apply least privilege | Users only accessed their own department shares |
| Reserve Full Control for admins | Admin accounts were the only ones with Full Control |
| Test denied access | HR was denied from Sales and Sales was denied from HR |
| Use ABE | Users only saw shares they had access to |
| Map drives with GPO | Used item-level targeting for department drives |
| Avoid secrets in docs | Replaced credentials with placeholders |
| Document limitations | Noted that DC01 as file server is not production best practice |

---

## Production Design Note

In this lab, `DC01` handled both domain services and file shares.

For a home lab, this is acceptable.

In production, a better design would be:

```text
DC01  → Active Directory, DNS, authentication
FS01  → File server and department shares
```

This separation improves:

- Security
- Performance
- Stability
- Troubleshooting
- Role separation

---

## What I Would Improve Next

If I continued improving this lab, I would add:

- A dedicated file server named `FS01`
- A backup plan for department shares
- Quotas for user or department storage
- File screening rules
- Auditing for file access
- A safer Linux mount using a credentials file instead of putting credentials directly in the command
- More testing from phone and TV for Jellyfin
- Screenshots for each major configuration

---

# Final Reflection

This file server project helped me practice a realistic Junior System Administrator task: managing access to shared folders.

I learned that file server administration is not just about creating folders. It includes planning access, creating groups, applying permissions, testing users, troubleshooting access denied problems, and documenting everything clearly.

The most important lessons I learned were:

- Use groups instead of direct user permissions.
- Share permissions and NTFS permissions both matter.
- The most restrictive permission wins.
- Test with real users from client machines.
- Verify denied access, not just successful access.
- Use Group Policy to simplify user access.
- Keep secrets out of documentation.
- A Domain Controller should not usually be a production file server.

This lab gave me hands-on practice with Windows Server, Active Directory, SMB/CIFS, Group Policy, Linux mounts, and Jellyfin.
