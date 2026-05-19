# Monitoring Lab Diagrams

## Overview

This folder contains diagrams for the Monitoring lab in my SysAdmin-HomeLab project.

The purpose of these diagrams is to show how **Uptime Kuma** was deployed and how it monitored key systems in the lab.

This monitoring lab used:

- Uptime Kuma
- Docker
- LINUX01 as the monitoring server
- DC01, CLIENT01, CLIENT02, and Jellyfin as monitored systems
- Email alerts for outage notifications

> Security note: Do not include passwords, SMTP credentials, tokens, or private notification URLs in diagrams.

---

## Diagram 1: Monitoring Lab Topology

This diagram shows the main monitoring layout.

```mermaid
flowchart TD
    AdminPC[Admin Workstation / Browser] --> Kuma[Uptime Kuma Dashboard<br>http://192.168.58.133:3001]

    Kuma --> Docker[Docker<br>Running on LINUX01]
    Docker --> Linux01[LINUX01<br>192.168.58.133]

    Kuma --> DC01[DC01<br>Domain Controller<br>TCP Port 389]
    Kuma --> Client01[CLIENT01<br>Windows Client<br>Ping Check]
    Kuma --> Client02[CLIENT02<br>Windows Client<br>Ping Check]
    Kuma --> Jellyfin[Jellyfin<br>Media Server<br>TCP Port 8096]

    Kuma --> Email[Email Alerts<br>Configured]
```

---

## Diagram 2: Monitoring Server Design

This diagram shows how Uptime Kuma was hosted on LINUX01 using Docker.

```mermaid
flowchart TD
    Linux01[LINUX01<br>Ubuntu Server<br>192.168.58.133]

    Linux01 --> DockerService[Docker Service<br>Enabled on Boot]
    DockerService --> KumaContainer[Uptime Kuma Container<br>Container Name: uptime-kuma]
    KumaContainer --> Port3001[Web Dashboard<br>Port 3001]
    KumaContainer --> Volume[Docker Volume<br>uptime-kuma:/app/data]

    Browser[Admin Browser] --> Port3001
```

---

## Diagram 3: Uptime Kuma Monitor Types

This diagram shows the types of checks used for each monitored system.

```mermaid
flowchart LR
    Kuma[Uptime Kuma]

    Kuma --> DC01Check[TCP Port Check<br>DC01<br>Port 389]
    Kuma --> Client01Check[Ping Check<br>CLIENT01]
    Kuma --> Client02Check[Ping Check<br>CLIENT02]
    Kuma --> JellyfinCheck[TCP Port Check<br>Jellyfin<br>Port 8096]

    DC01Check --> ADService[LDAP / AD-related Service]
    Client01Check --> ClientOnline1[Client Online Status]
    Client02Check --> ClientOnline2[Client Online Status]
    JellyfinCheck --> JellyfinService[Jellyfin Service Availability]
```

---

## Diagram 4: Alert Flow

This diagram shows the basic alerting flow used in the lab.

```mermaid
flowchart TD
    Monitor[Uptime Kuma Monitor] --> Check{Is the system or service online?}

    Check -->|Yes| Green[Status: Online]
    Check -->|No| Retry[Retry Check<br>Retries: 3]

    Retry --> ConfirmDown{Still Down?}

    ConfirmDown -->|No| Green
    ConfirmDown -->|Yes| Alert[Send Email Alert]

    Alert --> Admin[Admin Receives Notification]
```

---

## Diagram 5: Jellyfin Monitoring Troubleshooting

This diagram shows the troubleshooting path for the Jellyfin monitor.

```mermaid
flowchart TD
    Start[Jellyfin Monitor Created] --> Keyword[HTTP Keyword Check<br>Keyword: Jellyfin]

    Keyword --> Red[Monitor Stayed Red]
    Red --> Review[Review Monitor Type]
    Review --> Change[Changed to TCP Port Check]

    Change --> TCP[TCP Port Check<br>Host: 192.168.58.133<br>Port: 8096]
    TCP --> Green[Jellyfin Showed Online]
```

---

## Diagram 6: Windows Client Ping Monitoring

This diagram shows why Windows Firewall may affect ping monitoring.

```mermaid
flowchart TD
    Kuma[Uptime Kuma] --> Ping[Ping CLIENT01 / CLIENT02]

    Ping --> Firewall{Windows Firewall Allows ICMP?}

    Firewall -->|Yes| Online[Client Shows Online]
    Firewall -->|No| Down[Client May Show Down]

    Down --> Rule[Add ICMP Allow Rule]
    Rule --> Online
```

---

## Diagram 7: Future VPN Monitoring Idea

VPN monitoring was not completed in this project. This diagram shows a possible future design.

```mermaid
flowchart TD
    Kuma[Uptime Kuma] --> VPNCheck[Future VPN Monitor]

    VPNCheck --> Option1[Option 1:<br>Ping Internal VPN IP]
    VPNCheck --> Option2[Option 2:<br>Push Monitor]

    Option1 --> InternalIP[Example Internal VPN IP<br>10.0.0.1]
    Option2 --> Cron[Cron Job on VPN Server<br>Sends Heartbeat to Kuma]

    InternalIP --> VPNStatus[VPN Availability Status]
    Cron --> VPNStatus
```

---

## Diagram Notes

### What Was Actually Built

The following monitoring setup was completed:

- Uptime Kuma was deployed in Docker.
- Docker ran on LINUX01.
- Uptime Kuma dashboard was accessed on port `3001`.
- DC01 was monitored using TCP port `389`.
- CLIENT01 was monitored using ping.
- CLIENT02 was monitored using ping.
- Jellyfin was monitored using TCP port `8096`.
- Email alerts were configured.

### What Was Not Completed

The following items were not completed in this lab:

- VPN monitoring
- CPU monitoring
- RAM monitoring
- Disk usage monitoring
- Log monitoring
- Centralized logging

These can be added as future improvements.

---

## Suggested Diagram Files

If I export diagrams as images later, I can save them with these filenames:

| Diagram | Suggested Filename |
|---|---|
| Monitoring Lab Topology | `monitoring-lab-topology.png` |
| Monitoring Server Design | `uptime-kuma-docker-design.png` |
| Uptime Kuma Monitor Types | `uptime-kuma-monitor-types.png` |
| Alert Flow | `email-alert-flow.png` |
| Jellyfin Troubleshooting | `jellyfin-monitor-troubleshooting.png` |
| Windows Client Ping Monitoring | `windows-client-ping-monitoring.png` |
| Future VPN Monitoring Idea | `future-vpn-monitoring.png` |

---

## README Embed Example

To include one of these diagrams in the main Monitoring README later, use this format:

```markdown
![Monitoring Lab Topology](./Diagrams/monitoring-lab-topology.png)
```

Or keep the Mermaid diagram directly in the README if GitHub renders it correctly.

---

## Security Reminder

Before uploading diagrams to GitHub, make sure they do not show:

- Passwords
- Email passwords
- SMTP app passwords
- API tokens
- Private notification URLs
- Personal email addresses
- Public IP addresses
- Private keys
- Recovery codes
