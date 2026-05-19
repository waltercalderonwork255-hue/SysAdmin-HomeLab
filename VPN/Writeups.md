# WireGuard VPN Lab - Writeups

## Overview

This write-up documents my WireGuard VPN lab for my SysAdmin home lab portfolio.

The VPN server was configured on `LINUX01`, an Ubuntu Server VM running inside VMware NAT. The goal was to practice setting up a Linux-based VPN server and understand how remote access works in a lab environment.

This project is documented honestly: WireGuard was installed and configured, the VPN service came up, and the Windows client configuration was created/imported, but full VPN client connection testing and remote access testing were not completed because the VPN server stayed behind VMware NAT.

> Project Status: WireGuard VPN configured on Ubuntu Server, but full client connection testing was not completed due to VMware NAT limitations.

## Lab Summary

| Item | Details |
|---|---|
| Project | WireGuard VPN Lab |
| Server | LINUX01 |
| Server OS | Ubuntu Server |
| VPN Tool | WireGuard |
| Main Client | Windows VM |
| Network Type | VMware NAT |
| VPN Subnet | `10.0.0.0/24` |
| Server VPN IP | `10.0.0.1` |
| Client VPN IP | `10.0.0.2` |
| WireGuard Port | `51820/UDP` |
| Linux Interface | `ens33` |
| Final Status | VPN configured, but full connection testing not completed |

## Purpose of the Lab

The purpose of this lab was to learn how a VPN is configured and how it can be used for secure remote access.

In a real environment, a VPN allows an authorized user to securely connect to an internal network from another location. For this home lab, the goal was to simulate that idea by configuring WireGuard on a Linux server and preparing a Windows client to connect to it.

This project helped me practice:

- Installing WireGuard on Ubuntu Server
- Creating server and client VPN keys
- Building a WireGuard server configuration
- Adding a VPN client peer
- Creating a client configuration file
- Importing a VPN tunnel into WireGuard for Windows
- Understanding VPN routing
- Understanding NAT limitations
- Documenting troubleshooting steps clearly

## What I Built

I configured a WireGuard VPN server on `LINUX01`.

The VPN server used:

```text
VPN subnet: 10.0.0.0/24
Server VPN IP: 10.0.0.1
Client VPN IP: 10.0.0.2
WireGuard port: 51820/UDP
Linux interface: ens33
```

The WireGuard server configuration included:

- A VPN interface address
- A listening port
- A server private key
- An iptables MASQUERADE rule
- A client peer entry

The Windows VM was prepared as the VPN client. A client config was created and imported into WireGuard for Windows.

## What Worked

The following parts of the lab were completed:

- WireGuard was installed on Ubuntu Server
- Server keys were generated
- Client keys were generated
- The `wg0.conf` server config was created
- The `client.conf` client config was created
- The client peer was added to the server config
- The Linux interface was identified as `ens33`
- The WireGuard service came up
- `sudo wg` showed VPN status
- The client config was prepared for Windows
- The client tunnel was imported into WireGuard for Windows

## What Did Not Fully Work

Full VPN client connection testing was not completed.

The main reason was the network design. The VPN server stayed inside VMware NAT, which made it difficult to properly test inbound VPN connectivity from another network.

Phone testing was also not completed.

Remote access to internal lab services, such as file shares or domain services, was not confirmed through the VPN.

## Key Lesson: VPN Configuration vs Network Reachability

One of the biggest lessons from this lab was that a VPN can be configured correctly but still fail if the network path is not correct.

WireGuard can be installed, keys can be generated, and the service can come up, but the client still needs a way to reach the VPN server.

In this lab, `LINUX01` was behind VMware NAT. That allowed normal outbound internet traffic from the VM, but it made inbound VPN testing difficult.

### Outbound traffic worked

Examples of outbound traffic from the VM:

```text
LINUX01 -> Internet
LINUX01 -> Package downloads
LINUX01 -> Updates
```

### Inbound traffic was limited

The VPN server needed inbound traffic like this:

```text
VPN Client -> WireGuard Server
```

But because the server stayed behind VMware NAT, the inbound path was not easy to complete.

## NAT vs Bridged Networking Write-Up

### NAT Networking

In VMware NAT mode, the VM is placed behind a virtual NAT network.

This is useful because the VM can access the internet without being directly exposed to the physical network.

NAT worked well for:

- Installing packages
- Downloading updates
- Keeping the lab isolated
- Allowing outbound traffic from the VM

However, NAT was not ideal for hosting a VPN server because VPN servers need inbound traffic from clients.

The issue looked like this:

```text
External VPN Client
        |
        | Tries to connect to VPN server
        |
Home Router / Physical Network
        |
        | Cannot directly reach VMware NAT-only VM
        |
LINUX01 behind VMware NAT
```

### Bridged Networking

Bridged networking would be a better design for this lab in the future.

In bridged mode, the VM receives an IP address from the real physical network. This makes the VM easier to reach from other devices on the network.

A better future VPN design would be:

```text
External VPN Client
        |
        | WireGuard VPN
        |
Home Router
        |
        | UDP 51820 forwarded to VPN server
        |
LINUX01 with bridged network IP
```

This would make it easier to test:

- A successful WireGuard handshake
- Phone connection from mobile data
- Access to internal lab resources
- File server access through VPN
- Internal DNS or domain access through VPN

## WireGuard Server Write-Up

The WireGuard server was configured on Ubuntu Server.

The server configuration file was:

```text
/etc/wireguard/wg0.conf
```

The server config included:

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = YOUR_SERVER_PRIVATE_KEY

PostUp = iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE
```

The `Address` value assigned the server its VPN tunnel IP.

The `ListenPort` value made WireGuard listen on UDP port `51820`.

The `PrivateKey` value was the server private key. This value should never be uploaded to GitHub.

The `PostUp` and `PostDown` rules were used to add and remove an iptables NAT rule when the VPN interface starts and stops.

The interface used in the NAT rule was:

```text
ens33
```

## WireGuard Client Write-Up

The Windows VM was prepared as the WireGuard client.

The client config used:

```ini
[Interface]
PrivateKey = YOUR_CLIENT_PRIVATE_KEY
Address = 10.0.0.2/24
DNS = YOUR_INTERNAL_DNS_SERVER

[Peer]
PublicKey = YOUR_SERVER_PUBLIC_KEY
Endpoint = YOUR_SERVER_ENDPOINT:51820
AllowedIPs = 192.168.58.0/24
PersistentKeepalive = 25
```

The client IP was:

```text
10.0.0.2
```

The `AllowedIPs` value was intended to route internal lab traffic through the VPN tunnel.

The intended lab network was:

```text
192.168.58.0/24
```

The tunnel was imported into the Windows WireGuard client, but full connection testing was not completed.

## File Transfer Write-Up

The client configuration file was originally stored in:

```text
/etc/wireguard/client.conf
```

This directory is restricted because WireGuard configuration files contain sensitive keys.

To copy the config to the Windows VM, I copied it to the Linux user home directory first:

```bash
sudo cp /etc/wireguard/client.conf /home/admin01/client.conf
sudo chmod 644 /home/admin01/client.conf
```

Then I used SCP from Windows PowerShell:

```powershell
scp admin01@VPN_SERVER_IP:/home/admin01/client.conf C:\Users\USERNAME\Desktop\client.conf
```

This taught me that Linux permissions protect sensitive VPN files and that copying config files requires careful handling.

Important security note:

The copied `client.conf` file contains a private key. Temporary copies should be removed after use and should not be uploaded to GitHub.

## QR Code Write-Up

I also worked on creating a QR code for possible mobile WireGuard setup.

The command used to display a terminal QR code was:

```bash
qrencode -t ansiutf8 < /home/admin01/client.conf
```

A PNG QR code was also generated:

```bash
qrencode -o /home/admin01/client.png < /home/admin01/client.conf
```

The QR code image was easier to move or view than a terminal QR code.

However, phone testing was not completed.

Important security note:

A WireGuard QR code contains the client configuration, including the private key. It should not be uploaded to GitHub or shared publicly.

## Troubleshooting Write-Up

### Problem

The VPN service came up, but full VPN client connection testing was not completed.

### Cause

The VPN server stayed behind VMware NAT. Because of that, the server was not easily reachable from a separate external network.

### Why This Matters

A VPN server needs to receive inbound traffic from VPN clients. If the server is behind a NAT layer that the client cannot reach, the VPN connection cannot be fully tested.

### Fix for the Future

The recommended future fix is to move `LINUX01` to bridged networking or create a network setup where the VPN server has a reachable IP address.

A better future design would include:

- `LINUX01` using bridged networking
- Router forwarding UDP `51820` to the VPN server
- WireGuard client using the correct endpoint
- Testing from a separate network
- Confirming a successful handshake
- Testing access to internal resources

## Security Write-Up

This lab involved several sensitive files and values.

The following should not be uploaded to GitHub:

- WireGuard server private key
- WireGuard client private key
- Full working client configuration
- QR codes containing client config data
- Real public IP address
- Router login details
- Passwords
- Tokens
- Screenshots showing sensitive information

Safe placeholders should be used instead:

```text
YOUR_SERVER_PRIVATE_KEY
YOUR_CLIENT_PRIVATE_KEY
YOUR_SERVER_PUBLIC_KEY
YOUR_CLIENT_PUBLIC_KEY
YOUR_SERVER_ENDPOINT
YOUR_INTERNAL_DNS_SERVER
YOUR_PUBLIC_IP
VPN_SERVER_IP
USERNAME
```

## Verification Write-Up

The following items were verified:

| Verification Item | Status |
|---|---|
| WireGuard installed | Verified |
| Server keys generated | Verified |
| Client keys generated | Verified |
| Server config created | Verified |
| Client config created | Verified |
| Client peer added | Verified |
| Linux interface identified as `ens33` | Verified |
| WireGuard service came up | Verified |
| `sudo wg` showed VPN status | Verified |
| Windows client config import | Verified |
| Full VPN client tunnel connection | Not verified |
| Remote access to internal services | Not verified |
| Phone VPN test | Not verified |
| Bridged network retest | Not verified |

## What I Would Do Differently

If I repeated this project, I would first design the network path before configuring the VPN.

The biggest improvement would be to use bridged networking for `LINUX01` so the VPN server could receive inbound traffic more easily.

I would also:

- Confirm IP forwarding
- Confirm firewall rules
- Use separate client configs for each device
- Delete temporary client config files after copying
- Avoid keeping QR codes on disk longer than needed
- Test from a separate network
- Confirm successful WireGuard handshakes
- Test access to internal services through the VPN
- Document logs during troubleshooting

## Beginner-Friendly Explanation

A VPN lets a device connect securely to another network over an encrypted tunnel.

In this lab, I tried to create a VPN tunnel between a Windows VM and a Linux VPN server.

WireGuard handled the VPN tunnel. The Linux server had the VPN address `10.0.0.1`, and the client was planned to use `10.0.0.2`.

The VPN setup came up, but testing was limited because the Linux VPN server was hidden behind VMware NAT. That means it could reach out to the internet, but outside devices could not easily reach back into it.

The key lesson was that VPN setup requires both correct software configuration and correct network design.

## Professional Summary

This lab demonstrates hands-on exposure to Linux VPN configuration, WireGuard setup, peer configuration, and network troubleshooting.

Although the final VPN connection was not fully tested, the lab still demonstrates important junior sysadmin skills:

- Reading and editing Linux configuration files
- Installing and managing services
- Understanding VPN keys and peers
- Using SCP for file transfer
- Understanding network interface names
- Understanding NAT limitations
- Documenting incomplete testing honestly
- Identifying realistic next steps for improvement

## Resume Bullets

- Configured a WireGuard VPN server on Ubuntu Server to practice VPN setup, key generation, peer configuration, and Linux network interface configuration.
- Troubleshot VPN testing limitations caused by VMware NAT and documented why inbound VPN access requires a reachable server IP or bridged networking.
- Created beginner-friendly VPN documentation covering server configuration, client configuration, NAT behavior, verification steps, and lessons learned.

## Interview Explanation

I configured a WireGuard VPN lab on an Ubuntu Server VM named `LINUX01`. I installed WireGuard, generated the server and client keys, created the server configuration, added a Windows VM as a peer, and created a client configuration for WireGuard for Windows.

The VPN service came up, but full connection testing was limited because I kept the server inside VMware NAT. That taught me an important networking lesson: even if the VPN software is configured, the VPN server still needs to be reachable by the client. For a future retest, I would move the VPN server to bridged networking or another setup where the server has a reachable IP address.

This project helped me practice Linux administration, VPN configuration, client setup, NAT troubleshooting, and technical documentation.
