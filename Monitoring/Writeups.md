# Monitoring Lab Writeups

## Overview

This folder contains the written documentation for the Monitoring lab in my SysAdmin-HomeLab project.

The goal of this writeup is to explain what I built, why I built it, what I tested, what issues I ran into, and what I learned from the project.

This project used **Uptime Kuma** running in **Docker** on **LINUX01** to monitor important lab systems and services.

---

## Project Summary

In this project, I deployed Uptime Kuma as a monitoring dashboard for my home lab. Uptime Kuma was installed using Docker on LINUX01, which had the IP address `192.168.58.133`.

I monitored the following systems:

- DC01
- CLIENT01
- CLIENT02
- Jellyfin

The monitoring setup focused on **uptime and service availability**, not full performance monitoring. I did not configure CPU, RAM, disk, or log monitoring in this project.

Email alerts were also configured so that Uptime Kuma could notify me when a monitored system or service goes down.

---

## Why I Built This

Monitoring is an important part of system administration. A sysadmin needs to know when servers, clients, or services stop working.

Before this project, I could manually check whether a server was online, but that is not efficient. A monitoring dashboard gives a central place to see what is online and what is down.

This project helped me practice:

- Deploying a monitoring tool
- Running an application in Docker
- Checking server availability
- Checking service availability
- Configuring email alerts
- Troubleshooting failed monitor checks
- Documenting monitoring results

---

## Lab Environment

| Component | Details |
|---|---|
| Monitoring Server | LINUX01 |
| Monitoring Server IP | 192.168.58.133 |
| Monitoring Tool | Uptime Kuma |
| Deployment Method | Docker |
| Uptime Kuma Port | 3001 |
| Dashboard URL | `http://192.168.58.133:3001` |
| Monitored Systems | DC01, CLIENT01, CLIENT02, Jellyfin |
| Alerts | Email alerts configured |
| Logs Reviewed | None |
| Performance Monitoring | Not configured |
| VPN Monitoring | Future improvement |

---

## What I Monitored

### DC01

DC01 was monitored using a TCP port check on port `389`.

Port `389` is used for LDAP, which is related to Active Directory. This was useful because it checked more than just whether the server was powered on. It helped confirm that an Active Directory-related service was reachable.

| Item | Details |
|---|---|
| System | DC01 |
| Monitor Type | TCP Port |
| Port | 389 |
| Status | Online |
| Purpose | Check LDAP / Active Directory-related service availability |

---

### CLIENT01

CLIENT01 was monitored using a ping check.

Since CLIENT01 is a Windows client and not a major server in this lab, a ping check was enough to confirm that it was online and reachable.

| Item | Details |
|---|---|
| System | CLIENT01 |
| Monitor Type | Ping |
| Status | Online |
| Purpose | Check if the Windows client was reachable |

---

### CLIENT02

CLIENT02 was monitored using a ping check.

Like CLIENT01, this was used to confirm basic network availability.

| Item | Details |
|---|---|
| System | CLIENT02 |
| Monitor Type | Ping |
| Status | Online |
| Purpose | Check if the Windows client was reachable |

---

### Jellyfin

Jellyfin was monitored using a TCP port check on port `8096`.

I originally tried an HTTP keyword check, but it did not work reliably. I changed the monitor to a TCP port check, which successfully showed Jellyfin as online.

| Item | Details |
|---|---|
| System | Jellyfin |
| Monitor Type | TCP Port |
| Host | 192.168.58.133 |
| Port | 8096 |
| Status | Online |
| Purpose | Check if the Jellyfin service was reachable |

---

## What I Did Not Monitor

This project was focused on basic uptime and service checks. I did not configure full performance or log monitoring.

Not configured in this project:

- CPU monitoring
- RAM monitoring
- Disk monitoring
- Windows Event Viewer log monitoring
- Linux log monitoring
- Centralized logging
- VPN monitoring

VPN monitoring was kept as a future improvement because I was still having issues with the VPN setup.

---

## Writeup: Installing Docker

### What I Did

I installed Docker on LINUX01 using the Ubuntu package repository.

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker
```

### Why I Did It

Docker allowed me to run Uptime Kuma as a container. This made the setup easier because I did not need to manually install all of Uptime Kuma's application dependencies.

Using Docker also made it easier to restart and manage the monitoring tool.

### Verification

I checked Docker with:

```bash
sudo systemctl status docker
```

This confirmed that Docker was running.

### Lesson Learned

Docker is useful for deploying tools quickly in a lab environment. It also keeps applications separated from the main operating system.

---

## Writeup: Deploying Uptime Kuma

### What I Did

I deployed Uptime Kuma using a Docker container.

```bash
sudo docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
```

### Why I Did It

Uptime Kuma provides a simple dashboard for monitoring services and hosts. It is beginner-friendly and useful for learning the basics of monitoring.

### Important Configuration Details

| Setting | Value |
|---|---|
| Container Name | uptime-kuma |
| Image | louislam/uptime-kuma:1 |
| Host Port | 3001 |
| Container Port | 3001 |
| Docker Volume | uptime-kuma |
| Container Data Path | /app/data |
| Restart Policy | always |

### Verification

After the container started, I opened the dashboard in a browser:

```text
http://192.168.58.133:3001
```

The dashboard loaded successfully.

### Lesson Learned

Using a Docker volume is important because it keeps the Uptime Kuma data persistent. Without persistent storage, monitoring configuration could be lost if the container is removed.

---

## Writeup: Monitoring DC01

### What I Did

I added DC01 as a monitor in Uptime Kuma.

The monitor used a TCP port check on port `389`.

### Why I Did It

DC01 is an important system because it is the domain controller. If Active Directory-related services are unavailable, domain users and computers may be affected.

A TCP port check is better than only using ping because ping only confirms that the server responds on the network. It does not confirm that a specific service is reachable.

### Verification

Uptime Kuma showed DC01 as online.

### Lesson Learned

Service-level monitoring is more useful than basic host monitoring. A server can be online but still have a failed or unreachable service.

---

## Writeup: Monitoring Windows Clients

### What I Did

I added CLIENT01 and CLIENT02 to Uptime Kuma using ping checks.

### Why I Did It

CLIENT01 and CLIENT02 are Windows clients in the lab. They do not host major services, so the main thing I wanted to know was whether they were online and reachable.

### Troubleshooting Note

If Windows clients do not respond to ping, Windows Firewall may be blocking ICMP traffic.

The firewall rule used was:

```powershell
netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow
```

### Verification

Uptime Kuma showed CLIENT01 and CLIENT02 as online.

### Lesson Learned

A system showing as down in a monitoring tool does not always mean the system is actually offline. It could be caused by firewall settings blocking the monitoring traffic.

---

## Writeup: Monitoring Jellyfin

### What I Did

I added Jellyfin to Uptime Kuma.

The first idea was to use an HTTP keyword check:

| Setting | Value |
|---|---|
| Monitor Type | HTTP(s) - Keyword |
| URL | `http://192.168.58.133:8096` |
| Keyword | Jellyfin |

This did not work reliably, so I changed the monitor to a TCP port check:

| Setting | Value |
|---|---|
| Monitor Type | TCP Port |
| Hostname | 192.168.58.133 |
| Port | 8096 |

### Why I Did It

Jellyfin is a web-based service running in the lab. Monitoring port `8096` helped confirm that the Jellyfin service was reachable.

### Verification

Uptime Kuma showed Jellyfin as online after changing the monitor to a TCP port check.

### Lesson Learned

HTTP keyword checks can be useful, but they can also fail if the page content does not match exactly or if the application behaves differently than expected.

A TCP port check is simpler and works well for confirming basic service availability.

---

## Writeup: Configuring Email Alerts

### What I Did

I configured email alerts in Uptime Kuma.

### Why I Did It

A monitoring dashboard is useful, but alerts make it more practical. Instead of manually checking the dashboard, email alerts can notify me when something goes down.

### Security Note

I did not document the exact SMTP credentials, email passwords, or app passwords because those are sensitive.

Safe documentation format:

```text
Notification Type: Email
SMTP Server: REDACTED
SMTP Username: REDACTED
SMTP Password/App Password: REDACTED
Recipient Email: REDACTED
Status: Configured
```

### Verification

Email alerts were configured in Uptime Kuma.

### Lesson Learned

Alerting is an important part of monitoring. A monitoring system is more useful when it can notify the admin about issues automatically.

---

## Troubleshooting Writeup

### Issue 1: Jellyfin Monitor Stayed Red

**Problem:**  
The Jellyfin monitor stayed red when using an HTTP keyword check.

**Cause:**  
The HTTP keyword check may have been too specific or unreliable for this setup.

**Fix:**  
I changed the monitor type to TCP Port and used port `8096`.

**Verification:**  
Uptime Kuma showed Jellyfin as online.

**Lesson learned:**  
If a web keyword check does not work, use a simpler TCP port check to confirm the service is reachable.

---

### Issue 2: Windows Clients May Show as Down

**Problem:**  
A Windows client may show as down even when it is powered on.

**Cause:**  
Windows Firewall may block ICMP ping traffic.

**Fix:**  
Allow ICMP echo requests through Windows Firewall.

```powershell
netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow
```

**Verification:**  
The Windows clients showed as online in Uptime Kuma.

**Lesson learned:**  
Monitoring results can be affected by firewall rules. A failed ping is not always proof that a system is offline.

---

## Monitoring Best Practices Used

The following monitoring settings were used or documented:

| Setting | Value |
|---|---|
| Heartbeat Interval | 60 seconds |
| Retries | 3 |
| Resend Notification | 0 / Disabled |

### Why These Settings Matter

A 60-second heartbeat interval is reasonable for a home lab. It checks often enough to notice problems without being excessive.

Retries help prevent false alarms. A device may miss one check because of a temporary network issue, but that does not always mean the system is truly down.

Disabling repeated notifications helps prevent alert fatigue.

---

## Security Notes

For security, I avoided documenting sensitive information.

Do not upload the following to GitHub:

- Uptime Kuma passwords
- Email passwords
- SMTP app passwords
- API tokens
- Notification URLs
- Private keys
- Recovery codes
- Personal email addresses
- Any real credentials

Monitoring dashboards should also not be exposed directly to the public internet. They should stay on the internal network or be protected behind secure remote access such as a VPN.

---

## Final Verification Summary

| Item | Result |
|---|---|
| Docker installed | Completed |
| Docker enabled on boot | Completed |
| Uptime Kuma deployed | Completed |
| Dashboard accessible | Completed |
| DC01 monitor configured | Completed |
| DC01 showed online | Completed |
| CLIENT01 monitor configured | Completed |
| CLIENT01 showed online | Completed |
| CLIENT02 monitor configured | Completed |
| CLIENT02 showed online | Completed |
| Jellyfin monitor configured | Completed |
| Jellyfin showed online | Completed |
| Email alerts configured | Completed |
| VPN monitoring | Future improvement |
| Log monitoring | Not completed |
| CPU/RAM/disk monitoring | Not completed |

---

## What I Learned

This project helped me understand the basics of monitoring in a sysadmin environment. I learned that monitoring should check more than whether a system is powered on. It is also important to monitor whether services are actually reachable.

I learned how to deploy Uptime Kuma in Docker, configure monitors, troubleshoot failed checks, and set up email alerts. I also learned that firewall rules can affect monitoring results and that TCP port checks can sometimes be more useful than simple ping checks.

This was a beginner-friendly monitoring project, but it gave me practical experience with concepts that are important in real IT environments.

---

## Future Improvements

In the future, I would improve this monitoring project by adding:

- VPN monitoring after WireGuard issues are fixed
- Disk usage monitoring
- CPU monitoring
- RAM monitoring
- Backup job monitoring
- Windows Event Viewer log review
- Linux log review
- Docker log review
- Centralized logging
- Alert testing documentation
- Screenshots with sensitive information redacted
- Monitoring for additional services such as DNS, DHCP, SMB, and SSH

---

## Interview Explanation

If asked about this project in an interview, I would explain it like this:

> I built a monitoring lab using Uptime Kuma running in Docker on a Linux server. I used it to monitor my domain controller, two Windows clients, and a Jellyfin media server. For DC01, I used a TCP port check on port 389 to monitor LDAP-related availability. For the Windows clients, I used ping checks. For Jellyfin, I used a TCP port check on port 8096 after an HTTP keyword check was not reliable.
>
> I also configured email alerts so the monitoring system could notify me when something went down. This project helped me learn the difference between basic uptime monitoring and service-level monitoring, and it also taught me how firewall rules can affect monitoring results.

---

## Resume Bullets

- Deployed Uptime Kuma in Docker on a Linux server to monitor home lab systems and service availability.
- Configured uptime and TCP port checks for DC01, Windows clients, and a Jellyfin media server, including LDAP monitoring on port 389.
- Configured email alerts and documented troubleshooting steps, security notes, and future monitoring improvements.
