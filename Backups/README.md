# Backup and Recovery Lab

## Overview

This project documents my backup and recovery lab for my SysAdmin-HomeLab GitHub repo.

The purpose of this lab was to practice backup tasks that a Junior System Administrator may perform, including configuring Windows Server Backup, preparing backup storage, setting up Linux backup folders, scheduling backups, troubleshooting backup failures, and verifying that data can be restored.

This lab used two backup methods:

1. **Windows Server Backup** for DC01.
2. **Windows SMB shares mounted on Linux, then copied with rsync** for Windows client backups.

## Goal

The goal of this project was to build a simple backup and recovery workflow in my home lab.

I wanted to practice:

- Backing up a Windows Server domain controller
- Adding and formatting a dedicated backup disk
- Creating scheduled backups
- Creating Linux backup storage folders
- Mounting Windows SMB shares on Linux
- Copying client files into Linux backup folders with rsync
- Running scheduled Linux backup jobs with cron
- Testing restore from backup
- Troubleshooting DNS, firewall, SMB, SSH, and mount issues

## Skills Demonstrated

- Windows Server Backup configuration
- Full server backup planning
- Backup disk setup in VMware Workstation
- Disk initialization and NTFS formatting
- Scheduled backup jobs
- Linux backup folder management
- SMB/CIFS share mounting
- rsync file backup
- Cron job scheduling
- Restore testing
- Windows Firewall troubleshooting
- DNS troubleshooting
- Backup verification
- Technical documentation

## Environment

| Component | Details |
|---|---|
| Backup Source | DC01, Client01, Client02 |
| Backup Destination | DC01 backup disk `E:` labeled `DC_Backups`; Linux backup folders under `/backups` |
| Backup Tool | Windows Server Backup, SMB/CIFS, rsync, cron |
| Schedule | Daily backups at 2:00 AM |
| Restore Test | Completed successfully |

## Backup Plan

The backup plan had two main parts.

First, I configured **Windows Server Backup** on DC01. DC01 used its own dedicated backup disk. The backup was configured as a full server backup and scheduled to run daily at **2:00 AM**.

Second, I configured Linux-based file backups for the Windows clients. The Windows client `Users` shares were shared over SMB, mounted on the Linux backup server, and then copied into `/backups` using `rsync`.

The final Linux client backup method was:

```text
Windows SMB share → mounted on Linux → rsync copy into /backups
