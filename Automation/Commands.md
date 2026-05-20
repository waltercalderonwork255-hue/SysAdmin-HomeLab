# Automation Commands

## Overview

This file documents the main commands used in the Automation Lab. These commands were used to practice PowerShell automation, Active Directory bulk user creation, Linux cron examples, Bash scripting, rsync backup automation, and Linux automatic updates.

This was completed in a home lab environment for Junior System Administrator practice.

> **Security Note:**  
> Do not upload real passwords, tokens, API keys, private keys, or secrets to GitHub. Any password values should be replaced with placeholders such as `PASSWORD_HERE`.

## Environment

| System | Purpose |
|---|---|
| DC01 | Windows Server / Active Directory automation |
| LINUX01 | Linux automation, cron examples, backup script, unattended-upgrades |
| PowerShell | Used for CSV generation and AD user creation |
| Bash | Used for Linux scripting and backup automation |
| Cron | Used for Linux scheduling examples |
| rsync | Used for backup automation |

---

# PowerShell Commands

## Create a Scripts Folder on DC01

```powershell
New-Item -Path "C:\scripts" -ItemType Directory
```

### Purpose

Creates a folder to store PowerShell scripts and CSV files.

---

## Generate a CSV File with 50 Test Users

```powershell
1..50 | ForEach-Object {
    [PSCustomObject]@{
        Name = "Test User $_"
        Username = "TestUser$_"
    }
} | Export-Csv "C:\scripts\users.csv" -NoTypeInformation
```

### Purpose

Creates a CSV file containing 50 test users.

### Expected Output

```text
C:\scripts\users.csv
```

Example CSV content:

```csv
"Name","Username"
"Test User 1","TestUser1"
"Test User 2","TestUser2"
"Test User 3","TestUser3"
```

---

## View the CSV File

```powershell
Import-Csv "C:\scripts\users.csv"
```

### Purpose

Displays the contents of the CSV file in PowerShell.

---

## Bulk Create Active Directory Users from CSV

```powershell
Import-Csv "C:\scripts\users.csv" | ForEach-Object {
    New-ADUser `
        -Name $_.Name `
        -SamAccountName $_.Username `
        -AccountPassword (ConvertTo-SecureString "PASSWORD_HERE" -AsPlainText -Force) `
        -Enabled $true
}
```

### Purpose

Imports the CSV file and creates Active Directory users.

### Important Note

The real lab password was removed from this GitHub-safe version and replaced with:

```text
PASSWORD_HERE
```

Do not upload real passwords to GitHub.

---

## Verify Test Users Were Created

```powershell
Get-ADUser -Filter 'SamAccountName -like "TestUser*"'
```

### Purpose

Checks whether the test users were created in Active Directory.

---

## Verify Test Users with Selected Properties

```powershell
Get-ADUser -Filter 'SamAccountName -like "TestUser*"' | Select-Object Name, SamAccountName, Enabled
```

### Purpose

Displays a cleaner list of the created test users.

---

## Count Created Test Users

```powershell
(Get-ADUser -Filter 'SamAccountName -like "TestUser*"').Count
```

### Purpose

Counts how many test users exist.

Expected count:

```text
50
```

---

## Optional: Export AD User List for Documentation

```powershell
Get-ADUser -Filter 'SamAccountName -like "TestUser*"' |
Select-Object Name, SamAccountName, Enabled |
Export-Csv "C:\scripts\created-test-users.csv" -NoTypeInformation
```

### Purpose

Exports the created test users into a second CSV file for documentation.

---

# Linux Commands

## Update Package Lists

```bash
sudo apt update
```

### Purpose

Refreshes the Linux package list.

---

## Upgrade Installed Packages

```bash
sudo apt upgrade -y
```

### Purpose

Installs available package updates.

---

## Open Root Crontab

```bash
sudo crontab -e
```

### Purpose

Opens the root user's cron schedule for editing.

---

## Linux Automatic Update Cron Example

```bash
0 3 * * * apt update && apt upgrade -y >> /var/log/auto-updates.log 2>&1
```

### Purpose

Runs Linux updates every day at 3 AM and logs output.

### Log File

```text
/var/log/auto-updates.log
```

### Cron Schedule Explanation

| Field | Value | Meaning |
|---|---|---|
| Minute | 0 | At minute 0 |
| Hour | 3 | At 3 AM |
| Day of Month | * | Every day of the month |
| Month | * | Every month |
| Day of Week | * | Every day of the week |

---

## Check Automatic Update Log

```bash
cat /var/log/auto-updates.log
```

### Purpose

Views the output from the automatic update cron example.

---

## Follow Automatic Update Log Live

```bash
tail -f /var/log/auto-updates.log
```

### Purpose

Watches the log file live as it updates.

---

# Bash Backup Script Commands

## Create Backup Script

```bash
sudo nano /usr/local/bin/backup.sh
```

### Purpose

Creates or edits the backup script.

---

## Backup Script Content

```bash
#!/bin/bash

SRC="/mnt/media"
DEST="/backup/media"

rsync -avh --delete "$SRC/" "$DEST/" >> /var/log/backup.log 2>&1
```

### Purpose

Uses `rsync` to sync files from the source directory to the backup destination and writes output to a log file.

---

## Make Backup Script Executable

```bash
sudo chmod +x /usr/local/bin/backup.sh
```

### Purpose

Gives the script execute permission.

---

## Verify Backup Script Permissions

```bash
ls -l /usr/local/bin/backup.sh
```

### Purpose

Confirms the script is executable.

Example output should include `x` permissions:

```text
-rwxr-xr-x
```

---

## Run Backup Script Manually

```bash
sudo /usr/local/bin/backup.sh
```

### Purpose

Runs the backup script manually for testing before scheduling.

---

## Check Backup Destination

```bash
ls -lah /backup/media
```

### Purpose

Verifies that files were copied to the backup destination.

---

## Check Backup Log

```bash
cat /var/log/backup.log
```

### Purpose

Views the backup log file.

---

## Follow Backup Log Live

```bash
tail -f /var/log/backup.log
```

### Purpose

Watches the backup log live while the script runs.

---

# Cron Backup Commands

## Open Root Crontab

```bash
sudo crontab -e
```

### Purpose

Opens the root user's scheduled cron jobs.

---

## Backup Cron Example

```bash
0 2 * * * /usr/local/bin/backup.sh
```

### Purpose

Runs the backup script every day at 2 AM.

### Cron Schedule Explanation

| Field | Value | Meaning |
|---|---|---|
| Minute | 0 | At minute 0 |
| Hour | 2 | At 2 AM |
| Day of Month | * | Every day of the month |
| Month | * | Every month |
| Day of Week | * | Every day of the week |

---

## List Root Cron Jobs

```bash
sudo crontab -l
```

### Purpose

Displays root's current scheduled cron jobs.

---

# rsync Commands

## Basic rsync Backup Command

```bash
rsync -avh --delete "/mnt/media/" "/backup/media/"
```

### Purpose

Syncs files from the source directory to the backup destination.

---

## rsync Backup with Logging

```bash
rsync -avh --delete "/mnt/media/" "/backup/media/" >> /var/log/backup.log 2>&1
```

### Purpose

Runs the backup and saves output/errors to a log file.

---

## rsync Dry Run

```bash
rsync -avh --delete --dry-run "/mnt/media/" "/backup/media/"
```

### Purpose

Shows what rsync would do without actually copying or deleting files.

### Why This Is Useful

This is safer when testing because `--delete` can remove files from the destination.

---

# unattended-upgrades Commands

## Install unattended-upgrades

```bash
sudo apt install unattended-upgrades
```

### Purpose

Installs the built-in Linux automatic update tool.

---

## Configure unattended-upgrades

```bash
sudo dpkg-reconfigure unattended-upgrades
```

### Purpose

Opens the configuration prompt for enabling unattended upgrades.

---

## Check unattended-upgrades Service Status

```bash
systemctl status unattended-upgrades
```

### Purpose

Checks whether the unattended-upgrades service is running.

---

## View unattended-upgrades Log

```bash
cat /var/log/unattended-upgrades/unattended-upgrades.log
```

### Purpose

Displays the unattended-upgrades log.

---

## Follow unattended-upgrades Log Live

```bash
tail -f /var/log/unattended-upgrades/unattended-upgrades.log
```

### Purpose

Watches unattended-upgrades logs live.

---

# Troubleshooting Commands

## Check Bash Script Syntax

```bash
bash -n /usr/local/bin/backup.sh
```

### Purpose

Checks the backup script for syntax errors without running it.

---

## Run Bash Script with Debug Output

```bash
sudo bash -x /usr/local/bin/backup.sh
```

### Purpose

Runs the script and shows each command as it executes.

---

## Check if Source Folder Exists

```bash
ls -lah /mnt/media
```

### Purpose

Confirms the backup source directory exists.

---

## Check if Destination Folder Exists

```bash
ls -lah /backup/media
```

### Purpose

Confirms the backup destination directory exists.

---

## Create Backup Destination Folder

```bash
sudo mkdir -p /backup/media
```

### Purpose

Creates the backup destination folder if it does not already exist.

---

## Check Disk Space

```bash
df -h
```

### Purpose

Shows available disk space.

---

## Check File and Folder Permissions

```bash
ls -ld /mnt/media
ls -ld /backup/media
```

### Purpose

Checks permissions on the source and destination directories.

---

## Check Recent Cron Logs

```bash
grep CRON /var/log/syslog
```

### Purpose

Shows recent cron activity on Ubuntu-based systems.

---

## Check Cron Logs for Backup Script

```bash
grep backup.sh /var/log/syslog
```

### Purpose

Checks whether the backup script was triggered by cron.

---

# GitHub Safety Checklist

Before uploading commands to GitHub, check for:

- No real passwords
- No API keys
- No tokens
- No private keys
- No personal usernames that should be hidden
- No public IP addresses that should stay private
- No sensitive file paths
- No production system names
- No confidential company information

Use safe placeholders when needed:

```text
PASSWORD_HERE
TOKEN_HERE
API_KEY_HERE
PRIVATE_PATH_HERE
```

---

# Lessons Learned

- PowerShell can automate repetitive Active Directory tasks.
- CSV files are useful for bulk user creation.
- Cron can schedule Linux automation tasks.
- Logs are important because scheduled jobs can fail silently.
- rsync is useful for backup automation, but `--delete` must be used carefully.
- Scripts should be tested manually before being scheduled.
- Passwords and secrets should never be hardcoded in GitHub documentation.
- Built-in tools like `unattended-upgrades` can be cleaner than custom update scripts.
