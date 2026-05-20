# Automation Diagrams

## Overview

This folder contains diagrams for the Automation Lab in my SysAdmin Home Lab project.

These diagrams explain how automation was used across Windows and Linux systems, including PowerShell automation, Active Directory bulk user creation, Bash scripting, rsync backups, cron examples, unattended-upgrades, and logging.

This was completed in a home lab environment for Junior System Administrator practice.

---

## Diagram 1: Automation Lab Overview

```mermaid
flowchart TD
    A["Automation Lab"] --> B["Windows Automation"]
    A --> C["Linux Automation"]

    B --> D["DC01"]
    D --> E["PowerShell"]
    E --> F["Generate users.csv"]
    E --> G["Bulk Create AD Users"]
    G --> H["Specific Lab OU"]

    C --> I["LINUX01"]
    I --> J["Bash Script"]
    I --> K["Cron Examples"]
    I --> L["unattended-upgrades"]

    J --> M["backup.sh"]
    M --> N["rsync Backup"]
    N --> O["Backup Log"]

    K --> P["Update Cron Example"]
    P --> Q["Update Log"]

    L --> R["Automatic Linux Updates"]
    R --> S["unattended-upgrades Log"]
```

### Explanation

This diagram shows the overall Automation Lab. DC01 was used for PowerShell and Active Directory automation. LINUX01 was used for Bash scripting, rsync backup automation, cron examples, and unattended-upgrades.

---

## Diagram 2: PowerShell Active Directory Automation

```mermaid
flowchart TD
    A["Start on DC01"] --> B["Run PowerShell"]
    B --> C["Create 50 Test Users"]
    C --> D["Export users.csv"]
    D --> E["Import CSV"]
    E --> F["Run New-ADUser"]
    F --> G["Create AD User Accounts"]
    G --> H["Place Users in Specific OU"]
    H --> I["Verify with Get-ADUser"]
```

### Explanation

This diagram shows the PowerShell workflow used to create test users in Active Directory.

The process started by generating a CSV file, importing it into PowerShell, and using `New-ADUser` to create the accounts.

---

## Diagram 3: CSV to Active Directory Workflow

```mermaid
flowchart LR
    A["users.csv"] --> B["Import-Csv"]
    B --> C["ForEach-Object"]
    C --> D["Read Name"]
    C --> E["Read Username"]
    D --> F["New-ADUser"]
    E --> F
    F --> G["Create TestUser Accounts"]
    G --> H["Specific Lab OU"]
```

### Explanation

This diagram shows how the CSV file was used as the input source for bulk Active Directory user creation.

Each row in the CSV contained user information. PowerShell read each row and created the matching AD user account.

---

## Diagram 4: Linux Backup Automation

```mermaid
flowchart TD
    A["LINUX01"] --> B["backup.sh"]
    B --> C["Set Source Directory"]
    B --> D["Set Backup Destination"]
    C --> E["Source: /mnt/media"]
    D --> F["Destination: /backup/media"]
    E --> G["Run rsync"]
    F --> G
    G --> H["Sync Files"]
    H --> I["Write Output to backup.log"]
    I --> J["Review Log"]
```

### Explanation

This diagram shows the Linux backup automation workflow.

The `backup.sh` script defines a source and destination, runs `rsync`, copies files, and writes the output to a log file.

---

## Diagram 5: Backup Scheduling Example

```mermaid
flowchart TD
    A["Root Crontab"] --> B["Backup Cron Entry"]
    B --> C["Runs Daily at 2 AM"]
    C --> D["Starts backup.sh"]
    D --> E["Runs rsync Backup"]
    E --> F["Writes to backup.log"]
    F --> G["Review Log for Verification"]
```

### Explanation

This diagram shows how the backup script could be scheduled with cron.

The cron example runs the backup script every day at 2 AM and logs the results for later review.

---

## Diagram 6: Linux Update Cron Example

```mermaid
flowchart TD
    A["Root Crontab"] --> B["Update Cron Entry"]
    B --> C["Runs Daily at 3 AM"]
    C --> D["apt update"]
    D --> E["apt upgrade -y"]
    E --> F["Write Output to auto-updates.log"]
    F --> G["Review Log for Errors"]
```

### Explanation

This diagram shows the Linux update cron example.

The cron job runs package updates and sends output and errors to a log file so the results can be reviewed later.

---

## Diagram 7: unattended-upgrades Workflow

```mermaid
flowchart TD
    A["LINUX01"] --> B["Install unattended-upgrades"]
    B --> C["Configure unattended-upgrades"]
    C --> D["Automatic Package Updates"]
    D --> E["Write Update Logs"]
    E --> F["Review unattended-upgrades Logs"]
```

### Explanation

This diagram shows the basic workflow for `unattended-upgrades`.

Instead of only relying on a custom cron job for updates, `unattended-upgrades` provides a built-in Linux method for automatic package updates.

---

## Diagram 8: Logging and Troubleshooting Flow

```mermaid
flowchart TD
    A["Automation Task Runs"] --> B{"Did It Work?"}

    B -->|Yes| C["Review Output"]
    C --> D["Document Successful Test"]

    B -->|No| E["Check Log File"]
    E --> F["Read Error Message"]
    F --> G["Fix Script or Command"]
    G --> H["Test Manually"]
    H --> A

    E --> I["backup.log"]
    E --> J["auto-updates.log"]
    E --> K["unattended-upgrades.log"]
```

### Explanation

This diagram shows the troubleshooting process for automation tasks.

Logs are important because scheduled jobs can fail silently. Checking log files helps confirm whether the automation worked or failed.

---

## Diagram 9: GitHub-Safe Documentation Flow

```mermaid
flowchart TD
    A["Raw Lab Notes"] --> B["Review for Sensitive Information"]
    B --> C{"Contains Secrets?"}

    C -->|Yes| D["Remove Secret"]
    D --> E["Replace with Placeholder"]

    C -->|No| F["Keep Technical Details"]

    E --> G["Create GitHub Documentation"]
    F --> G

    G --> H["README.md"]
    G --> I["Commands"]
    G --> J["Configs"]
    G --> K["Diagrams"]
    G --> L["Writeups"]
```

### Explanation

This diagram shows how the raw lab notes were cleaned before being added to GitHub.

Passwords and sensitive values should be removed or replaced with placeholders before publishing documentation.

---

## Diagram 10: Automation Folder Structure

```mermaid
flowchart TD
    A["Automation"] --> B["README.md"]
    A --> C["Commands"]
    A --> D["Configs"]
    A --> E["Diagrams"]
    A --> F["Writeups"]

    C --> G["Commands README.md"]
    D --> H["Configs README.md"]
    E --> I["Diagrams README.md"]
    F --> J["Writeups README.md"]
```

### Explanation

This diagram shows the recommended folder structure for the Automation project inside the SysAdmin-HomeLab GitHub repo.

Each folder documents a different part of the project.

---

## Suggested Screenshots to Add Later

| Screenshot | What to Capture | Why It Matters | Suggested Filename |
|---|---|---|---|
| Automation overview | DC01 and LINUX01 automation tasks | Shows the full project scope | `automation-overview.png` |
| PowerShell CSV creation | PowerShell command creating `users.csv` | Shows user generation automation | `powershell-csv-generation.png` |
| Active Directory users | Test users created in the specific OU | Proves the AD automation worked | `ad-test-users-ou.png` |
| Backup script | `backup.sh` open in terminal or editor | Shows the Bash automation script | `backup-script.png` |
| Cron examples | Root crontab showing automation examples | Shows scheduling practice | `cron-examples.png` |
| Backup log | `backup.log` output | Shows backup verification | `backup-log-output.png` |
| unattended-upgrades | Status or configuration output | Shows automatic update configuration | `unattended-upgrades-status.png` |
| GitHub folder structure | Automation project folders in GitHub | Shows organized documentation | `automation-folder-structure.png` |

---

## GitHub Mermaid Notes

GitHub supports Mermaid diagrams inside markdown files.

If a diagram does not render:

1. Make sure the diagram starts with:

```text
```mermaid
```

2. Make sure the diagram ends with:

```text
```
```

3. Use quotes around node text.
4. Avoid complicated symbols in node names.
5. Keep diagrams simple.
6. Avoid using raw HTML tags like `<br>` inside Mermaid nodes.
7. Avoid overly complex paths or special characters if GitHub gives an error.

---

## Lessons Learned

- Diagrams make automation workflows easier to understand.
- PowerShell automation can be documented as a CSV-to-Active-Directory workflow.
- Linux automation can be shown as a script, scheduler, backup, and log workflow.
- Logging is an important part of automation because it helps verify whether scripts worked.
- Mermaid diagrams are useful because they can be written directly in GitHub markdown.
- GitHub documentation should be sanitized before publishing.
