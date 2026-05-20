# Automation Writeups

## Overview

This folder contains writeups for the Automation Lab in my SysAdmin Home Lab project.

The purpose of this writeup is to explain what I automated, why I automated it, what issues I ran into, how I tested the automation, and what I learned from the project.

This project was completed in a home lab environment for Junior System Administrator practice.

---

## Project Summary

In this lab, I practiced automation tasks that are useful for system administration. I used PowerShell on DC01 to automate Active Directory test user creation and Bash on LINUX01 to automate Linux backup tasks.

The project included:

- Creating a CSV file with test users
- Using PowerShell to create Active Directory users from the CSV
- Creating users inside a specific OU
- Creating a Bash backup script with `rsync`
- Troubleshooting Bash script syntax
- Creating cron job examples for scheduled updates and backups
- Configuring `unattended-upgrades` for Linux automatic updates
- Replacing sensitive information before publishing documentation to GitHub

This was not a production environment. It was a home lab project built to practice real sysadmin concepts safely.

---

## Why Automation Matters for SysAdmins

Automation helps system administrators reduce repetitive work and improve consistency.

Instead of manually creating users, copying files, or running the same commands every day, sysadmins can use scripts and scheduled jobs to complete tasks faster and with fewer mistakes.

Automation is useful for tasks such as:

- Creating users
- Running backups
- Checking system status
- Installing updates
- Reviewing logs
- Cleaning up files
- Running scheduled maintenance
- Documenting repeatable processes

This lab helped me understand that automation should not just “run.” It should also be tested, logged, documented, and safe.

---

# Writeup 1: PowerShell CSV User Generation

## Objective

The objective of this part of the lab was to generate a CSV file containing test users for Active Directory automation.

## What I Did

I used PowerShell to create 50 test users and export them to a CSV file.

The script created users with names like:

```text
Test User 1
Test User 2
Test User 3
```

And usernames like:

```text
TestUser1
TestUser2
TestUser3
```

## Command Used

```powershell
1..50 | ForEach-Object {
    [PSCustomObject]@{
        Name = "Test User $_"
        Username = "TestUser$_"
    }
} | Export-Csv "C:\scripts\users.csv" -NoTypeInformation
```

## Why I Did It

I wanted to practice creating a structured input file for bulk user creation.

CSV files are commonly used in IT because they make it easier to import multiple records into a script or tool.

## Result

The CSV file was created successfully at:

```text
C:\scripts\users.csv
```

## What I Learned

I learned how PowerShell can quickly generate structured data and export it into a CSV file. This is useful because many sysadmin tasks depend on importing data from spreadsheets or CSV files.

---

# Writeup 2: Bulk Active Directory User Creation

## Objective

The objective of this part of the lab was to create Active Directory users automatically from a CSV file.

## What I Did

I used PowerShell to import the `users.csv` file and create test users in Active Directory.

The users were created inside a specific OU.

## GitHub-Safe Script

```powershell
Import-Csv "C:\scripts\users.csv" | ForEach-Object {
    New-ADUser `
        -Name $_.Name `
        -SamAccountName $_.Username `
        -AccountPassword (ConvertTo-SecureString "PASSWORD_HERE" -AsPlainText -Force) `
        -Enabled $true
}
```

## Why I Did It

Manually creating many users can take a long time and can lead to mistakes. Bulk user creation is a useful skill for sysadmins because it allows accounts to be created faster and more consistently.

## Result

The script successfully created the test users in Active Directory.

## Verification

I verified the users with PowerShell.

```powershell
Get-ADUser -Filter 'SamAccountName -like "TestUser*"'
```

## Security Note

The original lab script included a test password. Before documenting the project on GitHub, the password was removed and replaced with:

```text
PASSWORD_HERE
```

## What I Learned

I learned that automation can save time, but it also needs to be handled carefully. User creation scripts should avoid hardcoded passwords, should be tested in a lab OU, and should be reviewed before running.

---

# Writeup 3: Linux Backup Automation with rsync

## Objective

The objective of this part of the lab was to create a Bash script on LINUX01 to automate file backups using `rsync`.

## What I Did

I created a backup script called:

```text
/usr/local/bin/backup.sh
```

The script used `rsync` to copy files from a source directory to a backup destination.

## Script Used

```bash
#!/bin/bash

SRC="/mnt/media"
DEST="/backup/media"

rsync -avh --delete "$SRC/" "$DEST/" >> /var/log/backup.log 2>&1
```

## Why I Did It

Backups are an important sysadmin responsibility. I wanted to practice making a simple repeatable backup process instead of copying files manually.

## Result

The backup script was tested and worked successfully after troubleshooting.

## Verification

I checked the backup destination and log file.

```bash
ls -lah /backup/media
cat /var/log/backup.log
```

## Issue Encountered

The original rsync command had syntax problems because the command, options, variables, and log redirection were missing spaces.

Broken example:

```bash
rsync-avh--delete$SRC$DEST >> /var/log/backup.log2>&1
```

Corrected version:

```bash
rsync -avh --delete "$SRC/" "$DEST/" >> /var/log/backup.log 2>&1
```

## What I Learned

I learned that small syntax errors can break automation. I also learned that logging is important because it helps confirm whether a script worked or failed.

---

# Writeup 4: Cron Scheduling Examples

## Objective

The objective of this part of the lab was to document how Linux automation tasks can be scheduled with cron.

## What I Did

I created cron examples for:

- Automatic Linux updates
- Scheduled backup script execution

## Update Cron Example

```cron
0 3 * * * apt update && apt upgrade -y >> /var/log/auto-updates.log 2>&1
```

This example runs Linux updates every day at 3 AM and logs output to:

```text
/var/log/auto-updates.log
```

## Backup Cron Example

```cron
0 2 * * * /usr/local/bin/backup.sh
```

This example runs the backup script every day at 2 AM.

## Why I Did It

Scheduled tasks are common in system administration. Cron allows Linux systems to run jobs automatically at specific times.

## Result

The cron jobs were documented as examples.

## Verification

Verification would include checking the related log files after the scheduled time.

```bash
cat /var/log/auto-updates.log
cat /var/log/backup.log
```

## What I Learned

I learned how cron scheduling works and why logs are important. Since cron jobs run in the background, they can fail silently if output is not redirected to a log file.

---

# Writeup 5: unattended-upgrades Setup

## Objective

The objective of this part of the lab was to configure a built-in Linux tool for automatic updates.

## What I Did

I configured `unattended-upgrades` on Linux.

## Commands Used

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

## Why I Did It

Using a built-in update automation tool is usually cleaner than relying only on a custom cron job. It is also closer to how update automation may be handled in real environments.

## Result

The setup was completed.

## Verification

Useful verification commands include:

```bash
systemctl status unattended-upgrades
cat /var/log/unattended-upgrades/unattended-upgrades.log
```

## What I Learned

I learned that Linux has built-in tools for automatic updates. I also learned that automatic patching should be monitored and documented, especially when updates may require a reboot.

---

# Troubleshooting Writeup

## Issue 1: Cron Jobs Can Fail Silently

### Problem

Cron jobs run in the background and may not show errors on the screen.

### Cause

If output is not redirected, errors may be missed.

### Fix

Add log redirection to the cron command.

Example:

```bash
>> /var/log/auto-updates.log 2>&1
```

### Verification

Check the log file.

```bash
cat /var/log/auto-updates.log
```

### Lesson Learned

Scheduled automation needs logging so the sysadmin can confirm whether the job worked.

---

## Issue 2: rsync Syntax Error

### Problem

The original `rsync` command had spacing issues.

### Cause

The command did not include proper spaces between `rsync`, options, variables, and log redirection.

### Fix

Use the corrected command:

```bash
rsync -avh --delete "$SRC/" "$DEST/" >> /var/log/backup.log 2>&1
```

### Verification

The backup script was tested and worked successfully after troubleshooting.

### Lesson Learned

Always test scripts manually before scheduling them.

---

## Issue 3: Plaintext Password in PowerShell Script

### Problem

The original Active Directory user creation script included a plaintext lab password.

### Cause

The script used a hardcoded password during testing.

### Fix

The password was removed from the GitHub version and replaced with:

```text
PASSWORD_HERE
```

### Verification

The final GitHub documentation does not include the real password.

### Lesson Learned

Never upload passwords, tokens, API keys, private keys, or secrets to GitHub.

---

# Verification Summary

| Task | Verification | Status |
|---|---|---|
| CSV user generation | Confirmed `users.csv` was created | Completed |
| Bulk AD user creation | Confirmed users were created in AD | Completed |
| Specific OU placement | Users were created inside a specific OU | Completed |
| rsync backup script | Script worked after troubleshooting | Completed |
| Cron update example | Documented as example | Created example |
| Cron backup example | Documented as example | Created example |
| unattended-upgrades | Installed and configured | Completed |
| Sensitive info removal | Password replaced with placeholder | Completed |

---

# Security Reflection

This lab reminded me that automation can be powerful, but it can also create problems if it is not written safely.

Important security lessons from this project:

- Do not hardcode passwords.
- Do not upload secrets to GitHub.
- Use placeholders in public documentation.
- Test scripts in a lab before using them anywhere important.
- Use least privilege when possible.
- Be careful with commands like `rsync --delete`.
- Review logs after scheduled tasks run.
- Keep automation simple and documented.

---

# What I Would Improve Next Time

If I continued improving this lab, I would add:

- More comments inside each script
- Better error handling in the Bash backup script
- A dry-run mode for the rsync backup script
- Exact OU path in the PowerShell user creation script
- A safer password-handling method for AD user creation
- More detailed unattended-upgrades configuration notes
- Email or Telegram alerts for failed automation jobs
- Screenshots showing the scripts, logs, and successful results
- A cleanup script to remove test users after the lab
- A better logging format with timestamps

---

# Resume Bullets

- Automated Active Directory test user creation using PowerShell and CSV import, successfully creating bulk users inside a dedicated lab OU.
- Created and tested a Bash/rsync backup automation script on LINUX01, including troubleshooting script syntax and validating successful backup behavior.
- Configured Linux automation examples using cron and completed `unattended-upgrades` setup to practice scheduled maintenance and patching workflows.

---

# Interview Explanation

In my Automation Lab, I practiced using PowerShell and Bash to automate common sysadmin tasks.

On the Windows side, I created a CSV file with 50 test users and used PowerShell to create those users in Active Directory inside a specific OU. On the Linux side, I created a Bash script using `rsync` to automate backups on LINUX01. I also documented cron examples for scheduled jobs and configured `unattended-upgrades` for Linux automatic updates.

The biggest lesson I learned is that automation needs to be tested, logged, and documented. A script is not useful if I cannot verify that it worked. I also learned the importance of removing sensitive information like passwords before publishing anything to GitHub.
