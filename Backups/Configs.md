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
