# Network Diagram Notes - Active Directory Lab

## Overview

This file documents the basic network layout for my Active Directory home lab.

The lab was built using VMware NAT networking. The main server, `DC01`, was configured as the domain controller for the `corp.local` domain. Two Windows 11 Pro clients, `CLIENT01` and `CLIENT02`, were joined to the domain.

---

## Lab Network

```text
Network Type: VMware NAT
Network: 192.168.58.0/24
Default Gateway: 192.168.58.2
Domain: corp.local
```

---

## Main Diagram

```text
                    VMware NAT Network
                     192.168.58.0/24
                            |
                            |
              Default Gateway: 192.168.58.2
                            |
                            |
        ------------------------------------------------
        |                       |                      |
      DC01                  CLIENT01               CLIENT02
192.168.58.10             Windows 11 Pro          Windows 11 Pro
Windows Server 2022       Domain Joined           Domain Joined
AD DS / DNS / DHCP Role    corp.local              corp.local
```

---

## Server Details

### DC01

```text
Hostname: DC01
Operating System: Windows Server 2022 Evaluation
IP Address: 192.168.58.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.58.2
Preferred DNS: 192.168.58.10
Domain: corp.local
Network Type: VMware NAT
```

### Roles Installed on DC01

```text
Active Directory Domain Services
DNS Server
DHCP Server role installed for practice
```

### DHCP Note

```text
DHCP role was installed, but no DHCP scope was configured.
```

---

## Client Details

### CLIENT01

```text
Hostname: CLIENT01
Operating System: Windows 11 Pro
Domain Joined: Yes
Domain: corp.local
DNS Server: 192.168.58.10
Network Type: VMware NAT
```

### CLIENT02

```text
Hostname: CLIENT02
Operating System: Windows 11 Pro
Domain Joined: Yes
Domain: corp.local
DNS Server: 192.168.58.10
Network Type: VMware NAT
```

---

## Active Directory Structure Diagram

```text
corp.local
│
├── _Admins
│
├── _Servers
│
├── _Workstations
│
├── _Users
│   ├── IT
│   ├── HR
│   ├── Sales
│   └── Executives
│
├── _ServiceAccounts
│
└── _Groups
```

---

## Security Group Layout

```text
_Groups
│
├── GRP_Domain_Admins
├── GRP_Server_Admins
├── GRP_Workstation_Admins
└── GRP_Helpdesk_Admins
```

---

## Group Policy Link Layout

```text
corp.local
│
├── DomainPasswordPolicy
│
├── _Users
│   ├── Lock Screen Policy
│   └── Desktop Wallpaper Policy
│
├── _Workstations
│   ├── Disable USB Storage Policy
│   └── Workstation Local Admins Policy
│
└── _Servers
    └── Server Local Admins Policy
```

---

## DNS Flow

```text
CLIENT01 / CLIENT02
        |
        | DNS query for corp.local
        v
      DC01
192.168.58.10
        |
        | Responds with domain controller / domain information
        v
Client can locate Active Directory services
```

---

## Domain Join Flow

```text
CLIENT01 / CLIENT02
        |
        | Uses DNS Server: 192.168.58.10
        v
      DC01
        |
        | Finds corp.local domain
        v
Client joins domain successfully
```

---

## Important Network Notes

- `DC01` uses a static IP address.
- `DC01` uses itself as the preferred DNS server.
- Domain-joined clients use `192.168.58.10` as DNS.
- The VMware NAT gateway is `192.168.58.2`.
- The NAT gateway should not be used as DNS for domain-joined clients.
- DHCP role was installed for practice, but no DHCP scope was configured.
- `CLIENT01` and `CLIENT02` both joined the `corp.local` domain successfully.

---

## What This Diagram Shows

This diagram shows a small Windows domain environment with one domain controller and two domain-joined Windows clients. It demonstrates how clients rely on the domain controller for DNS and Active Directory services.

The diagram also shows the basic OU, group, and Group Policy structure used in the lab.
