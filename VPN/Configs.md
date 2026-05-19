# WireGuard VPN Lab - Configs

## Overview

This file documents the configuration examples used in the WireGuard VPN lab.

The VPN server was configured on `LINUX01`, an Ubuntu Server VM running inside VMware NAT. The WireGuard service came up, but full client connection and remote access testing were not completed because the server stayed behind VMware NAT.

> Security Reminder: Do not upload real private keys, public IP addresses, passwords, tokens, router details, or full client configuration files containing private keys.

## Lab Configuration Summary

| Setting | Value |
|---|---|
| VPN Server | LINUX01 |
| VPN Server OS | Ubuntu Server |
| VPN Client | Windows VM |
| VPN Tool | WireGuard |
| Network Type | VMware NAT |
| VPN Interface | `wg0` |
| Linux Network Interface | `ens33` |
| WireGuard Port | `51820/UDP` |
| VPN Subnet | `10.0.0.0/24` |
| VPN Server Address | `10.0.0.1/24` |
| VPN Client Address | `10.0.0.2/24` |
| Internal Lab Network | `192.168.58.0/24` |
| Final Status | VPN service came up, but full client connection testing was not completed |

## Important Security Notes

The configuration examples below are sanitized.

Do not upload real values for:

- Server private key
- Client private key
- Public IP address
- Router information
- Passwords
- Tokens
- Full client config files with working private keys
- QR codes that contain real VPN client configs

Use placeholders instead:

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

## WireGuard Server Configuration

File location:

```text
/etc/wireguard/wg0.conf
```

Sanitized server configuration:

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = YOUR_SERVER_PRIVATE_KEY

PostUp = iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE

[Peer]
PublicKey = YOUR_CLIENT_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
```

## Server Configuration Explanation

### `[Interface]`

The `[Interface]` section defines the WireGuard VPN server settings.

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = YOUR_SERVER_PRIVATE_KEY
```

| Setting | Purpose |
|---|---|
| `Address = 10.0.0.1/24` | Assigns the VPN server its VPN tunnel IP |
| `ListenPort = 51820` | Makes WireGuard listen on UDP port 51820 |
| `PrivateKey = YOUR_SERVER_PRIVATE_KEY` | Server private key used by WireGuard |

### `PostUp` and `PostDown`

```ini
PostUp = iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE
```

| Setting | Purpose |
|---|---|
| `PostUp` | Runs when the VPN interface starts |
| `PostDown` | Runs when the VPN interface stops |
| `iptables -t nat` | Uses the NAT table |
| `POSTROUTING` | Applies the rule after routing decisions are made |
| `-o ens33` | Sends traffic out through the `ens33` interface |
| `MASQUERADE` | Allows VPN client traffic to be NATed through the server |

In this lab, the Linux network interface was confirmed as:

```text
ens33
```

### `[Peer]`

The `[Peer]` section defines an allowed VPN client.

```ini
[Peer]
PublicKey = YOUR_CLIENT_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
```

| Setting | Purpose |
|---|---|
| `PublicKey` | Public key of the VPN client |
| `AllowedIPs = 10.0.0.2/32` | Allows this client to use VPN IP `10.0.0.2` |

## WireGuard Client Configuration

File created on the Linux server:

```text
/etc/wireguard/client.conf
```

Temporary copy location used for SCP:

```text
/home/admin01/client.conf
```

Sanitized client configuration:

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

## Client Configuration Explanation

### `[Interface]`

```ini
[Interface]
PrivateKey = YOUR_CLIENT_PRIVATE_KEY
Address = 10.0.0.2/24
DNS = YOUR_INTERNAL_DNS_SERVER
```

| Setting | Purpose |
|---|---|
| `PrivateKey` | Client private key used by WireGuard |
| `Address = 10.0.0.2/24` | Assigns the VPN client its tunnel IP |
| `DNS` | Points the VPN client to the internal DNS server |

### `[Peer]`

```ini
[Peer]
PublicKey = YOUR_SERVER_PUBLIC_KEY
Endpoint = YOUR_SERVER_ENDPOINT:51820
AllowedIPs = 192.168.58.0/24
PersistentKeepalive = 25
```

| Setting | Purpose |
|---|---|
| `PublicKey` | Public key of the WireGuard server |
| `Endpoint` | Address and port of the VPN server |
| `AllowedIPs = 192.168.58.0/24` | Routes traffic for the lab network through the VPN |
| `PersistentKeepalive = 25` | Helps keep NAT sessions alive |

## Intended VPN Addressing

| Device | VPN Address |
|---|---|
| WireGuard Server | `10.0.0.1/24` |
| Windows VPN Client | `10.0.0.2/24` |

## Intended Lab Network Access

The intended client route was:

```ini
AllowedIPs = 192.168.58.0/24
```

This means the VPN client was intended to send traffic for the internal lab network through the VPN tunnel.

Examples of intended access:

| Resource | Status |
|---|---|
| File server access | Not fully tested |
| Domain/DNS access | Not fully tested |
| Internal lab services | Not fully tested |
| Ping across VPN | Not fully confirmed |

## VMware NAT Configuration Context

The VPN server stayed inside VMware NAT.

This allowed the VM to reach the internet for outbound traffic, such as package installation.

However, it made inbound VPN testing difficult because external devices could not directly reach the VPN server VM.

### NAT Design Used

```text
Windows VM / Lab Client
        |
        | VMware NAT Network
        |
LINUX01 Ubuntu Server
        |
        | Outbound internet works
        |
Internet
```

### Inbound VPN Limitation

The issue with inbound testing was:

```text
External VPN Client
        |
        | Attempts to connect to VPN endpoint
        |
Home Router
        |
        | Cannot directly forward to VMware NAT-only VM
        |
LINUX01 inside VMware NAT
```

Because of this, the project is documented as a VPN configuration and troubleshooting lab, not a fully completed remote access deployment.

## Recommended Future Bridged Configuration

A better future design would use bridged networking.

```text
External VPN Client
        |
        | WireGuard VPN
        |
Home Router
        |
        | UDP 51820 port forward
        |
LINUX01 with bridged network IP
```

With bridged networking, `LINUX01` would receive an IP address directly on the physical network. This would make it easier for the router to forward UDP port `51820` to the VPN server.

## Port and Protocol Configuration

| Service | Port | Protocol | Purpose |
|---|---:|---|---|
| WireGuard | `51820` | UDP | VPN tunnel traffic |
| SSH | `22` | TCP | SCP file transfer and remote Linux access |

## File Locations

| File | Purpose | Safe to Upload? |
|---|---|---|
| `/etc/wireguard/wg0.conf` | WireGuard server config | Only sanitized version |
| `/etc/wireguard/client.conf` | WireGuard client config | No, contains private key |
| `/home/admin01/client.conf` | Temporary copy for SCP | No, contains private key |
| `/home/admin01/client.png` | QR code for client config | No, contains private key inside QR code |
| `server_private.key` | Server private key | No |
| `server_public.key` | Server public key | Yes, but sanitize if unsure |
| `client_private.key` | Client private key | No |
| `client_public.key` | Client public key | Yes, but sanitize if unsure |

## SCP Transfer Configuration

The client config could not be copied directly from:

```text
/etc/wireguard/client.conf
```

Because `/etc/wireguard/` is restricted.

The file was temporarily copied to:

```text
/home/admin01/client.conf
```

Using:

```bash
sudo cp /etc/wireguard/client.conf /home/admin01/client.conf
sudo chmod 644 /home/admin01/client.conf
```

Then copied from Windows using SCP:

```powershell
scp admin01@VPN_SERVER_IP:/home/admin01/client.conf C:\Users\USERNAME\Desktop\client.conf
```

> Security Note: The temporary readable copy should be deleted after transfer because it contains the client private key.

## QR Code Configuration

A QR code was generated from the client config for possible phone testing.

Terminal QR code:

```bash
qrencode -t ansiutf8 < /home/admin01/client.conf
```

PNG QR code:

```bash
qrencode -o /home/admin01/client.png < /home/admin01/client.conf
```

The PNG file location was:

```text
/home/admin01/client.png
```

> Security Note: The QR code contains the VPN client configuration, including the private key. Do not upload the QR code to GitHub.

## Optional Configurations Not Confirmed

The following settings are common in WireGuard VPN setups, but I do not remember confirming them in this lab. They should be tested and documented later if used.

### IPv4 Forwarding

Temporary IPv4 forwarding:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Permanent IPv4 forwarding file:

```text
/etc/sysctl.conf
```

Possible setting:

```text
net.ipv4.ip_forward=1
```

### UFW Firewall Rule

Allow WireGuard traffic:

```bash
sudo ufw allow 51820/udp
```

Check firewall status:

```bash
sudo ufw status
```

## Sanitized Full Example: Server Config

```ini
# /etc/wireguard/wg0.conf

[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = YOUR_SERVER_PRIVATE_KEY

PostUp = iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE

[Peer]
PublicKey = YOUR_CLIENT_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
```

## Sanitized Full Example: Client Config

```ini
# client.conf

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

## Final Configuration Status

| Configuration Item | Status |
|---|---|
| WireGuard installed | Completed |
| Server key pair generated | Completed |
| Client key pair generated | Completed |
| Server config created | Completed |
| Client config created | Completed |
| Client peer added to server | Completed |
| Linux interface identified as `ens33` | Completed |
| iptables MASQUERADE rule included | Completed/attempted |
| Windows client config import | Completed |
| VPN service came up | Completed |
| Full VPN client connection | Not completed |
| Remote access to internal services | Not completed |
| Bridged network retest | Not completed |

## Cleanup Reminder

Remove temporary client config files and QR codes when finished:

```bash
rm /home/admin01/client.conf
rm /home/admin01/client.png
```

Check the files are gone:

```bash
ls /home/admin01
```

## Notes

- This lab used Ubuntu Server on `LINUX01`.
- The VPN server stayed on VMware NAT.
- The WireGuard interface was `wg0`.
- The Linux network interface was `ens33`.
- The VPN subnet was `10.0.0.0/24`.
- The server VPN IP was `10.0.0.1`.
- The client VPN IP was `10.0.0.2`.
- The WireGuard port was `51820/UDP`.
- The Windows VM was the main VPN client.
- Phone testing was not completed.
- Full remote access testing was not completed.
- Bridged networking is the recommended future improvement.
