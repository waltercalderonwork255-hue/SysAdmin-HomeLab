# File Server Lab Configs

## Overview

This document contains the configuration details for the File Server and SMB/CIFS Lab.

The purpose of this file is to document the main configuration choices used in the lab, including:

- Shared folders
- Share names
- Active Directory security groups
- Share permissions
- NTFS permissions
- Access-Based Enumeration
- Group Policy drive mapping
- Linux SMB/CIFS mount configuration
- Jellyfin media library paths

No passwords, private keys, or secrets are included in this documentation.

---

## Lab Environment

| Component | Configuration |
|---|---|
| File Server | `DC01` |
| File Server Role | File sharing hosted on Domain Controller for lab purposes |
| Domain | `corp.local` |
| Client Machines | `client01`, `client02` |
| Linux Server | `LINUX01` |
| Media Server | Jellyfin |
| File Sharing Protocol | SMB/CIFS |
| File Share Location | `C:\Shares` |
| Linux Mount Point | `/mnt/media` |
| Jellyfin Port | `8096` |

> Note: In a real production environment, the file server should normally be separate from the Domain Controller.

---

## Folder Structure on DC01

The following folders were created on `DC01`:

```text
C:\Shares
├── HR
├── Sales
├── IT
├── Executives
└── Media
