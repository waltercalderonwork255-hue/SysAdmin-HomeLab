# Backup Lab Configurations

## Overview

This document contains the configuration details for the Backup and Recovery Lab in the SysAdmin-HomeLab repository.

The purpose of this file is to document the backup configuration settings used for:

- Windows Server Backup on DC01
- Dedicated backup disk setup
- Linux backup server folder structure
- SMB/CIFS mounted Windows client backups
- rsync backup configuration
- Cron scheduled backup jobs
- DNS settings
- Firewall rules
- Restore testing

Sensitive information such as passwords, tokens, private keys, recovery keys, and real credentials are not included.

## Windows Server Backup Configuration

| Setting | Configuration |
|---|---|
| Server | DC01 |
| Server Role | Domain Controller |
| Backup Tool | Windows Server Backup |
| Backup Type | Full Server |
| Backup Schedule | Daily |
| Backup Time | 2:00 AM |
| Backup Destination Type | Dedicated backup disk |
| Backup Disk Letter | `E:` |
| Backup Disk Label | `DC_Backups` |
| Backup Status | Completed successfully |
| Restore Test | Completed successfully |

## Windows Server Backup Scope

The Windows Server Backup job was configured as a **Full Server** backup.

The full server backup included:

- Active Directory
- DNS
- SYSVOL
- System state
- Files
- Registry
- Server configuration data

This was used because DC01 is the domain controller in the lab and needed its own Windows Server backup process.

## Windows Server Backup Wizard Settings

| Wizard Setting | Selected Option |
|---|---|
| Backup Configuration | Full Server |
| Backup Frequency | Once a day |
| Backup Time | 2:00 AM |
| Destination Type | Back up to a hard disk dedicated for backups |
| Destination Disk | `E:` |
| Volume Label | `DC_Backups` |

## VMware Backup Disk Configuration

A dedicated virtual disk was added to the DC01 virtual machine for backup storage.

| Setting | Configuration |
|---|---|
| Virtualization Platform | VMware Workstation |
| Virtual Machine | DC01 |
| Disk Controller | SCSI |
| Disk Type | Create a new virtual disk |
| Disk Size | 40–60 GB |
| Disk Storage Option | Store virtual disk as a single file |
| Disk File Name | `Windows Server 2022_DC01_BackupDisk.vmdk` |
| Purpose | Dedicated Windows Server Backup destination |

## Windows Disk Management Configuration

After the new virtual disk was added to DC01, it was initialized and formatted in Windows Disk Management.

| Setting | Configuration |
|---|---|
| Disk Number | Disk 1 |
| Initial Status | Unknown / Not Initialized |
| Partition Style | GPT |
| Volume Type | New Simple Volume |
| File System | NTFS |
| Drive Letter | `E:` |
| Volume Label | `DC_Backups` |
| Purpose | Dedicated backup storage for DC01 |

## Linux Backup Server Configuration

| Setting | Configuration |
|---|---|
| Linux Server | LINUX01 |
| Backup Role | Backup storage server for Windows clients |
| Backup Method | Windows SMB share mounted on Linux, then copied with `rsync` |
| Backup Root Folder | `/backups` |
| Client01 Backup Folder | `/backups/client01` |
| Client02 Backup Folder | `/backups/client02` |
| Mount Point for Client01 | `/mnt/client01` |
| Mount Point for Client02 | `/mnt/client02` |
| Automation Tool | cron |
| Backup Schedule | Daily around 2:00 AM |
| Restore Test | Completed successfully |

## Linux Backup Folder Structure

```text
/backups
├── client01
└── client02
```

## Linux Backup Folder Purpose

| Folder | Purpose |
|---|---|
| `/backups/client01` | Stores backed up files from Client01 |
| `/backups/client02` | Stores backed up files from Client02 |

## Linux Backup Folder Creation

```bash
sudo mkdir -p /backups/client01
sudo mkdir -p /backups/client02
```

## Linux Backup Folder Verification

```bash
ls /backups
```

Expected output:

```text
client01
client02
```

## Linux Backup Folder Permissions

```bash
sudo chmod -R 755 /backups
```

## Linux Backup Folder Permission Note

The `/backups` directory was configured with basic permissions for the lab.

In a real environment, backup folders should use stricter permissions and be limited to backup administrators or dedicated backup service accounts.

## Final Client Backup Method

The final working backup method for the Windows clients was:

```text
Windows SMB share → mounted on Linux → rsync copy into /backups
```

This was used because `rsync` over SSH did not work properly with the Windows clients. Windows did not have `rsync` installed on the remote side, so SMB/CIFS was used instead.

## Windows Client SMB Share Configuration

The Windows client `Users` folder was shared over SMB.

| Setting | Configuration |
|---|---|
| Share Location | `C:\Users` |
| Share Name | `Users` |
| Access Method | SMB/CIFS |
| Firewall Rule Required | File and Printer Sharing SMB-In |
| Firewall Profile | Domain |
| Authentication | Domain/user credentials |
| Passwords Stored in Repository | No |

## Client SMB Mount Points

| Client | SMB Share | Linux Mount Point | Linux Backup Destination |
|---|---|---|---|
| Client01 | `//192.168.58.132/Users` | `/mnt/client01` | `/backups/client01` |
| Client02 | `//192.168.58.131/Users` | `/mnt/client02` | `/backups/client02` |

## SMB Mount Commands

```bash
sudo mount -t cifs //192.168.58.132/Users /mnt/client01 -o username=USERNAME
sudo mount -t cifs //192.168.58.131/Users /mnt/client02 -o username=USERNAME
```

## SMB Mount Verification

```bash
ls /mnt/client01
ls /mnt/client02
```

Expected example output:

```text
Public
Default
USERNAME
```

## SMB Authentication Note

Real passwords should not be included in documentation or committed to GitHub.

Use placeholders such as:

```text
USERNAME
PASSWORD
DOMAIN\USERNAME
username@corp.local
```

## rsync Configuration

After the Windows SMB shares were mounted on Linux, `rsync` was used to copy files from the mounted share into the backup folders.

## Client01 rsync Backup

```bash
rsync -av /mnt/client01/ /backups/client01/
```

## Client02 rsync Backup

```bash
rsync -av /mnt/client02/ /backups/client02/
```

## Optional Mirror Mode

```bash
rsync -av --delete /mnt/client01/ /backups/client01/
rsync -av --delete /mnt/client02/ /backups/client02/
```

## Mirror Mode Warning

The `--delete` option makes the backup destination match the source.

This can be useful for keeping an exact mirror, but it can also delete files from the backup folder if they are deleted from the source. This option should be used carefully.

## Cron Configuration

Cron was used to schedule Linux backup jobs.

## Cron Service Configuration

```bash
sudo apt update
sudo apt install cron
sudo systemctl start cron
sudo systemctl enable cron
```

## Cron Service Verification

```bash
systemctl status cron
```

## Cron Job Editor

```bash
crontab -e
```

## Cron Backup Schedule

```bash
0 2 * * * rsync -av /mnt/client01/ /backups/client01/
15 2 * * * rsync -av /mnt/client02/ /backups/client02/
```

## Cron Schedule Explanation

| Cron Time | Meaning | Backup Job |
|---|---|---|
| `0 2 * * *` | Runs daily at 2:00 AM | Client01 backup |
| `15 2 * * *` | Runs daily at 2:15 AM | Client02 backup |

## Cron Job Verification

```bash
crontab -l
```

## DNS Configuration

Linux DNS was configured to use the domain controller for lab name resolution.

| Setting | Configuration |
|---|---|
| Domain | `corp.local` |
| DNS Server | `192.168.58.10` |
| DNS Server Role | DC01 domain controller |
| Linux DNS Method | `systemd-resolved` |
| Resolver Config File | `/etc/systemd/resolved.conf` |

## DNS Settings

```text
DNS=192.168.58.10
Domains=corp.local
```

## DNS Configuration File

```text
/etc/systemd/resolved.conf
```

## Restart DNS Resolver

```bash
sudo systemctl restart systemd-resolved
```

## DNS Verification

```bash
ping client01
ping client02
```

## DNS Troubleshooting Note

Hostname resolution needed cleanup during the lab. Mounting by IP address worked when hostname resolution did not.

Testing by IP address helped separate DNS issues from network or SMB issues.

## Windows Firewall Configuration

Windows Firewall rules were enabled to allow backup-related communication from the Linux backup server.

## ICMP Firewall Rule

| Setting | Configuration |
|---|---|
| Rule Name | File and Printer Sharing Echo Request ICMPv4-In |
| Profile | Domain |
| Purpose | Allow ping testing from Linux |
| Status | Enabled |

## SMB Firewall Rule

| Setting | Configuration |
|---|---|
| Rule Name | File and Printer Sharing SMB-In |
| Profile | Domain |
| Purpose | Allow Linux to access Windows SMB shares |
| Status | Enabled |

## SSH Firewall Rule

OpenSSH Server was tested but was not used as the final backup method.

| Setting | Configuration |
|---|---|
| Rule Name | OpenSSH Server `sshd` |
| Port | 22 |
| Profile | Domain |
| Purpose | SSH testing from Linux |
| Result | SSH worked, but `rsync` over SSH failed because Windows did not have `rsync` available |

## OpenSSH Server Test Configuration

OpenSSH Server was installed and tested on a Windows client.

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
Get-Service sshd
```

## SSH Test from Linux

```bash
ssh username@client02
```

## Failed rsync Over SSH Test

```bash
rsync -avn username@client02:/Users/username/ /backups/client02/
rsync -av username@client02:/Users/username/ /backups/client02/
```

## rsync Over SSH Error

```text
rsync: command not found
rsync error code 12
```

## rsync Over SSH Result

`rsync` over SSH was not used as the final backup method because Windows did not have `rsync` available on the remote side.

The final method was changed to SMB/CIFS mounted on Linux, followed by local `rsync` into `/backups`.

## smbclient Configuration

`smbclient` was installed for SMB troubleshooting.

```bash
sudo apt update
sudo apt install smbclient
```

## smbclient Test Command

```bash
smbclient -L //192.168.58.131 -U username@corp.local
```

## Unmount Configuration

If the wrong SMB share was mounted to the wrong mount point, the share was unmounted and checked.

```bash
sudo umount /mnt/client02
mount | grep client02
```

## Restore Testing Configuration

Restore testing was completed successfully.

| Setting | Result |
|---|---|
| Backup files existed | Verified |
| Restore performed | Completed |
| Restored data checked | Completed |
| Restore result | Successful |

## Restore Test Process

```text
1. Confirm files existed in the backup location.
2. Restore a file or folder from the backup.
3. Open or check the restored data.
4. Confirm the restored data matched the expected result.
```

## Security Configuration Notes

- Real passwords were not included in this documentation.
- Tokens were not included in this documentation.
- Private keys were not included in this documentation.
- Recovery keys were not included in this documentation.
- SMB commands use placeholders instead of real credentials.
- Backup storage should be protected from unauthorized access.
- Backup jobs should be monitored.
- Backup logs should be reviewed.
- Restore testing should be performed regularly.
- Firewall rules should be limited to required services only.
- SMB access should only be allowed where needed.
- Backup credentials should be stored securely outside of GitHub documentation.

## Final Configuration Summary

| Area | Final Configuration |
|---|---|
| DC01 Backup Method | Windows Server Backup |
| DC01 Backup Type | Full Server |
| DC01 Backup Destination | Dedicated `E:` drive labeled `DC_Backups` |
| DC01 Backup Schedule | Daily at 2:00 AM |
| Client Backup Method | SMB share mounted on Linux, then copied with `rsync` |
| Client01 Mount Point | `/mnt/client01` |
| Client02 Mount Point | `/mnt/client02` |
| Client01 Backup Destination | `/backups/client01` |
| Client02 Backup Destination | `/backups/client02` |
| Linux Automation Tool | cron |
| Linux Backup Schedule | Daily around 2:00 AM |
| Restore Testing | Completed successfully |
| Final Status | Backup and restore workflow completed successfully |
