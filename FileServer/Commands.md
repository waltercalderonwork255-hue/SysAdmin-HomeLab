# File Server Lab Commands

## Overview

This document contains the commands used during the File Server and SMB/CIFS Lab.

The lab included:

- Testing Windows file share access
- Applying Group Policy updates
- Installing Jellyfin on Linux
- Adding the Jellyfin repository
- Mounting a Windows SMB/CIFS share on Linux
- Verifying the mounted media share
- Opening the Jellyfin firewall port

No passwords or secrets are included in this document. Any sensitive values were replaced with placeholders.

---

## Windows File Share Access Commands

### Open DC01 Shares from Windows Client

Used on `client01` and `client02` to browse available shares on the file server.

```text
\\dc01
```

### Open HR Share Directly

```text
\\dc01\HR
```

### Open Sales Share Directly

```text
\\dc01\Sales
```

### Open IT Share Directly

```text
\\dc01\IT
```

### Open Executives Share Directly

```text
\\dc01\Executives
```

### Open Media Share Directly

```text
\\dc01\Media
```

---

## Windows Run Menu Shortcut

Used to open the Run box on Windows clients.

```text
WIN + R
```

Then enter the file server path:

```text
\\dc01
```

---

## Group Policy Commands

### Force Group Policy Update

Used on Windows clients after configuring drive mapping with Group Policy.

```cmd
gpupdate /force
```

### Purpose

This command forces the client machine to pull updated Group Policy settings from the domain.

It was used to apply mapped drives for:

| Department | Drive Letter | Network Path |
|---|---|---|
| HR | `H:` | `\\dc01\HR` |
| Sales | `S:` | `\\dc01\Sales` |
| IT | `I:` | `\\dc01\IT` |
| Executives | `E:` | `\\dc01\Executives` |

---

## Jellyfin Installation Commands

These commands were used on `LINUX01`.

### Update Package List

```bash
sudo apt update
```

### Install Required Dependencies

```bash
sudo apt install apt-transport-https ca-certificates curl gnupg
```

### Add Jellyfin GPG Key

```bash
curl -fsSL https://repo.jellyfin.org/jellyfin_team.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/jellyfin.gpg
```

### Add Jellyfin Repository

```bash
echo "deb [signed-by=/usr/share/keyrings/jellyfin.gpg] https://repo.jellyfin.org/ubuntu noble main" | sudo tee /etc/apt/sources.list.d/jellyfin.list
```

### Update Package List Again

```bash
sudo apt update
```

### Install Jellyfin

```bash
sudo apt install jellyfin
```

---

## Jellyfin Repository Troubleshooting Commands

These commands were used after the Jellyfin repository line was entered incorrectly.

### Open Jellyfin Repository File

```bash
sudo vim /etc/apt/sources.list.d/jellyfin.list
```

### Correct Repository Line

The corrected line should look like this:

```text
deb [signed-by=/usr/share/keyrings/jellyfin.gpg] https://repo.jellyfin.org/ubuntu noble main
```

### Update Package List After Fixing Repository

```bash
sudo apt update
```

---

## Linux SMB/CIFS Mount Commands

These commands were used on `LINUX01` to mount the Windows `Media` share.

### Create Media Mount Point

```bash
sudo mkdir -p /mnt/media
```

### Mount Windows Media Share

```bash
sudo mount -t cifs //192.168.58.10/Media /mnt/media -o username=YOUR_USERNAME,password=YOUR_PASSWORD
```

### Security Note

Do not store a real username or password in GitHub.

Use placeholders:

```text
YOUR_USERNAME
YOUR_PASSWORD
```

---

## Verify SMB/CIFS Mount

### List Mounted Media Folder

```bash
ls /mnt/media
```

Expected output:

```text
movies
tv
```

---

## Jellyfin Firewall Command

### Allow Jellyfin Port Through UFW

```bash
sudo ufw allow 8096
```

### Purpose

This command allows access to the Jellyfin web interface on port `8096`.

Jellyfin web URL format:

```text
http://<LINUX_IP>:8096
```

Example from this lab:

```text
http://192.168.58.133:8096
```

---

## Jellyfin Web Access Paths

### Generic Jellyfin URL

```text
http://<LINUX_IP>:8096
```

### Lab Jellyfin URL

```text
http://192.168.58.133:8096
```

---

## Jellyfin Library Paths

These paths were added in Jellyfin after the SMB/CIFS share was mounted.

### Movies Library Path

```text
/mnt/media/movies
```

### TV Library Path

```text
/mnt/media/tv
```

---

## Common Command Mistakes Fixed

During the lab, some commands failed because spaces were missing.

### Incorrect

```bash
sudomkdir-p /mnt/media
```

### Correct

```bash
sudo mkdir -p /mnt/media
```

---

### Incorrect

```bash
sudo mount-t cifs //192.168.58.10/Media /mnt/media -o username=YOUR_USERNAME,password=YOUR_PASSWORD
```

### Correct

```bash
sudo mount -t cifs //192.168.58.10/Media /mnt/media -o username=YOUR_USERNAME,password=YOUR_PASSWORD
```

---

### Incorrect

```bash
sudo ufw allow8096
```

### Correct

```bash
sudo ufw allow 8096
```

---

## Command Summary

| Task | Command |
|---|---|
| Browse file server shares | `\\dc01` |
| Access HR share | `\\dc01\HR` |
| Access Sales share | `\\dc01\Sales` |
| Access IT share | `\\dc01\IT` |
| Access Executives share | `\\dc01\Executives` |
| Access Media share | `\\dc01\Media` |
| Force Group Policy update | `gpupdate /force` |
| Update Linux packages | `sudo apt update` |
| Install Jellyfin dependencies | `sudo apt install apt-transport-https ca-certificates curl gnupg` |
| Add Jellyfin GPG key | `curl -fsSL https://repo.jellyfin.org/jellyfin_team.gpg.key \| sudo gpg --dearmor -o /usr/share/keyrings/jellyfin.gpg` |
| Add Jellyfin repository | `echo "deb [signed-by=/usr/share/keyrings/jellyfin.gpg] https://repo.jellyfin.org/ubuntu noble main" \| sudo tee /etc/apt/sources.list.d/jellyfin.list` |
| Install Jellyfin | `sudo apt install jellyfin` |
| Edit Jellyfin repo file | `sudo vim /etc/apt/sources.list.d/jellyfin.list` |
| Create media mount point | `sudo mkdir -p /mnt/media` |
| Mount Windows Media share | `sudo mount -t cifs //192.168.58.10/Media /mnt/media -o username=YOUR_USERNAME,password=YOUR_PASSWORD` |
| Verify mounted share | `ls /mnt/media` |
| Allow Jellyfin port | `sudo ufw allow 8096` |

---

## Notes

- Windows share paths were tested from `client01` and `client02`.
- `gpupdate /force` was used after configuring mapped drives through Group Policy.
- The SMB/CIFS mount connected the Windows `Media` share to Linux at `/mnt/media`.
- Jellyfin used `/mnt/media/movies` and `/mnt/media/tv` as library paths.
- Real credentials should never be committed to GitHub.
- The mount command in this document uses placeholders instead of real credentials.
