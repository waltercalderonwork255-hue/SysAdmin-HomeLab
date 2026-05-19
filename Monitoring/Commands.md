# Monitoring Lab Commands

## Overview

This document contains the commands used during the Monitoring lab for the SysAdmin-HomeLab project.

The monitoring tool used was **Uptime Kuma**, deployed with **Docker** on **LINUX01**.

This lab focused on:

- Installing Docker
- Running Uptime Kuma in a container
- Verifying Docker status
- Allowing Windows clients to respond to ping
- Documenting basic monitoring-related commands

> Security note: Do not store Uptime Kuma passwords, email passwords, SMTP app passwords, tokens, or notification URLs in GitHub.

---

## System Information

| Item | Details |
|---|---|
| Monitoring Server | LINUX01 |
| Monitoring Server IP | 192.168.58.133 |
| Monitoring Tool | Uptime Kuma |
| Container Platform | Docker |
| Uptime Kuma Web Port | 3001 |
| Dashboard URL | `http://192.168.58.133:3001` |

---

## Linux Commands

### Update Package List

Used to refresh the package list before installing Docker.

```bash
sudo apt update
```

---

### Install Docker

Used to install Docker from the Ubuntu repositories.

```bash
sudo apt install docker.io -y
```

---

### Enable and Start Docker

Used to start Docker and enable it to run automatically at boot.

```bash
sudo systemctl enable --now docker
```

---

### Check Docker Service Status

Used to verify that Docker is running.

```bash
sudo systemctl status docker
```

---

## Uptime Kuma Docker Commands

### Deploy Uptime Kuma

Used to run Uptime Kuma in Docker.

```bash
sudo docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
```

### Command Breakdown

| Part | Meaning |
|---|---|
| `sudo docker run` | Runs a new Docker container |
| `-d` | Runs the container in the background |
| `--restart=always` | Restarts the container automatically if it stops |
| `-p 3001:3001` | Maps port 3001 on the server to port 3001 in the container |
| `-v uptime-kuma:/app/data` | Creates persistent storage for Uptime Kuma data |
| `--name uptime-kuma` | Names the container `uptime-kuma` |
| `louislam/uptime-kuma:1` | Uptime Kuma Docker image |

---

### Access Uptime Kuma Dashboard

After deploying the container, access the dashboard in a browser:

```text
http://192.168.58.133:3001
```

---

## Helpful Docker Verification Commands

These commands were not all required, but they are useful for checking and troubleshooting the Uptime Kuma container.

### List Running Containers

```bash
sudo docker ps
```

### List All Containers

```bash
sudo docker ps -a
```

### View Uptime Kuma Logs

```bash
sudo docker logs uptime-kuma
```

### Follow Uptime Kuma Logs Live

```bash
sudo docker logs -f uptime-kuma
```

### Stop Uptime Kuma Container

```bash
sudo docker stop uptime-kuma
```

### Start Uptime Kuma Container

```bash
sudo docker start uptime-kuma
```

### Restart Uptime Kuma Container

```bash
sudo docker restart uptime-kuma
```

---

## Windows Commands

### Allow ICMP Ping Through Windows Firewall

Run this command in **PowerShell as Administrator** on each Windows client if ping checks fail.

```powershell
netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow
```

### Why This Was Needed

Uptime Kuma used ping checks for:

- CLIENT01
- CLIENT02

Windows Firewall may block incoming ping requests by default. If ICMP is blocked, the client can appear down in Uptime Kuma even when the computer is actually online.

---

### Test Ping from Another System

Use this to test if a Windows client responds to ping.

```powershell
ping CLIENT01
```

```powershell
ping CLIENT02
```

Or test by IP address if DNS is not working:

```powershell
ping <CLIENT01-IP-ADDRESS>
```

```powershell
ping <CLIENT02-IP-ADDRESS>
```

---

## Monitoring Checks Configured in Uptime Kuma

These were configured in the Uptime Kuma web dashboard, not from the terminal.

### DC01 Monitor

| Setting | Value |
|---|---|
| Monitor Type | TCP Port |
| Port | 389 |
| Purpose | Check LDAP / Active Directory-related service availability |
| Status | Online |

---

### CLIENT01 Monitor

| Setting | Value |
|---|---|
| Monitor Type | Ping |
| Purpose | Check if the client is online |
| Status | Online |

---

### CLIENT02 Monitor

| Setting | Value |
|---|---|
| Monitor Type | Ping |
| Purpose | Check if the client is online |
| Status | Online |

---

### Jellyfin Monitor

| Setting | Value |
|---|---|
| Monitor Type | TCP Port |
| Hostname | 192.168.58.133 |
| Port | 8096 |
| Purpose | Check if Jellyfin service is reachable |
| Status | Online |

---

## Email Alert Configuration

Email alerts were configured in the Uptime Kuma dashboard.

Exact email, SMTP, password, app password, or token settings are intentionally not documented for security reasons.

### Safe Documentation Format

Use this format in documentation instead of exposing secrets:

```text
Notification Type: Email
SMTP Server: Redacted
SMTP Username: Redacted
SMTP Password/App Password: Redacted
Recipient Email: Redacted
Status: Configured
```

---

## Recommended Uptime Kuma Settings

These settings help reduce false alarms.

| Setting | Value |
|---|---|
| Heartbeat Interval | 60 seconds |
| Retries | 3 |
| Resend Notification | 0 / Disabled |

---

## Troubleshooting Commands

### Check if Docker Is Running

```bash
sudo systemctl status docker
```

---

### Restart Docker

```bash
sudo systemctl restart docker
```

---

### Restart Uptime Kuma

```bash
sudo docker restart uptime-kuma
```

---

### Check if Uptime Kuma Container Exists

```bash
sudo docker ps -a
```

---

### Check Uptime Kuma Logs

```bash
sudo docker logs uptime-kuma
```

---

### Check if Port 3001 Is Listening

```bash
sudo ss -tulpn | grep 3001
```

---

### Test Jellyfin Port from Linux

```bash
nc -vz 192.168.58.133 8096
```

If `nc` is not installed:

```bash
sudo apt install netcat-openbsd -y
```

Then run:

```bash
nc -vz 192.168.58.133 8096
```

---

### Test DC01 LDAP Port from Linux

Replace `<DC01-IP-ADDRESS>` with the IP address of DC01.

```bash
nc -vz <DC01-IP-ADDRESS> 389
```

---

## Commands I Actually Used

These are the main commands used directly in this lab:

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo systemctl status docker
sudo docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
```

Windows PowerShell command:

```powershell
netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow
```

---

## Notes

- Uptime Kuma was accessed at `http://192.168.58.133:3001`.
- DC01 was monitored using TCP port `389`.
- CLIENT01 and CLIENT02 were monitored using ping.
- Jellyfin was monitored using TCP port `8096`.
- Email alerts were configured.
- VPN monitoring was not completed and is listed as a future improvement.
- No CPU, RAM, disk, or log monitoring was configured in this project.
