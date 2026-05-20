# Automation Configs

## Overview

This file documents the configuration examples used in the Automation Lab. These configs support the automation work completed with PowerShell, Bash, cron, rsync, and `unattended-upgrades`.

The goal of this folder is to show the configuration side of the automation project in a clean, GitHub-safe way.

This was completed in a home lab environment for Junior System Administrator practice.

> **Security Note:**  
> Do not upload real passwords, API keys, tokens, private keys, or secrets to GitHub. Any sensitive values should be replaced with placeholders such as `PASSWORD_HERE`.

---

## Environment

| Component | Details |
|---|---|
| Windows Server | DC01 |
| Linux Server | LINUX01 |
| PowerShell Configs | CSV user file and bulk user creation example |
| Linux Configs | Bash backup script, cron examples, unattended-upgrades setup |
| Scheduler | Cron examples |
| Backup Tool | rsync |
| Update Tool | unattended-upgrades |

---

# PowerShell / Active Directory Configs

## Config: users.csv

### Purpose

The `users.csv` file was used as the input file for bulk Active Directory user creation.

Instead of manually creating each user, PowerShell imported this CSV file and created the accounts automatically.

### File Location

```text
C:\scripts\users.csv
```

### Example CSV Content

```csv
"Name","Username"
"Test User 1","TestUser1"
"Test User 2","TestUser2"
"Test User 3","TestUser3"
"Test User 4","TestUser4"
"Test User 5","TestUser5"
```

### Full CSV Generation Command

```powershell
1..50 | ForEach-Object {
    [PSCustomObject]@{
        Name = "Test User $_"
        Username = "TestUser$_"
    }
} | Export-Csv "C:\scripts\users.csv" -NoTypeInformation
```

### Important Notes

- The CSV was created for lab testing only.
- The users were test users, not real employee accounts.
- The CSV was used to practice bulk user creation.
- The users were created inside a specific OU.
- The exact OU path was not documented and should be added later if available.

---

## Config: Bulk AD User Creation Script

### Purpose

This PowerShell configuration/script imports the CSV file and creates Active Directory users.

### GitHub-Safe Version

```powershell
Import-Csv "C:\scripts\users.csv" | ForEach-Object {
    New-ADUser `
        -Name $_.Name `
        -SamAccountName $_.Username `
        -AccountPassword (ConvertTo-SecureString "PASSWORD_HERE" -AsPlainText -Force) `
        -Enabled $true
}
```

### Sensitive Information Removed

The original lab script included a plaintext lab password. That password was removed for GitHub and replaced with:

```text
PASSWORD_HERE
```

### Configuration Notes

| Setting | Value |
|---|---|
| Input File | `C:\scripts\users.csv` |
| User Display Name | Pulled from `Name` column |
| Username / SamAccountName | Pulled from `Username` column |
| Password | Replaced with `PASSWORD_HERE` |
| Account Status | Enabled |
| OU Location | Specific OU used; exact OU path needs to be documented |

### Recommended Future Improvement

Add the `-Path` option to specify the exact OU where the users should be created.

Example:

```powershell
Import-Csv "C:\scripts\users.csv" | ForEach-Object {
    New-ADUser `
        -Name $_.Name `
        -SamAccountName $_.Username `
        -AccountPassword (ConvertTo-SecureString "PASSWORD_HERE" -AsPlainText -Force) `
        -Enabled $true `
        -Path "OU=TestUsers,DC=corp,DC=local"
}
```

> Replace `OU=TestUsers,DC=corp,DC=local` with the real lab OU path if known.

---

# Bash / Linux Configs

## Config: backup.sh

### Purpose

The `backup.sh` script was used on LINUX01 to automate rsync backups.

The script copies files from a source directory to a backup destination and logs the results.

### File Location

```text
/usr/local/bin/backup.sh
```

### GitHub-Safe Script

```bash
#!/bin/bash

SRC="/mnt/media"
DEST="/backup/media"

rsync -avh --delete "$SRC/" "$DEST/" >> /var/log/backup.log 2>&1
```

### Configuration Breakdown

| Setting | Value |
|---|---|
| Script Path | `/usr/local/bin/backup.sh` |
| Source Directory | `/mnt/media` |
| Destination Directory | `/backup/media` |
| Backup Tool | `rsync` |
| Logging Path | `/var/log/backup.log` |
| Delete Behavior | `--delete` enabled |
| Status | Tested successfully after troubleshooting |

### rsync Options Explained

| Option | Purpose |
|---|---|
| `-a` | Archive mode; preserves file properties |
| `-v` | Verbose output |
| `-h` | Human-readable output |
| `--delete` | Deletes files in destination if they no longer exist in source |

### Logging Configuration

```bash
>> /var/log/backup.log 2>&1
```

### What This Does

- `>> /var/log/backup.log` appends normal output to the backup log.
- `2>&1` sends error output to the same log file.
- This helps troubleshoot automation after the script runs.

### Safety Notes

- `--delete` can remove files from the backup destination.
- Test with `--dry-run` before using `--delete`.
- Confirm source and destination paths before running the script.
- Review `/var/log/backup.log` after testing.

### Safer Test Version

```bash
#!/bin/bash

SRC="/mnt/media"
DEST="/backup/media"

rsync -avh --delete --dry-run "$SRC/" "$DEST/"
```

### Improved Future Version

```bash
#!/bin/bash

SRC="/mnt/media"
DEST="/backup/media"
LOG="/var/log/backup.log"

echo "Backup started: $(date)" >> "$LOG"

if [ ! -d "$SRC" ]; then
    echo "ERROR: Source directory does not exist: $SRC" >> "$LOG"
    exit 1
fi

if [ ! -d "$DEST" ]; then
    echo "ERROR: Destination directory does not exist: $DEST" >> "$LOG"
    exit 1
fi

rsync -avh --delete "$SRC/" "$DEST/" >> "$LOG" 2>&1

if [ $? -eq 0 ]; then
    echo "Backup completed successfully: $(date)" >> "$LOG"
else
    echo "ERROR: Backup failed: $(date)" >> "$LOG"
fi
```

### Why This Version Is Better

- Adds start and end timestamps.
- Checks if the source folder exists.
- Checks if the destination folder exists.
- Logs success or failure.
- Is easier to troubleshoot.

---

# Cron Configs

## Config: Root Crontab

### Purpose

Cron was used as a scheduling example for Linux automation tasks.

The examples show how updates and backups could be scheduled automatically.

### Open Root Crontab

```bash
sudo crontab -e
```

---

## Config: Automatic Update Cron Example

### Purpose

This cron example runs package updates daily at 3 AM and logs the results.

### Cron Entry

```cron
0 3 * * * apt update && apt upgrade -y >> /var/log/auto-updates.log 2>&1
```

### Schedule Breakdown

| Field | Value | Meaning |
|---|---|---|
| Minute | `0` | At minute 0 |
| Hour | `3` | At 3 AM |
| Day of Month | `*` | Every day |
| Month | `*` | Every month |
| Day of Week | `*` | Every day of the week |

### Log File

```text
/var/log/auto-updates.log
```

### Status

Created as an example.

### Notes

This was documented as a cron example. For a cleaner and more standard update method, `unattended-upgrades` was also configured.

---

## Config: Backup Cron Example

### Purpose

This cron example runs the backup script every day at 2 AM.

### Cron Entry

```cron
0 2 * * * /usr/local/bin/backup.sh
```

### Schedule Breakdown

| Field | Value | Meaning |
|---|---|---|
| Minute | `0` | At minute 0 |
| Hour | `2` | At 2 AM |
| Day of Month | `*` | Every day |
| Month | `*` | Every month |
| Day of Week | `*` | Every day of the week |

### Related Script

```text
/usr/local/bin/backup.sh
```

### Related Log

```text
/var/log/backup.log
```

### Status

Created as an example.

### Notes

The backup script itself was tested successfully after troubleshooting. The cron schedule was documented as an example for scheduled automation.

---

# unattended-upgrades Config

## Config: unattended-upgrades

### Purpose

`unattended-upgrades` was configured to practice built-in Linux automatic update automation.

This is often cleaner than using a basic cron job for updates.

### Install Command

```bash
sudo apt install unattended-upgrades
```

### Configuration Command

```bash
sudo dpkg-reconfigure unattended-upgrades
```

### Status

Completed.

### Common Config File Locations

```text
/etc/apt/apt.conf.d/20auto-upgrades
/etc/apt/apt.conf.d/50unattended-upgrades
```

### Example 20auto-upgrades Config

```text
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

### What This Means

| Setting | Meaning |
|---|---|
| `Update-Package-Lists "1"` | Updates package lists automatically |
| `Unattended-Upgrade "1"` | Enables unattended upgrades |

### Log Location

```text
/var/log/unattended-upgrades/unattended-upgrades.log
```

### Verification Commands

```bash
systemctl status unattended-upgrades
```

```bash
cat /var/log/unattended-upgrades/unattended-upgrades.log
```

### Notes

The exact final configuration file contents were not documented in the raw notes. The examples above show common Ubuntu unattended-upgrades configuration locations and settings.

---

# Logging Configs

## Backup Log

### Path

```text
/var/log/backup.log
```

### Used By

```text
/usr/local/bin/backup.sh
```

### Purpose

Stores output and errors from the rsync backup script.

### Example Log Redirection

```bash
>> /var/log/backup.log 2>&1
```

---

## Automatic Updates Log

### Path

```text
/var/log/auto-updates.log
```

### Used By

```text
Linux update cron example
```

### Purpose

Stores output and errors from the update cron example.

### Example Log Redirection

```bash
>> /var/log/auto-updates.log 2>&1
```

---

## unattended-upgrades Log

### Path

```text
/var/log/unattended-upgrades/unattended-upgrades.log
```

### Used By

```text
unattended-upgrades
```

### Purpose

Stores logs from the built-in unattended-upgrades service.

---

# Permissions Configs

## backup.sh Permissions

### Purpose

The backup script needed execute permission before it could run directly.

### Command

```bash
sudo chmod +x /usr/local/bin/backup.sh
```

### Expected Permission Example

```text
-rwxr-xr-x
```

### Verification

```bash
ls -l /usr/local/bin/backup.sh
```

---

## Backup Directory Permissions

### Source Directory

```text
/mnt/media
```

### Destination Directory

```text
/backup/media
```

### Verification Commands

```bash
ls -ld /mnt/media
ls -ld /backup/media
```

### Notes

The exact permissions were not documented in the raw notes. Permissions should be reviewed if backups fail.

---

# GitHub-Safe Placeholder Values

Use placeholder values in public documentation.

| Placeholder | Use For |
|---|---|
| `PASSWORD_HERE` | Passwords |
| `TOKEN_HERE` | Tokens |
| `API_KEY_HERE` | API keys |
| `PRIVATE_KEY_HERE` | Private keys |
| `PRIVATE_PATH_HERE` | Sensitive personal paths |
| `USERNAME_HERE` | Personal usernames if needed |

---

# Sensitive Information Removed

The following sensitive information should not be included in this GitHub repo:

- Real passwords
- API keys
- Tokens
- Private keys
- Recovery keys
- Personal account information
- Company information
- Production system information

The Active Directory lab password was replaced with:

```text
PASSWORD_HERE
```

---

# Config Summary

| Config | Location | Purpose | Status |
|---|---|---|---|
| users.csv | `C:\scripts\users.csv` | Input file for bulk AD user creation | Completed |
| AD user creation script | PowerShell | Creates users from CSV | Completed and tested |
| backup.sh | `/usr/local/bin/backup.sh` | rsync backup automation | Tested successfully |
| Backup log | `/var/log/backup.log` | Stores backup output/errors | Used in script |
| Update cron example | Root crontab | Example scheduled updates | Created example |
| Backup cron example | Root crontab | Example scheduled backup | Created example |
| unattended-upgrades | `/etc/apt/apt.conf.d/` | Built-in automatic updates | Completed |
| auto-updates log | `/var/log/auto-updates.log` | Update cron example log | Created example |
| unattended-upgrades log | `/var/log/unattended-upgrades/unattended-upgrades.log` | Built-in update log | Completed tool log location |

---

# Lessons Learned

- Config files and scripts should be documented clearly.
- Automation should include logging for verification and troubleshooting.
- Cron jobs can fail silently if output is not logged.
- rsync is useful for backups, but `--delete` must be used carefully.
- PowerShell can automate repetitive Active Directory tasks.
- CSV files are useful for bulk user creation.
- Built-in tools like `unattended-upgrades` can be cleaner than custom update cron jobs.
- GitHub documentation should use sanitized examples instead of real secrets.
