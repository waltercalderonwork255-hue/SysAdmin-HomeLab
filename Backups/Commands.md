# Backup Lab Commands

## Overview

This document contains the commands and GUI settings used in the Backup and Recovery Lab.

Sensitive information such as passwords, tokens, private keys, recovery keys, and real credentials are not included.

## Linux Backup Folder Setup

### Create Backup Folders

```bash
sudo mkdir -p /backups/client01
sudo mkdir -p /backups/client02
```

### Verify Backup Folders

```bash
ls /backups
```

Expected output:

```text
client01
client02
```

### Set Backup Folder Permissions

```bash
sudo chmod -R 755 /backups
```

### Verify Folder Structure

```bash
ls -R /backups
```

Expected output:

```text
/backups:
client01  client02

/backups/client01:

/backups/client02:
```

## rsync Installation

### Update Package List

```bash
sudo apt update
```

### Install rsync

```bash
sudo apt install rsync -y
```

### Verify rsync Installation

```bash
rsync --version
```

## Cron Installation and Setup

### Install Cron

```bash
sudo apt update
sudo apt install cron
```

### Start Cron Service

```bash
sudo systemctl start cron
```

### Enable Cron at Boot

```bash
sudo systemctl enable cron
```

### Check Cron Status

```bash
systemctl status cron
```

### Edit User Crontab

```bash
crontab -e
```

### View Saved Cron Jobs

```bash
crontab -l
```

## Cron Backup Jobs

### Client Backup Schedule

```bash
0 2 * * * rsync -av /mnt/client01/ /backups/client01/
15 2 * * * rsync -av /mnt/client02/ /backups/client02/
```

### Schedule Meaning

```text
0 2 * * *    Runs every day at 2:00 AM
15 2 * * *   Runs every day at 2:15 AM
```

## Manual rsync Backup Commands

### Backup Client01

```bash
rsync -av /mnt/client01/ /backups/client01/
```

### Backup Client02

```bash
rsync -av /mnt/client02/ /backups/client02/
```

## Optional rsync Mirror Commands

### Mirror Client01 Backup

```bash
rsync -av --delete /mnt/client01/ /backups/client01/
```

### Mirror Client02 Backup

```bash
rsync -av --delete /mnt/client02/ /backups/client02/
```

## SMB/CIFS Mount Setup

### Create Mount Points

```bash
sudo mkdir -p /mnt/client01
sudo mkdir -p /mnt/client02
```

### Mount Client01 SMB Share

```bash
sudo mount -t cifs //192.168.58.132/Users /mnt/client01 -o username=USERNAME
```

### Mount Client02 SMB Share

```bash
sudo mount -t cifs //192.168.58.131/Users /mnt/client02 -o username=USERNAME
```

### Verify Client01 Mount

```bash
ls /mnt/client01
```

### Verify Client02 Mount

```bash
ls /mnt/client02
```

Expected example output:

```text
Public
Default
USERNAME
```

## SMB Troubleshooting Commands

### Install smbclient

```bash
sudo apt update
sudo apt install smbclient
```

### List SMB Shares

```bash
smbclient -L //192.168.58.131 -U username@corp.local
```

## DNS Troubleshooting Commands

### Check Current DNS Configuration

```bash
cat /etc/resolv.conf
```

### Edit systemd-resolved Configuration

```bash
sudo vim /etc/systemd/resolved.conf
```

### Example DNS Settings

```text
DNS=192.168.58.10
Domains=corp.local
```

### Restart systemd-resolved

```bash
sudo systemctl restart systemd-resolved
```

### Test Name Resolution

```bash
ping client01
ping client02
```

## Install Ping Utility if Needed

```bash
sudo apt update
sudo apt install iputils-ping
```

## OpenSSH Server Test Commands

### Install and Start OpenSSH Server on Windows

Run in PowerShell as Administrator:

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
Get-Service sshd
```

### Test SSH from Linux

```bash
ssh username@client02
```

## Failed rsync Over SSH Test Commands

### Dry Run rsync Over SSH

```bash
rsync -avn username@client02:/Users/username/ /backups/client02/
```

### Attempt rsync Over SSH

```bash
rsync -av username@client02:/Users/username/ /backups/client02/
```

### Error Encountered

```text
rsync: command not found
rsync error code 12
```

## Unmount Wrong SMB Share

### Unmount Client02 Mount Point

```bash
sudo umount /mnt/client02
```

### Verify It Is Unmounted

```bash
mount | grep client02
```

If the command returns nothing, the mount point is no longer mounted.

## Restore Test Commands

### Verify Backup Files Exist

```bash
ls /backups/client01
ls /backups/client02
```

### Example Restore Copy

```bash
cp -r /backups/client01/RESTORE_TEST_FILE /tmp/
```

### Verify Restored File

```bash
ls /tmp/
```

## Windows Server Backup Commands and Settings

Most Windows Server Backup steps were completed through the GUI.

### Install Windows Server Backup

```text
Server Manager → Manage → Add Roles and Features → Features → Windows Server Backup
```

### Open Windows Server Backup

```text
Tools → Windows Server Backup
```

### Start Backup Schedule Wizard

```text
Local Backup → Backup Schedule...
```

### Backup Configuration

```text
Backup type: Full Server
Schedule: Once a day
Time: 2:00 AM
Destination: Back up to a hard disk dedicated for backups
Selected disk: E:
```

## VMware Backup Disk Settings

### Virtual Disk Configuration

```text
Controller: SCSI
Disk type: Create a new virtual disk
Disk size: 40–60 GB
Store virtual disk as a single file
File name: Windows Server 2022_DC01_BackupDisk.vmdk
```

## Windows Disk Management Settings

### Backup Disk Configuration

```text
Partition style: GPT
File system: NTFS
Drive letter: E:
Volume label: DC_Backups
```

## Windows Firewall Rules

### Enable ICMP Rule

```text
Windows Defender Firewall with Advanced Security
Inbound Rules
File and Printer Sharing (Echo Request - ICMPv4-In)
Enable Rule
Profile: Domain
```

### Enable SMB Rule

```text
Windows Defender Firewall with Advanced Security
Inbound Rules
File and Printer Sharing (SMB-In)
Enable Rule
Profile: Domain
```

### Enable OpenSSH Rule

```text
Windows Defender Firewall with Advanced Security
Inbound Rules
OpenSSH Server (sshd)
Enable Rule
Profile: Domain
Port: 22
```

## Notes

- Do not include real passwords in commands committed to GitHub.
- Use placeholders like `USERNAME`, `PASSWORD`, or `username@corp.local`.
- The final working client backup method was SMB mounted on Linux, then copied with `rsync`.
- `rsync` over SSH was tested but not used as the final method because Windows did not have `rsync` available on the remote side.
- Windows Server Backup completed successfully.
- Restore testing completed successfully.
