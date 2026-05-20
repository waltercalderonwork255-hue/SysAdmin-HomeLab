# Automation Lab

## Overview

This project documents basic automation tasks completed in my SysAdmin home lab. The purpose of this lab was to practice using scripts and scheduled tools to reduce repetitive manual work, improve consistency, and build skills that are useful for a Junior System Administrator role.

The automation work included PowerShell automation on DC01 and Linux automation on LINUX01. I practiced creating test users in Active Directory, using CSV files for bulk user creation, writing Bash scripts, using cron examples, automating rsync backups, and configuring Linux automatic updates with `unattended-upgrades`.

This was a home lab project, not a production environment.

## Goal

The goal of this project was to practice automating common sysadmin tasks, including:

- Creating test users in Active Directory using PowerShell
- Generating a CSV file for bulk user creation
- Creating Active Directory users from a CSV file
- Placing test users into a specific OU
- Writing a Bash backup script
- Testing and troubleshooting rsync backup automation
- Creating cron job examples for scheduled Linux tasks
- Configuring Linux automatic updates with `unattended-upgrades`
- Learning how logging helps with troubleshooting automation

## Skills Demonstrated

- PowerShell scripting
- CSV generation and importing
- Active Directory user automation
- Bash scripting
- Linux system administration
- Cron job syntax
- rsync backup automation
- Linux automatic update configuration
- Script troubleshooting
- Basic automation testing
- Safe documentation practices
- Removing sensitive information before uploading to GitHub

## Environment

| Component | Details |
|---|---|
| Scripting Languages | PowerShell, Bash |
| Systems Automated | DC01, LINUX01 |
| Scheduler Used | Cron examples; Windows Task Scheduler was not used |
| Scripts Created | CSV user generation, bulk AD user creation, rsync backup script, Linux update cron example, unattended-upgrades setup |
| Related Projects | Active Directory, Backups, FileServer |

## Automation Summary

| Script or Task | Language/Tool | Purpose | Schedule | Status |
|---|---|---|---|---|
| Generate users.csv | PowerShell | Create a CSV file with 50 test users | Manual run | Completed |
| Bulk AD user creation | PowerShell / Active Directory | Create users from a CSV file into a specific OU | Manual run | Completed and tested |
| Linux automatic updates cron example | Cron / apt | Run Linux updates automatically and log output | Daily at 3 AM example | Created example |
| rsync backup script | Bash / rsync | Sync files from source to backup destination | Daily at 2 AM example | Tested successfully after troubleshooting |
| unattended-upgrades | Linux package tool | Configure built-in Linux automatic updates | Built-in automatic update scheduling | Completed |

## Project Steps

### Step 1: Created a CSV File for Test Users

**What I did:**  
I used PowerShell to create a CSV file containing 50 test users.

**Why I did it:**  
CSV files are commonly used by sysadmins to automate repetitive tasks such as creating multiple user accounts. This helped me practice bulk user provisioning in a lab environment.

**Script/command used:**  

```powershell
1..50 | ForEach-Object {
    [PSCustomObject]@{
        Name = "Test User $_"
        Username = "TestUser$_"
    }
} | Export-Csv "C:\scripts\users.csv" -NoTypeInformation
```

**Verification:**  
The CSV file was created at:

```text
C:\scripts\users.csv
```

The file contained test users such as:

```text
Test User 1, TestUser1
Test User 2, TestUser2
Test User 3, TestUser3
```

### Step 2: Created Bulk Active Directory Users

**What I did:**  
I created a PowerShell script to import users from the CSV file and create Active Directory user accounts.

**Why I did it:**  
Creating users manually can take a lot of time and can lead to mistakes. Bulk user creation helps sysadmins create accounts faster and more consistently.

**Script/command used:**  

```powershell
Import-Csv "C:\scripts\users.csv" | ForEach-Object {
    New-ADUser `
        -Name $_.Name `
        -SamAccountName $_.Username `
        -AccountPassword (ConvertTo-SecureString "PASSWORD_HERE" -AsPlainText -Force) `
        -Enabled $true
}
```

**Verification:**  
The script successfully created the test users in Active Directory. The users were created inside a specific OU.

Example verification command:

```powershell
Get-ADUser -Filter 'SamAccountName -like "TestUser*"'
```

**Security note:**  
The original lab script used a test password. For GitHub, the password was removed and replaced with:

```text
PASSWORD_HERE
```

Passwords should never be uploaded to GitHub.

### Step 3: Created a Linux Automatic Update Cron Example

**What I did:**  
I created a cron job example to automatically run Linux updates and save the output to a log file.

**Why I did it:**  
Scheduled updates are a common automation task. Logging is important because scheduled jobs run in the background and can fail without showing errors on the screen.

**Script/command used:**  

Open the root crontab:

```bash
sudo crontab -e
```

Cron example:

```bash
0 3 * * * apt update && apt upgrade -y >> /var/log/auto-updates.log 2>&1
```

**Verification:**  
This was created as an example. Verification would include checking the update log after the scheduled time:

```bash
cat /var/log/auto-updates.log
```

### Step 4: Created a Bash Backup Script

**What I did:**  
I created a Bash script on LINUX01 to automate backups using `rsync`.

**Why I did it:**  
Backups are an important sysadmin responsibility. Automating backups helps make sure files are copied consistently instead of relying on manual work.

**Script/command used:**  

Create the script:

```bash
sudo nano /usr/local/bin/backup.sh
```

Backup script:

```bash
#!/bin/bash

SRC="/mnt/media"
DEST="/backup/media"

rsync -avh --delete "$SRC/" "$DEST/" >> /var/log/backup.log 2>&1
```

**Verification:**  
The script was tested and worked successfully after troubleshooting.

Verification commands:

```bash
ls -lah /backup/media
cat /var/log/backup.log
```

### Step 5: Made the Backup Script Executable

**What I did:**  
I changed the permissions on the backup script so it could be executed.

**Why I did it:**  
Linux scripts need execute permission before they can be run directly.

**Script/command used:**  

```bash
sudo chmod +x /usr/local/bin/backup.sh
```

**Verification:**  

```bash
ls -l /usr/local/bin/backup.sh
```

The script should show executable permissions.

### Step 6: Created a Cron Example for the Backup Script

**What I did:**  
I created a cron example to run the backup script every day at 2 AM.

**Why I did it:**  
Scheduling backup scripts allows backups to run automatically without needing to manually start them.

**Script/command used:**  

Open the root crontab:

```bash
sudo crontab -e
```

Cron example:

```bash
0 2 * * * /usr/local/bin/backup.sh
```

**Verification:**  
This was created as an example. Verification would include checking the backup directory and log file after the scheduled time:

```bash
ls -lah /backup/media
cat /var/log/backup.log
```

### Step 7: Configured unattended-upgrades

**What I did:**  
I configured the built-in Linux automatic update tool called `unattended-upgrades`.

**Why I did it:**  
Using a built-in update automation tool is usually cleaner than relying only on a custom cron job for patching. It is a more standard way to handle automatic updates on Ubuntu-based systems.

**Script/command used:**  

Install unattended-upgrades:

```bash
sudo apt install unattended-upgrades
```

Enable/configure unattended-upgrades:

```bash
sudo dpkg-reconfigure unattended-upgrades
```

**Verification:**  
The setup was completed. Future improvements could include documenting the exact configuration and checking update logs.

## Scripts

### Script: Generate Test Users CSV

**Purpose:**  
Create a CSV file with 50 test users for Active Directory bulk user creation.

**What it does:**  
The script creates test user names and usernames, then exports them to a CSV file.

**How to run it:**  
Run the command in PowerShell on DC01 or another system with access to the correct script path.

```powershell
1..50 | ForEach-Object {
    [PSCustomObject]@{
        Name = "Test User $_"
        Username = "TestUser$_"
    }
} | Export-Csv "C:\scripts\users.csv" -NoTypeInformation
```

**Expected output:**  
A CSV file is created here:

```text
C:\scripts\users.csv
```

**Safety notes:**  

- Review the CSV before using it to create accounts.
- Use test users only in a lab environment.
- Do not use real employee information in a public GitHub repository.

### Script: Bulk Active Directory User Creation

**Purpose:**  
Create multiple Active Directory users from a CSV file.

**What it does:**  
The script imports the CSV file and creates enabled Active Directory user accounts.

**How to run it:**  
Run the script in PowerShell with the correct Active Directory permissions.

```powershell
Import-Csv "C:\scripts\users.csv" | ForEach-Object {
    New-ADUser `
        -Name $_.Name `
        -SamAccountName $_.Username `
        -AccountPassword (ConvertTo-SecureString "PASSWORD_HERE" -AsPlainText -Force) `
        -Enabled $true
}
```

**Expected output:**  
The test users are created in Active Directory.

**Safety notes:**  

- Do not hardcode real passwords.
- Do not upload real passwords to GitHub.
- Use placeholders such as `PASSWORD_HERE`.
- Create test users inside a lab OU.
- Test the script in a lab before using a similar process anywhere important.
- Consider adding an OU path to the script in the future.

### Script: Linux Automatic Updates Cron Example

**Purpose:**  
Show how Linux updates could be scheduled with cron and logged.

**What it does:**  
Runs `apt update` and `apt upgrade -y`, then sends output and errors to a log file.

**How to run it:**  
Add the following line to the root crontab:

```bash
0 3 * * * apt update && apt upgrade -y >> /var/log/auto-updates.log 2>&1
```

**Expected output:**  
A log file should be created or updated here:

```text
/var/log/auto-updates.log
```

**Safety notes:**  

- Automatic updates should be tested carefully.
- Some updates may require a reboot.
- Production systems usually need maintenance windows.
- `unattended-upgrades` may be a cleaner option than a basic cron update job.

### Script: rsync Backup Script

**Purpose:**  
Automate file backups using `rsync`.

**What it does:**  
Syncs files from `/mnt/media` to `/backup/media` and logs output to `/var/log/backup.log`.

**How to run it:**  

```bash
sudo /usr/local/bin/backup.sh
```

Script:

```bash
#!/bin/bash

SRC="/mnt/media"
DEST="/backup/media"

rsync -avh --delete "$SRC/" "$DEST/" >> /var/log/backup.log 2>&1
```

**Expected output:**  
Files from the source directory should copy to the backup destination.

**Safety notes:**  

- The `--delete` option removes files from the destination if they no longer exist in the source.
- Test with non-critical files first.
- Confirm source and destination paths before running.
- Use quotes around variables to avoid issues with spaces.
- Review the log file after running the script.

### Script: unattended-upgrades Setup

**Purpose:**  
Configure built-in Linux automatic updates.

**What it does:**  
Installs and enables the `unattended-upgrades` package.

**How to run it:**  

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
```

**Expected output:**  
The system is configured to use unattended upgrades.

**Safety notes:**  

- Review configuration before relying on it.
- Monitor update logs.
- Understand whether reboots are required.
- Use maintenance windows for important systems.

## Troubleshooting

### Issue: Cron Jobs Can Fail Silently

**Problem:**  
A cron job can run in the background without showing errors on the screen.

**Cause:**  
Cron jobs do not display output like normal terminal commands. If output is not redirected, errors can be missed.

**Fix:**  
Redirect output and errors to a log file.

Example:

```bash
>> /var/log/auto-updates.log 2>&1
```

**Verification:**  

```bash
cat /var/log/auto-updates.log
```

**Lesson learned:**  
Automation should include logging so the sysadmin can confirm whether the job worked or failed.

### Issue: rsync Command Syntax Needed Cleanup

**Problem:**  
The original rsync command had spacing issues.

Original broken example:

```bash
rsync-avh--delete$SRC$DEST >> /var/log/backup.log2>&1
```

**Cause:**  
Bash commands require correct spacing between commands, options, variables, and redirects.

**Fix:**  
Use the corrected version:

```bash
rsync -avh --delete "$SRC/" "$DEST/" >> /var/log/backup.log 2>&1
```

**Verification:**  
The backup script was tested and worked successfully after troubleshooting.

**Lesson learned:**  
Small syntax mistakes can break automation. Scripts should be tested manually before scheduling.

### Issue: Hardcoded Password in PowerShell Script

**Problem:**  
The bulk user creation script included a plaintext lab password.

**Cause:**  
The script used a hardcoded password value during account creation.

**Fix:**  
For GitHub documentation, the password was replaced with:

```text
PASSWORD_HERE
```

**Verification:**  
The GitHub version of the README does not include the real password.

**Lesson learned:**  
Do not upload passwords, API keys, tokens, private keys, or secrets to GitHub.

## Verification and Testing

- Confirmed PowerShell generated a CSV file for 50 test users.
- Confirmed the bulk Active Directory user script successfully created users.
- Confirmed users were created inside a specific OU.
- Confirmed the rsync backup script worked after troubleshooting syntax issues.
- Created cron job examples for scheduled updates and scheduled backups.
- Completed setup of `unattended-upgrades`.
- Replaced the plaintext lab password with `PASSWORD_HERE` for the GitHub-safe version.

Useful verification commands:

```powershell
Get-ADUser -Filter 'SamAccountName -like "TestUser*"'
```

```bash
cat /var/log/auto-updates.log
cat /var/log/backup.log
ls -lah /backup/media
```

## Security and Best Practices

- Do not hardcode passwords in scripts.
- Do not upload passwords, API keys, tokens, private keys, or other secrets to GitHub.
- Use placeholders such as `PASSWORD_HERE` in public documentation.
- Test scripts manually before scheduling them.
- Add logging so script output can be reviewed later.
- Use least privilege where possible.
- Be careful with destructive options like `rsync --delete`.
- Add comments to scripts so they are easier to understand later.
- Use variables for paths to make scripts easier to update.
- Keep sanitized examples in GitHub instead of scripts with sensitive values.
- For Linux patching, consider built-in tools such as `unattended-upgrades`.
- For larger environments, test automation in a lab before using it on important systems.

## Screenshots to Add Later

| Screenshot | What to Capture | Why It Matters | Suggested Filename |
|---|---|---|---|
| PowerShell CSV generation | PowerShell command and successful CSV creation | Shows user generation automation | `powershell-users-csv.png` |
| users.csv file | Open CSV file with test users | Shows input file for bulk automation | `users-csv-example.png` |
| Active Directory test users | ADUC or PowerShell output showing TestUser accounts | Proves bulk user creation worked | `ad-bulk-users-created.png` |
| Test user OU | The specific OU containing the test users | Shows organized AD user placement | `ad-test-users-ou.png` |
| Cron job example | Root crontab showing scheduled job examples | Shows Linux scheduling practice | `cron-examples.png` |
| Backup script | Sanitized `backup.sh` file | Shows Bash automation script | `backup-script.png` |
| Backup log | `/var/log/backup.log` output | Shows logging and verification | `backup-log-output.png` |
| unattended-upgrades setup | Terminal/configuration screen for unattended-upgrades | Shows Linux update automation | `unattended-upgrades-setup.png` |

## What I Learned

This project helped me understand how automation can reduce repetitive manual work in a sysadmin environment. I practiced creating users from a CSV file, writing simple PowerShell and Bash scripts, using rsync for backup automation, creating cron examples, and configuring Linux automatic updates.

I also learned that automation needs to be tested carefully. A small syntax issue, missing log file, incorrect path, or hardcoded password can cause problems. Good automation should be simple, documented, tested, logged, and safe.

## Recommended Improvements

- Add better logging to every script.
- Add error handling to the Bash backup script.
- Add PowerShell comments and help sections.
- Add script parameters instead of hardcoded paths.
- Add the exact OU path to the PowerShell user creation script.
- Store sanitized examples only in GitHub.
- Add email or Telegram alerts for failed jobs.
- Add a test mode or dry-run option for rsync.
- Improve password handling for bulk user creation.
- Document how to safely disable or remove scheduled jobs.
- Compare manual cron patching with `unattended-upgrades`.

## Resume Bullets

- Automated Active Directory test user creation using PowerShell and CSV import, successfully creating bulk users inside a dedicated lab OU.
- Created and tested a Bash/rsync backup automation script on LINUX01, including troubleshooting script syntax and validating successful backup behavior.
- Configured Linux automation examples using cron and completed `unattended-upgrades` setup to practice scheduled maintenance and patching workflows.

## Interview Explanation

In my Automation Lab, I practiced automating common Junior SysAdmin tasks using PowerShell and Bash. On the Windows side, I created a CSV file with test users and used PowerShell to create Active Directory accounts from that CSV inside a specific OU. On the Linux side, I worked on LINUX01 with rsync backup automation, cron examples, and unattended-upgrades.

The main thing I learned is that automation is not just about making a script run. It also needs to be safe, tested, logged, and documented. I practiced checking output, fixing script syntax, adding log files, and removing sensitive information like hardcoded passwords before documenting the project on GitHub.
