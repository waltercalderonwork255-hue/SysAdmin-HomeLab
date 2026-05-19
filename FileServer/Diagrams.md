# File Server Lab Diagrams

## Overview

This document contains diagrams for the File Server and SMB/CIFS Lab.

The diagrams show:

- Overall file server lab topology
- Department share design
- Permission model
- Group Policy drive mapping
- SMB/CIFS media share connection to Jellyfin
- Access-Based Enumeration behavior

These diagrams are written in Mermaid so they can render directly in GitHub.

---

## Lab Topology

```mermaid
flowchart TD
    DC01["DC01<br>Windows Server<br>Domain Controller<br>File Shares"]
    CLIENT01["client01<br>Windows Client"]
    CLIENT02["client02<br>Windows Client"]
    LINUX01["LINUX01<br>Linux Server<br>Jellyfin"]
    JELLYFIN["Jellyfin Web UI<br>Port 8096"]

    CLIENT01 -->|"Domain Login<br>SMB Access"| DC01
    CLIENT02 -->|"Domain Login<br>SMB Access"| DC01
    LINUX01 -->|"CIFS Mount<br>//192.168.58.10/Media"| DC01
    JELLYFIN -->|"Reads Media From<br>/mnt/media"| LINUX01
```

---

## File Share Structure on DC01

```mermaid
flowchart TD
    ROOT["C:\\Shares"]

    ROOT --> HR["HR Share<br>C:\\Shares\\HR"]
    ROOT --> SALES["Sales Share<br>C:\\Shares\\Sales"]
    ROOT --> IT["IT Share<br>C:\\Shares\\IT"]
    ROOT --> EXEC["Executives Share<br>C:\\Shares\\Executives"]
    ROOT --> MEDIA["Media Share<br>C:\\Shares\\Media"]

    HR --> HRPERM["GRP_HR: Modify<br>Admins: Full Control"]
    SALES --> SALESPERM["GRP_Sales: Modify<br>Admins: Full Control"]
    IT --> ITPERM["GRP_IT: Modify<br>Admins: Full Control"]
    EXEC --> EXECPERM["GRP_Executives: Read/Write<br>Admins: Full Control"]
    MEDIA --> MEDIAPERM["Windows Clients: Read Only<br>Admins: Full Control"]
```

---

## Department Access Model

```mermaid
flowchart LR
    HRUSER["HR User"]
    SALESUSER["Sales User"]
    ITUSER["IT User"]
    EXECUSER["Executive User"]
    ADMIN["Admin Account"]

    GRPHR["GRP_HR"]
    GRPSALES["GRP_Sales"]
    GRPIT["GRP_IT"]
    GRPEXEC["GRP_Executives"]

    HRSHARE["HR Share"]
    SALESSHARE["Sales Share"]
    ITSHARE["IT Share"]
    EXECSHARE["Executives Share"]

    HRUSER --> GRPHR -->|"Modify"| HRSHARE
    SALESUSER --> GRPSALES -->|"Modify"| SALESSHARE
    ITUSER --> GRPIT -->|"Modify"| ITSHARE
    EXECUSER --> GRPEXEC -->|"Read/Write"| EXECSHARE

    ADMIN -->|"Full Control"| HRSHARE
    ADMIN -->|"Full Control"| SALESSHARE
    ADMIN -->|"Full Control"| ITSHARE
    ADMIN -->|"Full Control"| EXECSHARE
```

---

## Permission Flow

```mermaid
flowchart TD
    USER["User logs into domain client"]
    GROUP["User membership checked<br>Active Directory Security Group"]
    SHAREPERM["Share Permission Checked"]
    NTFSPERM["NTFS Permission Checked"]
    DECISION{"Allowed by both?"}
    ACCESS["Access Granted"]
    DENIED["Access Denied"]

    USER --> GROUP
    GROUP --> SHAREPERM
    SHAREPERM --> NTFSPERM
    NTFSPERM --> DECISION
    DECISION -->|"Yes"| ACCESS
    DECISION -->|"No"| DENIED
```

---

## Most Restrictive Permission Rule

```mermaid
flowchart LR
    SHARE["Share Permission"]
    NTFS["NTFS Permission"]
    RESULT["Final Effective Permission"]

    SHARE --> RESULT
    NTFS --> RESULT

    RESULT --> NOTE["Most restrictive permission wins"]
```

---

## Access-Based Enumeration

```mermaid
flowchart TD
    ABE["Access-Based Enumeration Enabled"]

    HRUSER["HR User"]
    SALESUSER["Sales User"]
    EXECUSER["Executive User"]

    HRVIEW["Visible: HR Only"]
    SALESVIEW["Visible: Sales Only"]
    EXECVIEW["Visible: Executives Only"]

    HIDDEN1["Sales / IT / Executives Hidden"]
    HIDDEN2["HR / IT / Executives Hidden"]
    HIDDEN3["HR / Sales / IT Hidden"]

    ABE --> HRUSER
    ABE --> SALESUSER
    ABE --> EXECUSER

    HRUSER --> HRVIEW --> HIDDEN1
    SALESUSER --> SALESVIEW --> HIDDEN2
    EXECUSER --> EXECVIEW --> HIDDEN3
```

---

## Group Policy Drive Mapping

```mermaid
flowchart TD
    GPO["Drive Mapping Policy<br>Linked to _Users OU"]

    HRGROUP["GRP_HR"]
    SALESGROUP["GRP_Sales"]
    ITGROUP["GRP_IT"]
    EXECGROUP["GRP_Executives"]

    HDRIVE["H: Drive<br>\\\\dc01\\HR"]
    SDRIVE["S: Drive<br>\\\\dc01\\Sales"]
    IDRIVE["I: Drive<br>\\\\dc01\\IT"]
    EDRIVE["E: Drive<br>\\\\dc01\\Executives"]

    GPO -->|"Item-level targeting"| HRGROUP
    GPO -->|"Item-level targeting"| SALESGROUP
    GPO -->|"Item-level targeting"| ITGROUP
    GPO -->|"Item-level targeting"| EXECGROUP

    HRGROUP --> HDRIVE
    SALESGROUP --> SDRIVE
    ITGROUP --> IDRIVE
    EXECGROUP --> EDRIVE
```

---

## Client Access Testing

```mermaid
flowchart TD
    CLIENTS["client01 and client02"]

    HRTEST["HR User Test"]
    SALESTEST["Sales User Test"]
    EXECTEST["Executive User Test"]
    ADMINTEST["Admin Test"]

    HRPASS["Can access HR<br>Denied from Sales"]
    SALESPASS["Can access Sales<br>Denied from HR"]
    EXECPASS["Can access Executives"]
    ADMINPASS["Full Control access"]

    CLIENTS --> HRTEST --> HRPASS
    CLIENTS --> SALESTEST --> SALESPASS
    CLIENTS --> EXECTEST --> EXECPASS
    CLIENTS --> ADMINTEST --> ADMINPASS
```

---

## Media Share and Jellyfin Flow

```mermaid
flowchart TD
    DC01["DC01<br>Windows File Server"]
    MEDIASHARE["Media Share<br>\\\\dc01\\Media<br>C:\\Shares\\Media"]
    LINUX01["LINUX01<br>Linux Server"]
    MOUNT["/mnt/media<br>CIFS Mount"]
    MOVIES["/mnt/media/movies"]
    TV["/mnt/media/tv"]
    JELLYFIN["Jellyfin<br>http://LINUX_IP:8096"]
    CLIENT01["client01"]
    CLIENT02["client02"]

    DC01 --> MEDIASHARE
    MEDIASHARE -->|"Mounted with CIFS"| LINUX01
    LINUX01 --> MOUNT
    MOUNT --> MOVIES
    MOUNT --> TV
    MOVIES --> JELLYFIN
    TV --> JELLYFIN

    CLIENT01 -->|"Web Browser"| JELLYFIN
    CLIENT02 -->|"Web Browser"| JELLYFIN
```

---

## Media Share Permission Design

```mermaid
flowchart LR
    USERS["Windows Clients"]
    ADMINS["Admin Accounts"]
    MEDIA["Media Share"]

    USERS -->|"Read Only<br>View videos"| MEDIA
    ADMINS -->|"Full Control<br>Manage media files"| MEDIA
```

---

## Production Design Comparison

```mermaid
flowchart TD
    subgraph LAB["Lab Design"]
        DC01LAB["DC01<br>AD DS / DNS / File Shares"]
        CLIENTLAB["client01 / client02"]
        CLIENTLAB --> DC01LAB
    end

    subgraph PROD["Better Production Design"]
        DC01PROD["DC01<br>AD DS / DNS / Authentication"]
        FS01["FS01<br>File Server"]
        CLIENTPROD["Client Machines"]

        CLIENTPROD --> DC01PROD
        CLIENTPROD --> FS01
        FS01 --> DC01PROD
    end
```

---

## Diagram Summary

| Diagram | Purpose |
|---|---|
| Lab Topology | Shows how DC01, Windows clients, LINUX01, and Jellyfin connect |
| File Share Structure | Shows the folders and shares created on DC01 |
| Department Access Model | Shows how users, groups, and shares connect |
| Permission Flow | Shows how share and NTFS permissions are checked |
| Most Restrictive Permission Rule | Shows that the most restrictive permission wins |
| Access-Based Enumeration | Shows how users only see allowed shares |
| Group Policy Drive Mapping | Shows mapped drives based on security groups |
| Client Access Testing | Shows permission testing from client machines |
| Media Share and Jellyfin Flow | Shows how Jellyfin reads media from the SMB/CIFS mount |
| Media Share Permission Design | Shows read-only client access and admin Full Control |
| Production Design Comparison | Shows why a separate file server is better in production |

---

## Notes

- These diagrams are written in Mermaid format.
- GitHub can render Mermaid diagrams directly inside Markdown files.
- The lab used `DC01` as the file server for simplicity.
- In production, file shares should usually be hosted on a separate file server such as `FS01`.
- No passwords or secrets are included in these diagrams.
