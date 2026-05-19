# Backup Lab Diagrams

## Overview

This document contains all diagrams for the Backup and Recovery Lab.

These diagrams show:

- DC01 backup using Windows Server Backup
- Windows client backups using SMB shares mounted on Linux
- rsync backup flow
- Cron backup schedule
- Troubleshooting flows
- Restore testing flow

Sensitive information such as passwords, tokens, private keys, recovery keys, and real credentials are not included.

## Backup Lab Topology

```mermaid
flowchart TD
    DC01[DC01<br>Windows Server 2022<br>Domain Controller] --> WSB[Windows Server Backup]
    WSB --> EDRIVE[E: Drive<br>DC_Backups<br>Dedicated Backup Disk]

    CLIENT01[Client01<br>Windows Client] --> SMB1[SMB Share<br>C:\Users]
    CLIENT02[Client02<br>Windows Client] --> SMB2[SMB Share<br>C:\Users]

    LINUX01[LINUX01<br>Linux Backup Server]

    SMB1 --> MNT1[/mnt/client01<br>Mounted on LINUX01]
    SMB2 --> MNT2[/mnt/client02<br>Mounted on LINUX01]

    LINUX01 --> MNT1
    LINUX01 --> MNT2

    MNT1 --> RSYNC1[rsync Backup Job]
    MNT2 --> RSYNC2[rsync Backup Job]

    RSYNC1 --> BACKUP1[/backups/client01]
    RSYNC2 --> BACKUP2[/backups/client02]
```

## Windows Server Backup Flow

```mermaid
flowchart TD
    START[Start on DC01] --> INSTALL[Install Windows Server Backup]
    INSTALL --> OPEN[Open Windows Server Backup]
    OPEN --> SCHEDULE[Create Backup Schedule]
    SCHEDULE --> FULL[Select Full Server Backup]
    FULL --> TIME[Schedule Daily Backup at 2:00 AM]
    TIME --> ERROR{Backup Disk Available?}

    ERROR -- No --> ADDDISK[Add Dedicated Virtual Disk in VMware]
    ADDDISK --> INIT[Initialize Disk in Disk Management]
    INIT --> FORMAT[Format Disk as NTFS]
    FORMAT --> LETTER[Assign Drive Letter E:]
    LETTER --> LABEL[Label Disk DC_Backups]
    LABEL --> DEST[Select E: as Backup Destination]

    ERROR -- Yes --> DEST

    DEST --> RUN[Backup Job Runs]
    RUN --> SUCCESS[Backup Completed Successfully]
    SUCCESS --> RESTORE[Restore Test Completed Successfully]
```

## DC01 Backup Design

```mermaid
flowchart LR
    DC01[DC01<br>Domain Controller] --> WSB[Windows Server Backup]
    WSB --> FULL[Full Server Backup]
    FULL --> DISK[E: DC_Backups<br>Dedicated Backup Disk]
    DISK --> VERIFY[Backup Completed Successfully]
    VERIFY --> RESTORE[Restore Test Completed Successfully]
```

## Linux Client Backup Flow

```mermaid
flowchart TD
    START[Start on LINUX01] --> FOLDERS[Create Backup Folders]
    FOLDERS --> CLIENT01DIR[/backups/client01]
    FOLDERS --> CLIENT02DIR[/backups/client02]

    START --> MOUNTS[Create Mount Points]
    MOUNTS --> MNT1[/mnt/client01]
    MOUNTS --> MNT2[/mnt/client02]

    CLIENT01[Client01<br>Windows Client] --> SHARE1[Share C:\Users over SMB]
    CLIENT02[Client02<br>Windows Client] --> SHARE2[Share C:\Users over SMB]

    SHARE1 --> MNT1
    SHARE2 --> MNT2

    MNT1 --> RSYNC1[rsync Copy]
    MNT2 --> RSYNC2[rsync Copy]

    RSYNC1 --> CLIENT01DIR
    RSYNC2 --> CLIENT02DIR

    CLIENT01DIR --> VERIFY1[Verify Files Exist]
    CLIENT02DIR --> VERIFY2[Verify Files Exist]

    VERIFY1 --> RESTORE[Restore Test]
    VERIFY2 --> RESTORE
    RESTORE --> SUCCESS[Restore Completed Successfully]
```

## Client Backup Design

```mermaid
flowchart LR
    CLIENT01[Client01<br>C:\Users] --> SMB1[SMB Share]
    SMB1 --> MNT1[/mnt/client01]
    MNT1 --> RSYNC1[rsync]
    RSYNC1 --> B1[/backups/client01]

    CLIENT02[Client02<br>C:\Users] --> SMB2[SMB Share]
    SMB2 --> MNT2[/mnt/client02]
    MNT2 --> RSYNC2[rsync]
    RSYNC2 --> B2[/backups/client02]
```

## Final Client Backup Method

```mermaid
flowchart LR
    A[Windows Client SMB Share] --> B[Mounted on Linux]
    B --> C[Copied with rsync]
    C --> D[/backups Directory]
```

## Backup Schedule Diagram

```mermaid
timeline
    title Backup Schedule

    2:00 AM : DC01 Windows Server Backup
            : Client01 rsync backup

    2:15 AM : Client02 rsync backup
```

## Backup Folder Structure

```text
SysAdmin-HomeLab/
└── Backups/
    ├── README.md
    ├── Configs/
    ├── Commands/
    ├── Diagrams/
    └── Writeups/
```

## Linux Backup Folder Structure

```text
/backups
├── client01
└── client02
```

## Linux Mount Point Structure

```text
/mnt
├── client01
└── client02
```

## Troubleshooting Flow: Windows Server Backup Disk Error

```mermaid
flowchart TD
    ERROR[Error: No disks are available for backup storage] --> CHECK[Check if DC01 has a dedicated backup disk]
    CHECK --> NODISK{Dedicated disk attached?}

    NODISK -- No --> ADD[Add new virtual disk in VMware]
    ADD --> BOOT[Start DC01]
    BOOT --> DISKMGMT[Open Disk Management]
    DISKMGMT --> INIT[Initialize disk as GPT]
    INIT --> FORMAT[Create New Simple Volume]
    FORMAT --> NTFS[Format as NTFS]
    NTFS --> E[Assign E: Drive Letter]
    E --> LABEL[Label DC_Backups]
    LABEL --> RETRY[Retry Windows Server Backup Schedule]

    NODISK -- Yes --> REVIEW[Review disk status and formatting]
    REVIEW --> RETRY

    RETRY --> SUCCESS[Backup destination available]
```

## Troubleshooting Flow: rsync Over SSH Failed

```mermaid
flowchart TD
    START[Test rsync over SSH to Windows client] --> SSH[SSH connection works]
    SSH --> RSYNC[Run rsync command]
    RSYNC --> ERROR[Error: rsync command not found]
    ERROR --> CAUSE[Cause: Windows does not have rsync installed on remote side]
    CAUSE --> DECISION[Choose better method for Windows clients]
    DECISION --> SMB[Use SMB/CIFS share]
    SMB --> MOUNT[Mount Windows share on Linux]
    MOUNT --> LOCALRSYNC[Run local rsync from mount point]
    LOCALRSYNC --> BACKUP[Copy files into /backups]
    BACKUP --> SUCCESS[Client backup completed]
```

## Troubleshooting Flow: Hostname Failed but IP Worked

```mermaid
flowchart TD
    START[Try mounting SMB share by hostname] --> FAIL[Hostname does not work]
    FAIL --> TESTIP[Try mounting SMB share by IP address]
    TESTIP --> WORKS{IP mount works?}

    WORKS -- Yes --> DNSISSUE[Likely DNS or name resolution issue]
    DNSISSUE --> CHECKDNS[Check Linux DNS settings]
    CHECKDNS --> RESOLVED[Review systemd-resolved configuration]
    RESOLVED --> SETDNS[Set DNS to DC01<br>192.168.58.10]
    SETDNS --> TESTPING[Test ping client01 and client02]

    WORKS -- No --> NETWORK[Check network, firewall, SMB rule, and credentials]
```

## Troubleshooting Flow: Firewall Blocking Backup Access

```mermaid
flowchart TD
    START[Linux cannot reach Windows client] --> CHECKPING[Test ping]
    CHECKPING --> PINGFAIL{Ping fails?}

    PINGFAIL -- Yes --> ICMP[Enable ICMP Echo Request Rule]
    ICMP --> DOMAIN1[Use Domain Profile]
    DOMAIN1 --> TESTPING[Test ping again]

    TESTPING --> CHECKSMB[Test SMB access]
    CHECKSMB --> SMBFAIL{SMB fails?}

    SMBFAIL -- Yes --> SMBRULE[Enable File and Printer Sharing SMB-In]
    SMBRULE --> DOMAIN2[Use Domain Profile]
    DOMAIN2 --> TESTMOUNT[Test SMB mount again]

    SMBFAIL -- No --> SUCCESS[SMB access works]
    TESTMOUNT --> SUCCESS
```

## Restore Testing Flow

```mermaid
flowchart TD
    START[Start Restore Test] --> CHECK[Confirm backup files exist]
    CHECK --> RESTORE[Restore file or folder from backup]
    RESTORE --> OPEN[Open or inspect restored data]
    OPEN --> COMPARE[Confirm restored data matches expected result]
    COMPARE --> SUCCESS[Restore Test Completed Successfully]
```

## Final Summary Diagram

```mermaid
flowchart TD
    DC01[DC01] --> WSB[Windows Server Backup]
    WSB --> DCBACKUP[E: DC_Backups]

    CLIENT01[Client01] --> SMBCLIENT01[SMB Users Share]
    CLIENT02[Client02] --> SMBCLIENT02[SMB Users Share]

    SMBCLIENT01 --> LINUXMNT1[/mnt/client01]
    SMBCLIENT02 --> LINUXMNT2[/mnt/client02]

    LINUXMNT1 --> RSYNC1[rsync]
    LINUXMNT2 --> RSYNC2[rsync]

    RSYNC1 --> BACKUPCLIENT01[/backups/client01]
    RSYNC2 --> BACKUPCLIENT02[/backups/client02]

    DCBACKUP --> RESTORE[Restore Testing]
    BACKUPCLIENT01 --> RESTORE
    BACKUPCLIENT02 --> RESTORE

    RESTORE --> DONE[Backup and Restore Workflow Verified]
```
