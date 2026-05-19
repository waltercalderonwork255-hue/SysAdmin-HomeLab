# Backup Lab Writeup

## Overview

This writeup explains what I did in my Backup and Recovery Lab, what problems I ran into, how I fixed them, and what I learned.

This lab was part of my SysAdmin-HomeLab project. The goal was to practice backup and recovery tasks that are useful for a Junior System Administrator role.

The lab included two main backup methods:

1. **Windows Server Backup** for DC01.
2. **Windows SMB shares mounted on Linux, then copied with rsync** for Client01 and Client02.

Sensitive information such as passwords, tokens, private keys, recovery keys, and real credentials are not included.

## Lab Goal

The goal of this lab was to build a basic backup and recovery workflow for my home lab.

I wanted to practice:

- Backing up a Windows Server domain controller
- Adding a dedicated backup disk
- Scheduling backups
- Creating Linux backup folders
- Backing up Windows client files to Linux
- Using SMB/CIFS mounts
- Using `rsync` for file copies
- Automating backups with cron
- Testing restore from backup
- Troubleshooting backup failures

## Why Backups Matter

Backups are important because systems can fail, files can be deleted, updates can break services, and users may need data restored.

For sysadmin work, it is not enough to only create a backup job. The backup must also be verified and tested. A backup is only useful if the data can actually be restored.

This lab helped me understand that backup administration includes:

- Planning what needs to be backed up
- Choosing the correct backup destination
- Scheduling backup jobs
- Protecting backup storage
- Reviewing backup results
- Testing recovery

## What I Built

In this lab, I built a backup setup with two parts.

### DC01 Backup

DC01 was backed up using **Windows Server Backup**.

The backup was configured as a **Full Server** backup and scheduled to run daily at **2:00 AM**.

A dedicated backup disk was added to the DC01 virtual machine and configured as:

```text
Drive Letter: E:
Volume Label: DC_Backups
File System: NTFS
Purpose: Dedicated Windows Server Backup destination
```

The Windows Server Backup job completed successfully.

### Client Backup

Client01 and Client02 were backed up using a Linux backup server.

The final working backup method was:

```text
Windows SMB share → mounted on Linux → rsync copy into /backups
```

The Windows client `Users` folders were shared over SMB. The Linux server mounted those shares and used `rsync` to copy the data into backup folders.

The Linux backup folders were:

```text
/backups/client01
/backups/client02
```

The Linux mount points were:

```text
/mnt/client01
/mnt/client02
```

Cron was used to schedule the Linux backup jobs.

## Windows Server Backup Process

I started by installing Windows Server Backup on DC01 through Server Manager.

The backup was configured as a full server backup because DC01 is the domain controller in the lab. A full server backup is useful because it can include important server components such as:

- Active Directory
- DNS
- SYSVOL
- System state
- Files
- Registry

After starting the backup schedule wizard, I selected:

```text
Backup type: Full Server
Schedule: Daily
Time: 2:00 AM
```

At first, Windows Server Backup showed an error because there was no available backup disk.

```text
No disks are available for use as a backup storage
```

To fix this, I added a new virtual disk to the DC01 virtual machine in VMware Workstation.

The disk was then initialized in Disk Management, formatted as NTFS, assigned drive letter `E:`, and labeled `DC_Backups`.

After that, I selected the disk as the backup destination and completed the backup schedule.

The backup completed successfully.

## Linux Backup Process

For the Linux backup server, I created backup folders for each Windows client.

```bash
sudo mkdir -p /backups/client01
sudo mkdir -p /backups/client02
```

I verified the folders with:

```bash
ls /backups
```

Then I installed `rsync`:

```bash
sudo apt update
sudo apt install rsync -y
```

I also installed and enabled cron so backup jobs could run automatically:

```bash
sudo apt update
sudo apt install cron
sudo systemctl start cron
sudo systemctl enable cron
```

The scheduled backup jobs were configured to run around 2:00 AM.

```bash
0 2 * * * rsync -av /mnt/client01/ /backups/client01/
15 2 * * * rsync -av /mnt/client02/ /backups/client02/
```

The cron jobs ran successfully.

## SMB Backup Method

The original idea was to use `rsync` over SSH from Linux to the Windows clients. SSH worked, but `rsync` did not work properly because Windows did not have `rsync` installed on the remote side.

The error was:

```text
rsync: command not found
rsync error code 12
```

Because of that, I changed the method.

Instead of using `rsync` directly over SSH, I used SMB/CIFS.

The Windows client `Users` folders were shared over SMB, then mounted on Linux.

Example mount format:

```bash
sudo mount -t cifs //192.168.58.132/Users /mnt/client01 -o username=USERNAME
sudo mount -t cifs //192.168.58.131/Users /mnt/client02 -o username=USERNAME
```

After the shares were mounted, I used `rsync` locally on Linux to copy the files into `/backups`.

```bash
rsync -av /mnt/client01/ /backups/client01/
rsync -av /mnt/client02/ /backups/client02/
```

This became the final working method for the Windows client backups.

## Restore Testing

Restore testing was completed successfully.

For the restore test, I confirmed that backup files existed, restored data from the backup location, and checked that the restored data was usable.

The restore process followed this basic flow:

```text
1. Confirm backup files existed.
2. Restore a file or folder from the backup.
3. Open or inspect the restored data.
4. Confirm the restored data matched the expected result.
```

This was an important part of the lab because it proved that the backup was not just created, but could actually be used for recovery.

## Problems I Ran Into

### Problem 1: Windows Server Backup Had No Available Disk

**Problem:**  
Windows Server Backup showed that no disks were available for backup storage.

**Cause:**  
DC01 did not have a dedicated backup disk attached yet.

**Fix:**  
I added a new virtual disk in VMware Workstation, initialized it in Disk Management, formatted it as NTFS, assigned drive letter `E:`, and labeled it `DC_Backups`.

**Result:**  
The disk became available as a backup destination and the backup completed successfully.

**Lesson learned:**  
Backup software needs a valid backup destination before a backup schedule can be completed.

### Problem 2: rsync Over SSH Failed on Windows

**Problem:**  
SSH worked from Linux to Windows, but `rsync` over SSH failed.

**Cause:**  
Windows did not have `rsync` installed on the remote side.

**Error:**  

```text
rsync: command not found
rsync error code 12
```

**Fix:**  
I switched to SMB/CIFS. I mounted the Windows `Users` share on Linux and then used local `rsync` from the mount point to the backup folder.

**Result:**  
The client backup worked using SMB mounted on Linux and copied with `rsync`.

**Lesson learned:**  
Windows and Linux use different backup methods. SMB is a better fit for accessing Windows file shares in this lab.

### Problem 3: Hostname Did Not Work but IP Address Worked

**Problem:**  
Mounting SMB shares by hostname did not work, but mounting by IP address worked.

**Cause:**  
DNS or hostname resolution was not fully working from Linux.

**Fix:**  
I used the IP address to verify that SMB worked, then worked on DNS cleanup separately.

**Result:**  
SMB mounting worked by IP address.

**Lesson learned:**  
Testing by IP address can help separate DNS problems from network or firewall problems.

### Problem 4: Linux DNS Settings Kept Reverting

**Problem:**  
Changes made directly to `/etc/resolv.conf` did not stay permanent.

**Cause:**  
Ubuntu manages DNS through `systemd-resolved`, so manual changes to `/etc/resolv.conf` can be overwritten.

**Fix:**  
I edited the proper resolver configuration file:

```text
/etc/systemd/resolved.conf
```

Example settings:

```text
DNS=192.168.58.10
Domains=corp.local
```

Then I restarted the resolver:

```bash
sudo systemctl restart systemd-resolved
```

**Lesson learned:**  
On modern Ubuntu systems, DNS should be configured through the correct service instead of manually editing `/etc/resolv.conf`.

### Problem 5: Windows Firewall Blocked Access

**Problem:**  
Linux could not reach the Windows clients at first.

**Cause:**  
Windows Firewall was blocking ICMP ping and SMB access.

**Fix:**  
I enabled the correct firewall rules for the Domain profile:

```text
File and Printer Sharing (Echo Request - ICMPv4-In)
File and Printer Sharing (SMB-In)
```

**Result:**  
Linux was able to ping and access the Windows SMB shares.

**Lesson learned:**  
Turning off the firewall may work for testing, but the better sysadmin practice is to enable only the required firewall rules.

## Verification

The following items were verified during the lab:

- Windows Server Backup was installed
- Windows Server Backup opened successfully
- A dedicated backup disk was added to DC01
- The backup disk was initialized and formatted
- The disk appeared as `E: DC_Backups`
- Windows Server Backup completed successfully
- Linux backup folders were created
- `rsync` was installed
- Cron was installed and enabled
- Cron jobs ran successfully
- Windows SMB shares mounted successfully on Linux
- Windows client files were visible from Linux
- `rsync` copied files into `/backups`
- Restore testing completed successfully

## Security Notes

No real passwords or sensitive information should be included in this GitHub repository.

The documentation should not include:

- Passwords
- Tokens
- Private keys
- Recovery keys
- API keys
- Real credentials

For documentation, placeholders should be used instead:

```text
USERNAME
PASSWORD
DOMAIN\USERNAME
username@corp.local
```

In a real environment, backup credentials should be stored securely and backup storage should be protected from unauthorized access.

## What I Learned

This lab helped me understand that backups require more than just copying files.

A complete backup workflow includes:

- Backup planning
- Backup storage setup
- Scheduling
- Permissions
- Networking
- Firewall rules
- Verification
- Restore testing

I also learned that backup methods depend on the system being backed up. Windows Server Backup was useful for DC01, while SMB and `rsync` worked better for backing up Windows client files to Linux.

The biggest lesson from this lab was that a backup is not complete until it has been successfully restored.

## Real-World Takeaways

In a real company, backups are usually handled with more advanced tools and policies, such as:

- Backup software
- Centralized backup storage
- Backup monitoring
- Backup logs and alerts
- Retention policies
- Offsite or cloud backups
- Regular restore testing

This home lab was not a production environment, but it helped me practice the core concepts behind backup and recovery.

## Resume-Friendly Summary

In this project, I configured a backup and recovery workflow for a Windows and Linux home lab. I used Windows Server Backup for a domain controller and used SMB/CIFS mounts with `rsync` to back up Windows client data to a Linux backup server. I also scheduled backup jobs with cron, troubleshot DNS and firewall issues, and completed restore testing to verify that the backups worked.

## Interview Explanation

In my backup lab, I practiced setting up both Windows Server and Linux-based backups. For my domain controller, I installed Windows Server Backup, added a dedicated virtual backup disk, formatted it as NTFS, and scheduled a full server backup for 2:00 AM.

For the Windows clients, I originally tested `rsync` over SSH, but it failed because Windows did not have `rsync` installed on the remote side. I changed the method to use SMB shares mounted on Linux, then used `rsync` to copy the files into `/backups`.

I also configured cron jobs so the backups could run automatically, fixed firewall and DNS issues, and completed restore testing. This helped me understand that backups need to be verified and restored before they can be trusted.
