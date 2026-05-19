# Monitoring Lab

## Overview

This project documents a basic monitoring setup for my SysAdmin home lab. I used **Uptime Kuma** running in Docker on a Linux server to monitor important lab systems and services.

The purpose of this project was to practice how system administrators check whether servers, clients, and services are online and reachable. This lab focused on **uptime and service availability monitoring**, not full performance monitoring.

I monitored my domain controller, Windows clients, and Jellyfin media server from a central dashboard. I also configured email alerts so I could be notified when a monitored system or service goes down.

## Goal

The goal of this project was to build a simple monitoring dashboard that could help identify outages in my home lab.

I wanted to practice:

- Installing Docker on Linux
- Deploying a monitoring tool in a container
- Monitoring Windows and Linux systems
- Checking service availability with ping and TCP port checks
- Monitoring Active Directory-related service availability
- Monitoring a web-based application service
- Configuring email alerts
- Documenting troubleshooting and future improvements

## Skills Demonstrated

- Linux server administration
- Docker installation and service management
- Container deployment
- Uptime monitoring
- TCP port monitoring
- Ping monitoring
- Basic service availability checks
- Email alert configuration
- Windows Firewall troubleshooting
- Monitoring documentation
- Beginner-level troubleshooting

## Environment

| Component | Details |
|---|---|
| Monitoring Tool | Uptime Kuma |
| Monitoring Host | LINUX01 |
| Monitoring Host IP | 192.168.58.133 |
| Container Platform | Docker |
| Uptime Kuma Port | 3001 |
| Monitored Systems | DC01, CLIENT01, CLIENT02, Jellyfin |
| DC01 Monitor | TCP port check on port 389 |
| Client Monitor Type | Ping |
| Jellyfin Monitor | TCP port check on port 8096 |
| Alerts | Email alerts configured |
| Logs Reviewed | None |
| Performance Monitoring | Not configured; uptime/service monitoring only |
| VPN Monitoring | Future improvement |

## Monitoring Plan

The monitoring plan focused on checking whether important lab systems and services were available.

### What Systems Were Monitored

| System | Purpose | Monitor Type |
|---|---|---|
| DC01 | Domain Controller / Active Directory server | TCP Port |
| CLIENT01 | Windows domain client | Ping |
| CLIENT02 | Windows domain client | Ping |
| Jellyfin | Media server web application | TCP Port |

### What Services Were Monitored

| System | Service / Check | Port | Why It Matters |
|---|---|---|---|
| DC01 | LDAP / Active Directory-related service availability | 389 | Confirms an important AD-related port is reachable |
| Jellyfin | Jellyfin web service | 8096 | Confirms the Jellyfin service is reachable |
| CLIENT01 | Network availability | Ping | Confirms the client is online and reachable |
| CLIENT02 | Network availability | Ping | Confirms the client is online and reachable |

### What Logs Were Checked

No logs were reviewed during this project.

Future logs to review:

- Windows Event Viewer on DC01 and Windows clients
- Linux system logs on LINUX01
- Docker container logs
- Uptime Kuma logs
- Jellyfin service logs

### What Health Checks Were Used

The following health checks were used:

- Ping checks for Windows clients
- TCP port check for DC01 on port `389`
- TCP port check for Jellyfin on port `8096`
- Docker service status check for the monitoring host
- Uptime Kuma dashboard verification

### Why These Checks Matter

Monitoring helps sysadmins detect problems before users report them.

A server may be powered on and still have an important service unavailable. For example, a domain controller could respond to ping, but an Active Directory-related service could still be unreachable. Using a TCP port check gives better service-level visibility than only checking whether the server is powered on.

For this lab, Uptime Kuma helped provide a central place to view whether key systems and services were online.

## Project Steps

### Step 1: Installed Docker on LINUX01

**What I did:**  
I installed Docker on LINUX01.

**Why I did it:**  
Docker allowed me to run Uptime Kuma as a container without manually installing the application directly on the Linux server.

**Commands or settings used:**

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker
```

**Verification:**  
I checked that the Docker service was running.

```bash
sudo systemctl status docker
```

---

### Step 2: Deployed Uptime Kuma

**What I did:**  
I deployed Uptime Kuma using Docker.

**Why I did it:**  
Uptime Kuma provides a simple monitoring dashboard for checking system and service availability.

**Commands or settings used:**

```bash
sudo docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
```

**Verification:**  
After the container started, I opened the Uptime Kuma dashboard from my browser.

```text
http://192.168.58.133:3001
```

The Uptime Kuma dashboard loaded successfully.

> Security note: I created login credentials for Uptime Kuma, but passwords should not be stored in GitHub documentation.

---

### Step 3: Added DC01 Monitoring

**What I did:**  
I configured a TCP port monitor for DC01 using port `389`.

**Why I did it:**  
Port `389` is used for LDAP, which is related to Active Directory. Monitoring this port is more useful than only pinging the server because it checks whether an important AD-related service is reachable.

**Commands or settings used:**

| Setting | Value |
|---|---|
| Monitor Type | TCP Port |
| Port | 389 |
| Recommended Interval | 60 seconds |

**Verification:**  
Uptime Kuma showed DC01 as online.

---

### Step 4: Added Windows Client Monitoring

**What I did:**  
I added ping monitors for CLIENT01 and CLIENT02.

**Why I did it:**  
The Windows clients do not host major services in this lab, so a ping check was enough to confirm that they were online and reachable.

**Commands or settings used:**

| Setting | Value |
|---|---|
| Monitor Type | Ping |
| Systems | CLIENT01, CLIENT02 |
| Recommended Interval | 60 seconds |

**Verification:**  
Uptime Kuma showed CLIENT01 and CLIENT02 as online.

---

### Step 5: Allowed ICMP Through Windows Firewall

**What I did:**  
I used a Windows Firewall rule to allow ICMP ping traffic on the Windows clients.

**Why I did it:**  
If a Windows client is powered on but blocks ping, Uptime Kuma may show the client as down even though it is reachable in other ways.

**Commands or settings used:**  
Run in PowerShell as Administrator on the Windows client:

```powershell
netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow
```

**Verification:**  
After allowing ICMP, the Windows clients were able to show as online in Uptime Kuma.

---

### Step 6: Added Jellyfin Monitoring

**What I did:**  
I added monitoring for the Jellyfin media server.

**Why I did it:**  
Jellyfin is a web-based service, so I wanted to confirm that the service was reachable from the network.

**Commands or settings used:**

Initial idea:

| Setting | Value |
|---|---|
| Monitor Type | HTTP(s) - Keyword |
| URL | `http://192.168.58.133:8096` |
| Keyword | Jellyfin |

Final working monitor:

| Setting | Value |
|---|---|
| Monitor Type | TCP Port |
| Hostname | 192.168.58.133 |
| Port | 8096 |

**Verification:**  
Uptime Kuma showed Jellyfin as online.

---

### Step 7: Configured Email Alerts

**What I did:**  
I configured email alerts in Uptime Kuma.

**Why I did it:**  
Alerts are important because they notify a sysadmin when a monitored system or service goes down.

**Commands or settings used:**  
Exact email or SMTP settings are not included for security reasons.

**Verification:**  
Email alerts were configured in Uptime Kuma.

---

### Step 8: Adjusted Monitoring Best Practice Settings

**What I did:**  
I documented monitoring settings to reduce false alarms.

**Why I did it:**  
Monitoring tools can create false alerts if they mark a system down after only one failed check. Retries help confirm that the issue is real before alerting.

**Commands or settings used:**

| Setting | Value |
|---|---|
| Retries | 3 |
| Heartbeat Interval | 60 seconds |
| Resend Notification | 0 / Disabled |

**Verification:**  
The monitors were configured with practical settings for a home lab environment.

## Troubleshooting

### Issue: Jellyfin Monitor Stayed Red

**Problem:**  
The Jellyfin monitor stayed red when using the HTTP keyword check.

**Cause:**  
The HTTP keyword check may have been too specific or unreliable for this setup. The service could still be reachable even if the keyword check failed.

**Fix:**  
I changed the Jellyfin monitor to a TCP port check.

Updated settings:

| Setting | Value |
|---|---|
| Monitor Type | TCP Port |
| Hostname | 192.168.58.133 |
| Port | 8096 |

**Verification:**  
Uptime Kuma showed Jellyfin as online after using the TCP port check.

**Lesson learned:**  
HTTP keyword checks are useful, but they can fail if the page content does not match exactly. TCP port checks are simpler and useful for basic service availability monitoring.

---

### Issue: Windows Clients May Show as Down

**Problem:**  
CLIENT01 or CLIENT02 may show as down in Uptime Kuma even when the VM is powered on.

**Cause:**  
Windows Firewall may block incoming ICMP ping requests by default.

**Fix:**  
I allowed ICMP echo requests through Windows Firewall.

```powershell
netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow
```

**Verification:**  
The Windows clients showed as online in Uptime Kuma.

**Lesson learned:**  
A failed ping does not always mean the system is offline. Firewalls can block monitoring traffic.

## Verification and Testing

The following verification steps were completed:

- Docker was installed on LINUX01.
- Docker was enabled to start on boot.
- Docker service status was checked.
- Uptime Kuma was deployed in a Docker container.
- Uptime Kuma dashboard was accessible at:

```text
http://192.168.58.133:3001
```

- DC01 showed as online in Uptime Kuma.
- DC01 was monitored using TCP port `389`.
- CLIENT01 showed as online in Uptime Kuma.
- CLIENT02 showed as online in Uptime Kuma.
- Jellyfin showed as online in Uptime Kuma.
- Jellyfin was monitored using TCP port `8096`.
- Email alerts were configured.
- VPN monitoring was not completed and was kept as a future improvement.
- Log review was not performed.
- CPU, RAM, and disk performance monitoring were not configured.

## Security and Best Practices

- Do not expose the Uptime Kuma dashboard directly to the public internet.
- Use strong authentication for the monitoring dashboard.
- Do not store usernames, passwords, API keys, tokens, SMTP passwords, or notification URLs in GitHub documentation.
- Keep monitoring dashboards on the internal network or behind a VPN.
- Use TCP port checks for important services instead of relying only on ping.
- Use retries to reduce false alarms.
- Review logs regularly as a future improvement.
- Monitor important services such as Active Directory, file sharing, VPN, backup jobs, and media services.
- Document what each monitor checks and why it matters.
- Avoid publishing sensitive internal credentials or alert configuration details.

## Commands Used

### Update Linux Package List

```bash
sudo apt update
```

### Install Docker

```bash
sudo apt install docker.io -y
```

### Enable and Start Docker

```bash
sudo systemctl enable --now docker
```

### Check Docker Status

```bash
sudo systemctl status docker
```

### Run Uptime Kuma Container

```bash
sudo docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
```

### Allow ICMP Ping on Windows Client

Run in PowerShell as Administrator:

```powershell
netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow
```

## Screenshots to Add Later

| Screenshot | What to Capture | Why It Matters | Suggested Filename |
|---|---|---|---|
| Uptime Kuma Dashboard | Main dashboard showing DC01, CLIENT01, CLIENT02, and Jellyfin online | Shows the monitoring tool working | `uptime-kuma-dashboard.png` |
| Docker Service Status | Output of `sudo systemctl status docker` | Proves Docker is running | `docker-service-status.png` |
| Uptime Kuma Container | Running Uptime Kuma Docker container | Shows the monitoring container was deployed | `uptime-kuma-container.png` |
| DC01 Monitor | TCP port monitor for port 389 | Shows AD-related service monitoring | `dc01-ldap-monitor.png` |
| CLIENT01 Monitor | Ping monitor for CLIENT01 | Shows Windows client monitoring | `client01-ping-monitor.png` |
| CLIENT02 Monitor | Ping monitor for CLIENT02 | Shows Windows client monitoring | `client02-ping-monitor.png` |
| Jellyfin Monitor | TCP port monitor for port 8096 | Shows application/service monitoring | `jellyfin-monitor.png` |
| Email Alert Settings | Email notification configuration with sensitive info hidden | Shows alerting was configured | `email-alert-settings-redacted.png` |
| Windows Firewall Rule | ICMP allow rule on Windows client | Documents troubleshooting step | `windows-icmp-firewall-rule.png` |
| Monitor Settings | Retries, heartbeat interval, and resend notification settings | Shows best-practice tuning | `monitor-settings.png` |

## What I Learned

This project helped me understand the basics of infrastructure monitoring. I learned that monitoring is not just about checking whether a computer is powered on. A server can respond to ping while an important service is still unavailable.

Using TCP port checks helped me monitor services more accurately. For example, checking DC01 on port `389` gave me a better idea of whether an Active Directory-related service was reachable. Monitoring Jellyfin on port `8096` helped confirm that the media service was available.

I also learned that monitoring results can be affected by firewall rules. A Windows client may be online but still show as down if ICMP ping is blocked. This helped me understand why sysadmins need to troubleshoot both the monitoring tool and the systems being monitored.

## Recommended Improvements

Future improvements for this project:

- Add VPN monitoring after WireGuard issues are fixed.
- Add monitoring for backup jobs.
- Add disk usage monitoring.
- Add CPU and RAM monitoring.
- Review Windows Event Viewer logs.
- Review Linux system logs.
- Review Docker logs.
- Add centralized logging.
- Add a dedicated dashboard for service health.
- Test email alert delivery by intentionally stopping a service.
- Monitor additional services such as SMB, DNS, DHCP, and SSH.
- Protect the monitoring dashboard behind VPN access only.
- Document alert testing results.
- Add screenshots with sensitive information redacted.

## Resume Bullets

- Deployed Uptime Kuma in Docker on a Linux server to monitor home lab systems and service availability.
- Configured uptime and TCP port checks for DC01, Windows clients, and a Jellyfin media server, including LDAP monitoring on port 389.
- Configured email alerts and documented future monitoring improvements such as VPN monitoring, log review, and performance monitoring.

## Interview Explanation

This project was a monitoring lab for my SysAdmin home lab. I installed Docker on a Linux server and deployed Uptime Kuma to monitor important systems like my domain controller, Windows clients, and Jellyfin media server.

I used different types of checks depending on the system. For the Windows clients, I used ping checks because I mainly wanted to know if they were online. For DC01, I used a TCP port check on port `389` because that checks an Active Directory-related service instead of only checking if the server responds to ping. For Jellyfin, I used a TCP port check on port `8096` to confirm that the service was reachable.

I also configured email alerts so I could be notified if a monitored system or service went down. This project helped me understand how sysadmins use monitoring to detect problems early and how important it is to monitor services, not just servers.
