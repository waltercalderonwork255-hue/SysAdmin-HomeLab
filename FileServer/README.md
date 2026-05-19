# File Server and SMB/CIFS Lab

## Overview

This project documents a file server lab built as part of my SysAdmin-HomeLab portfolio. The purpose of this lab was to practice creating department-based file shares, applying permissions with Active Directory security groups, testing access from Windows client machines, and understanding how SMB/CIFS file sharing works in a Windows domain environment.

This lab also includes a personal media server component using Jellyfin on Linux. The media files were stored on a Windows SMB share and mounted on the Linux server so Jellyfin could access the media library.

For this lab, the file shares were hosted on `DC01`. In a real enterprise environment, a file server would usually be separate from the Domain Controller for better security, performance, and stability.

---

## Goal

The goal of this project was to create a realistic file sharing environment similar to what a Junior System Administrator might support in a business.

I wanted to practice:

- Creating department-based file shares
- Using Active Directory security groups for access control
- Configuring share permissions
- Configuring NTFS permissions
- Testing access from domain-joined Windows clients
- Verifying denied access for unauthorized users
- Enabling Access-Based Enumeration
- Mapping network drives with Group Policy
- Mounting a Windows SMB/CIFS share on Linux
- Connecting a mounted SMB share to Jellyfin
- Troubleshooting permissions, Linux commands, and repository setup issues

---

## Skills Demonstrated

- Windows file server administration
- SMB/CIFS file sharing
- NTFS permissions
- Share permissions
- Active Directory security groups
- Role-Based Access Control, also known as RBAC
- Access-Based Enumeration, also known as ABE
- Group Policy drive mapping
- Item-level targeting in Group Policy
- Windows client access testing
- Linux CIFS mounting
- Basic Jellyfin media server setup
- Permission troubleshooting
- Security best practices for file shares

---

## Environment

| Component | Details |
|---|---|
| File Server | `DC01` used as file server for lab purposes |
| Domain | `corp.local` |
| Client Machines | `client01` and `client02` |
| Linux Server | `LINUX01` |
| Media Server | Jellyfin |
| Share Names | `HR`, `Sales`, `IT`, `Executives`, `Media` |
| Permissions Model | Group-based permissions using Active Directory security groups |
| Authentication Method | Domain authentication for Windows shares; Linux CIFS mount for Jellyfin media access |

---

## File Share Design

The file share design was based on departments. Each department received its own shared folder, and access was controlled using Active Directory security groups.

The department shares created were:

- `HR`
- `Sales`
- `IT`
- `Executives`

The media share created was:

- `Media`

The access goal was:

- HR users should only access the HR share.
- Sales users should only access the Sales share.
- IT users should have Modify permissions to the IT share.
- Admin accounts should have Full Control.
- Executive users should access the Executives share.
- Windows clients should have read-only access to the Media share.
- Permissions should be assigned to groups instead of directly to users.

Security groups used:

- `GRP_HR`
- `GRP_Sales`
- `GRP_IT`
- `GRP_Executives`

Access was tested from both `client01` and `client02`.

---

## Project Steps

### Step 1: Created Security Groups

**What I did:**  
Created Active Directory security groups for department-based access.

Groups created:

```text
GRP_HR
GRP_Sales
GRP_IT
GRP_Executives
```

**Why I did it:**  
Permissions should be applied to groups instead of individual users. This makes permissions easier to manage and is closer to how access is handled in real organizations.

**Commands or settings used:**  

Created the groups in:

```text
Active Directory Users and Computers
```

Group settings:

| Setting | Value |
|---|---|
| Group Scope | Global |
| Group Type | Security |

Users were added based on department:

| User Type | Group |
|---|---|
| HR users | `GRP_HR` |
| Sales users | `GRP_Sales` |
| IT users | `GRP_IT` |
| Executive user | `GRP_Executives` |
| Admin accounts | Admin group / administrative access |

**Verification:**  
Confirmed that the correct users were members of the correct security groups.

---

### Step 2: Created Shared Folders on DC01

**What I did:**  
Created department folders on `DC01`.

Folders created:

```text
C:\Shares\HR
C:\Shares\Sales
C:\Shares\IT
C:\Shares\Executives
C:\Shares\Media
```

**Why I did it:**  
Each department needed its own location for shared files. The Media folder was created so Windows clients and Jellyfin could access media files from a central SMB share.

**Commands or settings used:**  
Created the folders manually on `DC01`.

**Verification:**  
Confirmed that the folders existed under:

```text
C:\Shares
```

---

### Step 3: Configured Share Permissions

**What I did:**  
Shared each folder using Windows Advanced Sharing.

Path used:

```text
Folder Properties → Sharing → Advanced Sharing
```

**Why I did it:**  
Share permissions control access over the network. Users need both share permissions and NTFS permissions to successfully access files.

**Commands or settings used:**  

Configured the following share permissions:

| Share | Group/User | Share Permission |
|---|---|---|
| `HR` | `GRP_HR` | Modify |
| `HR` | Admin accounts | Full Control |
| `Sales` | `GRP_Sales` | Modify |
| `Sales` | Admin accounts | Full Control |
| `IT` | `GRP_IT` | Modify |
| `IT` | Admin accounts | Full Control |
| `Executives` | `GRP_Executives` | Read/Write |
| `Executives` | Admin accounts | Full Control |
| `Media` | Windows clients | Read Only |

**Verification:**  
Confirmed the shares appeared when browsing to:

```text
\\dc01
```

---

### Step 4: Configured NTFS Permissions

**What I did:**  
Configured NTFS permissions from the Security tab of each folder.

Path used:

```text
Folder Properties → Security
```

**Why I did it:**  
NTFS permissions control file and folder access on the disk. Share permissions and NTFS permissions both apply when accessing files over the network.

The important rule learned:

```text
The most restrictive permission wins.
```

If either share permissions or NTFS permissions deny or limit access, the user may receive access denied.

**Commands or settings used:**  

HR folder:

| Group/User | Permission |
|---|---|
| SYSTEM | Kept default permissions |
| Administrators | Full Control |
| `GRP_HR` | Modify |

Sales folder:

| Group/User | Permission |
|---|---|
| SYSTEM | Kept default permissions |
| Administrators | Full Control |
| `GRP_Sales` | Modify |

IT folder:

| Group/User | Permission |
|---|---|
| SYSTEM | Kept default permissions |
| Administrators | Full Control |
| `GRP_IT` | Modify |

Executives folder:

| Group/User | Permission |
|---|---|
| SYSTEM | Kept default permissions |
| Administrators | Full Control |
| `GRP_Executives` | Read/Write |

Media folder:

| Group/User | Permission |
|---|---|
| SYSTEM | Kept default permissions |
| Administrators | Full Control |
| Windows clients | Read Only |

**Verification:**  
Tested access from both `client01` and `client02` using different users.

---

### Step 5: Tested File Share Access from Windows Clients

**What I did:**  
Tested access from both `client01` and `client02`.

On the client machines, I opened Run:

```text
WIN + R
```

Then entered:

```text
\\dc01
```

**Why I did it:**  
Testing from client machines confirms that the shares are available over the network and that permissions work as expected for real users.

**Commands or settings used:**  

Network path:

```text
\\dc01
```

Expected shares:

```text
HR
Sales
IT
Executives
Media
```

**Verification:**  

| Test | Result |
|---|---|
| Browsed to `\\dc01` from `client01` | Successful |
| Browsed to `\\dc01` from `client02` | Successful |
| HR user access | HR user could access HR share only |
| HR denied from Sales | Successful |
| Sales user access | Sales user could access Sales share only |
| Sales denied from HR | Successful |
| Executive user access | Executive user could access Executives share |
| IT user access | IT user had Modify access to IT share |
| Admin access | Admin accounts had Full Control |
| Media access | Windows clients could access media with read-only permissions |

---

### Step 6: Enabled Access-Based Enumeration

**What I did:**  
Enabled Access-Based Enumeration, also known as ABE.

**Why I did it:**  
ABE hides folders and shares that users do not have permission to access. This makes the environment cleaner and more secure from the user's perspective.

Example:

- HR user only sees HR resources.
- Sales user only sees Sales resources.
- Executive user only sees Executive resources.

**Commands or settings used:**  

Path used:

```text
Server Manager → File and Storage Services → Shares
```

Then:

```text
Right-click share → Properties → Settings → Enable Access-based enumeration
```

**Verification:**  
Verified that ABE worked. HR users only saw HR resources, Sales users only saw Sales resources, and the executive user only saw Executive resources.

---

### Step 7: Mapped Drives Automatically with Group Policy

**What I did:**  
Created a Group Policy Object to automatically map network drives for users.

**Why I did it:**  
Drive mapping is common in business environments. It makes shared folders easier for users to access without manually typing network paths.

**Commands or settings used:**  

Opened:

```text
Group Policy Management
```

Created a GPO linked to the `_Users` OU:

```text
Drive Mapping Policy
```

Edited the GPO:

```text
User Configuration
→ Preferences
→ Windows Settings
→ Drive Maps
```

Created mapped drives:

| Department | Drive Letter | Path | Target Group |
|---|---|---|---|
| HR | `H:` | `\\dc01\HR` | `GRP_HR` |
| Sales | `S:` | `\\dc01\Sales` | `GRP_Sales` |
| IT | `I:` | `\\dc01\IT` | `GRP_IT` |
| Executives | `E:` | `\\dc01\Executives` | `GRP_Executives` |

Item-level targeting was used:

```text
Common → Item-level targeting → Security Group
```

**Verification:**  

On the client machines:

```cmd
gpupdate /force
```

Expected result:

| User | Expected Drive |
|---|---|
| HR user | `H:` drive |
| Sales user | `S:` drive |
| IT user | `I:` drive |
| Executive user | `E:` drive |

Drive mapping was tested from Windows clients.

---

### Step 8: Tested File Creation, Editing, and Deletion

**What I did:**  
Tested creating, editing, and deleting files inside the shares.

**Why I did it:**  
A user being able to open a share does not always mean their permissions are correct. I needed to verify that Modify permissions worked as expected.

**Commands or settings used:**  
Tested file actions manually from Windows clients.

**Verification:**  

| Test | Result |
|---|---|
| HR file creation | Successful inside HR share |
| HR file editing | Successful inside HR share |
| HR file deletion | Successful inside HR share |
| Sales file creation | Successful inside Sales share |
| Sales file editing | Successful inside Sales share |
| Sales file deletion | Successful inside Sales share |
| Unauthorized access | HR was denied from Sales; Sales was denied from HR |
| Admin Full Control | Only admin accounts had Full Control |

---

### Step 9: Created Media Share for Jellyfin

**What I did:**  
Created a `Media` share on `DC01` for personal Jellyfin use.

Folder created:

```text
C:\Shares\Media
```

Share name:

```text
Media
```

**Why I did it:**  
This share was used to store movies and TV folders that could be mounted by the Linux Jellyfin server.

**Commands or settings used:**  

Media share permissions:

| Share | Permission |
|---|---|
| `Media` | Read Only for Windows clients |
| Admin accounts | Full Control |

**Verification:**  
Windows clients were able to access the Media share with read-only permissions to view videos.

---

### Step 10: Installed Jellyfin on LINUX01

**What I did:**  
Installed Jellyfin on `LINUX01`.

**Why I did it:**  
Jellyfin was used as a personal media server to stream media from the SMB/CIFS share.

**Commands or settings used:**  

Install dependencies:

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg
```

Add Jellyfin GPG key:

```bash
curl -fsSL https://repo.jellyfin.org/jellyfin_team.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/jellyfin.gpg
```

Add Jellyfin repository:

```bash
echo "deb [signed-by=/usr/share/keyrings/jellyfin.gpg] https://repo.jellyfin.org/ubuntu noble main" | sudo tee /etc/apt/sources.list.d/jellyfin.list
```

Update package list:

```bash
sudo apt update
```

Install Jellyfin:

```bash
sudo apt install jellyfin
```

**Verification:**  
The Jellyfin web page loaded successfully.

Jellyfin web interface:

```text
http://<LINUX_IP>:8096
```

Example from lab notes:

```text
http://192.168.58.133:8096
```

---

### Step 11: Mounted Windows SMB Share on Linux

**What I did:**  
Mounted the Windows `Media` share from `DC01` onto `LINUX01`.

**Why I did it:**  
Jellyfin needed access to the media files stored on the Windows file share.

**Commands or settings used:**  

Create mount point:

```bash
sudo mkdir -p /mnt/media
```

Mount SMB/CIFS share:

```bash
sudo mount -t cifs //192.168.58.10/Media /mnt/media -o username=YOUR_USERNAME,password=YOUR_PASSWORD
```

Important: Do not store real passwords in GitHub documentation. Use placeholders such as `YOUR_USERNAME` and `YOUR_PASSWORD`.

Check mount:

```bash
ls /mnt/media
```

Expected folders:

```text
movies
tv
```

Example media folder structure:

```text
Media/
├── movies/
│   └── Interstellar (2014).mp4
└── tv/
    └── Breaking Bad/
        └── Season 1/
```

**Verification:**  
Confirmed that `/mnt/media` showed the expected media folders.

---

### Step 12: Added Media Libraries in Jellyfin

**What I did:**  
Added media folders to Jellyfin.

**Why I did it:**  
Jellyfin needs library paths so it knows where movies and TV shows are stored.

**Commands or settings used:**  

In the Jellyfin web interface:

```text
Dashboard → Libraries → Add Library
```

Libraries added:

| Library Type | Path |
|---|---|
| Movies | `/mnt/media/movies` |
| TV | `/mnt/media/tv` |

Server name used:

```text
Corp Local Media
```

**Verification:**  
The Jellyfin page loaded correctly, and `client01` and `client02` were able to access Jellyfin.

Phone and TV access were not tested.

---

## Permissions Plan

| Share | Group/User | Permission | Reason |
|---|---|---|---|
| `HR` | `GRP_HR` | Modify | Allows HR users to create, edit, and delete files in the HR share |
| `HR` | Admin accounts | Full Control | Allows administrators to fully manage the share |
| `Sales` | `GRP_Sales` | Modify | Allows Sales users to create, edit, and delete files in the Sales share |
| `Sales` | Admin accounts | Full Control | Allows administrators to fully manage the share |
| `IT` | `GRP_IT` | Modify | Allows IT users to work with IT files without giving full administrative control |
| `IT` | Admin accounts | Full Control | Allows administrators to fully manage the share |
| `Executives` | `GRP_Executives` | Read/Write | Allows executive users to access and update executive files |
| `Executives` | Admin accounts | Full Control | Allows administrators to fully manage the share |
| `Media` | Windows clients | Read Only | Allows clients to view videos without modifying media files |
| `Media` | Admin accounts | Full Control | Allows administrators to manage the media files |

---

## Troubleshooting

### Issue: Understanding Share Permissions vs NTFS Permissions

**Problem:**  
Access can still fail even if one permission area looks correct.

**Cause:**  
Both share permissions and NTFS permissions apply when accessing files over the network.

**Fix:**  
Configured both share permissions and NTFS permissions correctly.

**Verification:**  
Tested access from `client01` and `client02` using users from different groups.

**Lesson learned:**  
The most restrictive permission wins. If share permissions allow access but NTFS permissions block access, the user will still be denied.

---

### Issue: Verifying User Access

**Problem:**  
I needed to confirm that users could only access the shares they were supposed to access.

**Cause:**  
File share permissions can be confusing because both share permissions and NTFS permissions apply.

**Fix:**  
Tested access using different users from `client01` and `client02`.

**Verification:**  
HR users could access only the HR share and were denied from Sales. Sales users could access only the Sales share and were denied from HR. The executive user could access the Executives share. Access-Based Enumeration worked, so users only saw the shares they had permission to access.

**Lesson learned:**  
Testing with real user accounts is important because permissions may look correct on the server but still need to be verified from the client side.

---

### Issue: Jellyfin Repository Command Error

**Problem:**  
There was an error while adding the Jellyfin repository.

**Cause:**  
The repository command or repository line was typed incorrectly.

**Fix:**  
Opened the repository file:

```bash
sudo vim /etc/apt/sources.list.d/jellyfin.list
```

Corrected the repository line:

```text
deb [signed-by=/usr/share/keyrings/jellyfin.gpg] https://repo.jellyfin.org/ubuntu noble main
```

Updated package list again:

```bash
sudo apt update
```

**Verification:**  
The package list updated successfully, and Jellyfin installed correctly.

**Lesson learned:**  
Linux repository files are sensitive to exact syntax. Small spacing or spelling mistakes can break `apt update`.

---

### Issue: SMB/CIFS Mount Syntax Errors

**Problem:**  
Some Linux commands did not work correctly at first.

**Cause:**  
Several commands were missing spaces, such as:

```text
sudomkdir-p
sudo mount-t
sudo ufw allow8096
```

**Fix:**  
Corrected the commands:

```bash
sudo mkdir -p /mnt/media
sudo mount -t cifs //192.168.58.10/Media /mnt/media -o username=YOUR_USERNAME,password=YOUR_PASSWORD
sudo ufw allow 8096
```

**Verification:**  
Ran:

```bash
ls /mnt/media
```

Confirmed the mounted share showed media folders.

**Lesson learned:**  
Linux commands require exact spacing. A missing space can cause the shell to treat the command as invalid.

---

### Issue: File Server Running on Domain Controller

**Problem:**  
The file server was running on `DC01`.

**Cause:**  
This was done for lab simplicity.

**Fix:**  
Documented that this is acceptable for a home lab, but not best practice in a real enterprise.

**Verification:**  
The lab worked for testing file shares, permissions, drive mapping, and SMB/CIFS access.

**Lesson learned:**  
In real environments, a Domain Controller should usually only handle AD DS, DNS, and authentication. File servers should normally be separate servers for security, performance, and stability.

---

## Verification and Testing

| Test | Result |
|---|---|
| Browsed to `\\dc01` from `client01` | Successful |
| Browsed to `\\dc01` from `client02` | Successful |
| HR user access | HR user could access HR share only |
| HR denied from Sales | Successful |
| Sales user access | Sales user could access Sales share only |
| Sales denied from HR | Successful |
| Executive user access | Executive user could access Executives share |
| IT user access | IT users had Modify permissions |
| Admin access | Only admin accounts had Full Control |
| Access-Based Enumeration | Verified; users only saw shares they had access to |
| HR file test | HR user could create, edit, and delete files inside HR share |
| Sales file test | Sales user could create, edit, and delete files inside Sales share |
| Media share access | Windows clients could access media with read-only permissions |
| Jellyfin web page | Loaded successfully |
| Jellyfin access from `client01` | Successful |
| Jellyfin access from `client02` | Successful |
| Phone Jellyfin access | Not tested |
| TV Jellyfin access | Not tested |

---

## Security and Best Practices

This lab demonstrated several important file server security concepts:

- Use groups instead of assigning permissions directly to users.
- Apply least privilege so users only access what they need.
- Give regular users Modify permissions when they need to create, edit, and delete files.
- Reserve Full Control for admin accounts.
- Separate share permissions and NTFS permissions clearly.
- Remember that the most restrictive permission wins.
- Use Access-Based Enumeration to hide shares or folders users cannot access.
- Avoid storing passwords, tokens, private keys, or secrets in documentation.
- Test access with multiple user accounts.
- Avoid using overly broad permissions in real environments.
- Use a separate file server instead of running file services directly on a Domain Controller in production.

---

## Commands Used

### Windows Network Access

```text
\\dc01
```

### Group Policy Update

```cmd
gpupdate /force
```

### Jellyfin Install Commands

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg
curl -fsSL https://repo.jellyfin.org/jellyfin_team.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/jellyfin.gpg
echo "deb [signed-by=/usr/share/keyrings/jellyfin.gpg] https://repo.jellyfin.org/ubuntu noble main" | sudo tee /etc/apt/sources.list.d/jellyfin.list
sudo apt update
sudo apt install jellyfin
```

### Edit Jellyfin Repository File

```bash
sudo vim /etc/apt/sources.list.d/jellyfin.list
```

Correct repository line:

```text
deb [signed-by=/usr/share/keyrings/jellyfin.gpg] https://repo.jellyfin.org/ubuntu noble main
```

### Create Linux Mount Point

```bash
sudo mkdir -p /mnt/media
```

### Mount SMB/CIFS Share

```bash
sudo mount -t cifs //192.168.58.10/Media /mnt/media -o username=YOUR_USERNAME,password=YOUR_PASSWORD
```

### Verify Mounted Share

```bash
ls /mnt/media
```

### Allow Jellyfin Firewall Port

```bash
sudo ufw allow 8096
```

### Jellyfin Web Access

```text
http://<LINUX_IP>:8096
```

Example from lab notes:

```text
http://192.168.58.133:8096
```

---

## Screenshots to Add Later

| Screenshot | What to Capture | Why It Matters | Suggested Filename |
|---|---|---|---|
| AD Security Groups | `GRP_HR`, `GRP_Sales`, `GRP_IT`, `GRP_Executives` in Active Directory | Shows group-based access control | `ad-security-groups.png` |
| Shared Folders | `C:\Shares\HR`, `C:\Shares\Sales`, `C:\Shares\IT`, `C:\Shares\Executives`, `C:\Shares\Media` | Shows file server folder structure | `shared-folders.png` |
| Share Permissions | HR, Sales, IT, Executives, or Media share permissions | Shows network-level permissions | `share-permissions.png` |
| NTFS Permissions | Security tab for HR, Sales, IT, or Executives folder | Shows file-level permissions | `ntfs-permissions.png` |
| Access-Based Enumeration | ABE enabled in share settings | Shows real-world file server configuration | `access-based-enumeration.png` |
| Client Share Access | Client browsing to `\\dc01` | Shows network access from client machine | `client-dc01-shares.png` |
| Drive Mapping GPO | Drive Maps section in Group Policy | Shows automated drive mapping | `gpo-drive-mapping.png` |
| Mapped Drive on Client | H:, S:, I:, or E: mapped drive | Shows successful GPO application | `mapped-drive-client.png` |
| Denied Access Test | HR denied from Sales or Sales denied from HR | Shows least privilege working | `denied-access-test.png` |
| Linux CIFS Mount | `ls /mnt/media` output | Shows Windows share mounted on Linux | `linux-cifs-mount.png` |
| Jellyfin Dashboard | Jellyfin library page | Shows media server setup | `jellyfin-dashboard.png` |
| Jellyfin Library Paths | `/mnt/media/movies` and `/mnt/media/tv` | Shows media library configuration | `jellyfin-library-paths.png` |

---

## What I Learned

In this project, I learned how Windows file shares use both share permissions and NTFS permissions. I also learned why permissions should be assigned to security groups instead of individual users. This makes permissions easier to manage and closer to how real businesses control access.

I learned that Modify permissions allow users to create, edit, and delete files without giving them Full Control. Full Control should be reserved for admin accounts because it gives more administrative power over the folder.

I also learned that Access-Based Enumeration helps users only see the shares they are allowed to access, which makes the environment cleaner and reduces confusion.

Another important lesson was that using a Domain Controller as a file server is acceptable for a small lab, but it is not best practice in production. In a real organization, the Domain Controller should focus on authentication, DNS, and Active Directory, while file services should run on a separate server.

The Linux and Jellyfin portion helped me understand how SMB/CIFS shares can be mounted from Linux and used by applications like Jellyfin.

---

## Resume Bullets

- Built a Windows file sharing lab using Active Directory security groups, NTFS permissions, and share permissions to control department-based access for HR, Sales, IT, and Executives.
- Configured and tested Access-Based Enumeration and Group Policy drive mappings so users only saw and accessed the shares assigned to their security groups.
- Verified file server permissions from multiple Windows clients by testing read, write, edit, delete, and denied-access scenarios for different user roles.

---

## Interview Explanation

In my file server lab, I created department-based shares for HR, Sales, IT, and Executives using Windows Server file sharing. I used Active Directory security groups instead of assigning permissions directly to users, which follows a more realistic business approach.

I configured both share permissions and NTFS permissions, then tested access from `client01` and `client02` using different users. HR users could only access HR, Sales users could only access Sales, and the executive user could access the Executives share. I also verified denied access, such as HR being blocked from Sales and Sales being blocked from HR.

I configured Access-Based Enumeration so users only saw the shares they had permission to access. I also practiced drive mapping through Group Policy so users could automatically receive the correct network drive based on their group membership.

One important thing I learned was that share permissions and NTFS permissions both matter, and the most restrictive permission wins. I also learned that Modify permissions are useful for normal users who need to create, edit, and delete files, while Full Control should be reserved for admin accounts.

I also added a Linux and Jellyfin component where I mounted a Windows SMB/CIFS media share on Linux and used it as a media library path in Jellyfin. The Jellyfin web page loaded successfully, and both Windows clients were able to access it.
