# Monitoring Lab Configs

## Overview

This document contains the configuration details used for the Monitoring lab in my SysAdmin-HomeLab project.

The monitoring tool used was **Uptime Kuma**, running in Docker on **LINUX01**. This lab focused on basic uptime and service availability monitoring for important home lab systems.

This document does **not** include passwords, SMTP credentials, notification tokens, or any other sensitive information.

## Security Note

Do not upload the following to GitHub:

- Uptime Kuma passwords
- Email passwords
- SMTP app passwords
- API tokens
- Notification URLs
- Private keys
- Recovery codes
- Any real credentials

Use placeholders or redacted values instead.

Example:

```text
SMTP Username: REDACTED
SMTP Password: REDACTED
Notification URL: REDACTED
```

## Monitoring Server Configuration

| Setting | Value |
|---|---|
| Monitoring Server | LINUX01 |
| Monitoring Server IP | 192.168.58.133 |
| Monitoring Tool | Uptime Kuma |
| Container Platform | Docker |
| Uptime Kuma Web Port | 3001 |
| Dashboard URL | `http://192.168.58.133:3001` |
| Data Storage | Docker volume: `uptime-kuma` |
| Restart Policy | `always` |

## Docker Container Configuration

Uptime Kuma was deployed as a Docker container.

```bash
sudo docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
```

### Container Settings

| Setting | Value |
|---|---|
| Container Name | uptime-kuma |
| Image | louislam/uptime-kuma:1 |
| Host Port | 3001 |
| Container Port | 3001 |
| Volume | uptime-kuma |
| Container Data Path | /app/data |
| Restart Policy | always |
| Run Mode | Detached |

### Port Mapping

| Host | Port |
|---|---|
| LINUX01 | 3001 |
| Uptime Kuma Container | 3001 |

### Volume Mapping

| Docker Volume | Container Path | Purpose |
|---|---|---|
| uptime-kuma | /app/data | Stores Uptime Kuma configuration and monitoring data |

## Uptime Kuma Dashboard Access

The dashboard was accessed from a browser using:

```text
http://192.168.58.133:3001
```

Credentials were created during setup, but they are intentionally not documented here.

```text
Username: REDACTED
Password: REDACTED
```

## Monitors Configured

### DC01 Monitor

| Setting | Value |
|---|---|
| Monitor Name | DC01 |
| Monitor Type | TCP Port |
| Hostname / IP | Needs clarification |
| Port | 389 |
| Heartbeat Interval | 60 seconds |
| Retries | 3 |
| Status | Online |
| Purpose | Monitor LDAP / Active Directory-related service availability |

### CLIENT01 Monitor

| Setting | Value |
|---|---|
| Monitor Name | CLIENT01 |
| Monitor Type | Ping |
| Hostname / IP | Needs clarification |
| Heartbeat Interval | 60 seconds |
| Retries | 3 |
| Status | Online |
| Purpose | Monitor Windows client availability |

### CLIENT02 Monitor

| Setting | Value |
|---|---|
| Monitor Name | CLIENT02 |
| Monitor Type | Ping |
| Hostname / IP | Needs clarification |
| Heartbeat Interval | 60 seconds |
| Retries | 3 |
| Status | Online |
| Purpose | Monitor Windows client availability |

### Jellyfin Monitor

| Setting | Value |
|---|---|
| Monitor Name | Jellyfin |
| Monitor Type | TCP Port |
| Hostname / IP | 192.168.58.133 |
| Port | 8096 |
| Heartbeat Interval | 60 seconds |
| Retries | 3 |
| Status | Online |
| Purpose | Monitor Jellyfin service availability |

## Initial Jellyfin Monitor Attempt

The first Jellyfin monitor idea used an HTTP keyword check.

| Setting | Value |
|---|---|
| Monitor Type | HTTP(s) - Keyword |
| URL | `http://192.168.58.133:8096` |
| Keyword | Jellyfin |
| Result | Did not work reliably |

The monitor was changed to a TCP port check on port `8096`.

## Email Alert Configuration

Email alerts were configured in Uptime Kuma.

Sensitive values are intentionally redacted.

| Setting | Value |
|---|---|
| Notification Type | Email |
| SMTP Server | REDACTED |
| SMTP Port | REDACTED |
| SMTP Username | REDACTED |
| SMTP Password / App Password | REDACTED |
| Sender Email | REDACTED |
| Recipient Email | REDACTED |
| Status | Configured |

## Global Monitoring Settings

These settings were used or documented as best-practice settings for the lab.

| Setting | Value |
|---|---|
| Heartbeat Interval | 60 seconds |
| Retries | 3 |
| Resend Notification | 0 / Disabled |

## Windows Firewall Configuration

Ping monitoring was used for CLIENT01 and CLIENT02.

If Windows Firewall blocks ICMP, the client may appear down even when it is online. To allow ping checks, this rule was used on the Windows clients.

```powershell
netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow
```

### Firewall Rule Details

| Setting | Value |
|---|---|
| Rule Name | ICMP Allow incoming V4 echo request |
| Protocol | ICMPv4 |
| ICMP Type | 8 |
| Direction | Inbound |
| Action | Allow |
| Purpose | Allow ping monitoring from Uptime Kuma |

## VPN Monitoring Configuration

VPN monitoring was not completed in this project.

It is listed as a future improvement because the VPN setup was still having issues.

Possible future VPN monitoring options:

### Option 1: Internal VPN Ping Check

| Setting | Value |
|---|---|
| Monitor Type | Ping |
| Target | Internal VPN IP |
| Example Target | 10.0.0.1 |
| Purpose | Check whether the VPN internal interface is reachable |

### Option 2: Push Monitor

| Setting | Value |
|---|---|
| Monitor Type | Push |
| Method | VPN server sends heartbeat to Uptime Kuma |
| Schedule | Cron job every minute |
| Purpose | Confirm that the VPN server is alive even if the VPN port does not respond publicly |

Sensitive push URLs should never be uploaded to GitHub.

## Performance Monitoring Configuration

CPU, RAM, and disk monitoring were not configured in this project.

This lab focused on:

- Host availability
- Service availability
- TCP port monitoring
- Ping monitoring
- Email alerts

Future improvements could include:

- Disk usage monitoring
- CPU usage monitoring
- RAM usage monitoring
- Docker container health monitoring

## Log Monitoring Configuration

Log monitoring was not configured in this project.

No logs were reviewed during this monitoring lab.

Future logs to review:

| System | Logs to Review |
|---|---|
| DC01 | Windows Event Viewer |
| CLIENT01 | Windows Event Viewer |
| CLIENT02 | Windows Event Viewer |
| LINUX01 | Linux system logs |
| Docker | Container logs |
| Uptime Kuma | Uptime Kuma container logs |
| Jellyfin | Jellyfin logs |

## Configuration Summary

| Item | Status |
|---|---|
| Docker installed | Completed |
| Docker enabled on boot | Completed |
| Uptime Kuma deployed | Completed |
| Dashboard accessible | Completed |
| DC01 monitor configured | Completed |
| CLIENT01 monitor configured | Completed |
| CLIENT02 monitor configured | Completed |
| Jellyfin monitor configured | Completed |
| Email alerts configured | Completed |
| VPN monitoring | Future improvement |
| Log monitoring | Not configured |
| CPU/RAM/disk monitoring | Not configured |

## Notes

- Uptime Kuma was used for uptime and service monitoring only.
- DC01 was monitored using TCP port `389`.
- CLIENT01 and CLIENT02 were monitored using ping.
- Jellyfin was monitored using TCP port `8096`.
- Email alerts were configured.
- Logs were not reviewed in this project.
- Performance monitoring was not configured.
- VPN monitoring was kept as a future improvement.
- Sensitive credentials were removed from this documentation.
