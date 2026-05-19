# WireGuard VPN Lab - Commands

## Overview

This file contains the main commands used during the WireGuard VPN lab.

The VPN server was configured on `LINUX01`, an Ubuntu Server VM running in VMware NAT. The VPN service came up, but full client connection testing was not completed because the VPN server stayed behind VMware NAT.

> Security Reminder: Do not upload real WireGuard private keys, public IP addresses, passwords, tokens, or full client configuration files containing private keys.

## Install WireGuard

Install WireGuard on Ubuntu Server:

```bash
sudo apt install wireguard
```

If needed, install WireGuard tools:

```bash
sudo apt install wireguard-tools
```

## Generate Server Keys

Generate the WireGuard server private and public keys:

```bash
wg genkey | tee server_private.key | wg pubkey > server_public.key
```

View the generated server keys:

```bash
cat server_private.key
cat server_public.key
```

> Do not upload the real private key to GitHub.

## Create the WireGuard Server Config

Open the WireGuard server config file:

```bash
sudo vim /etc/wireguard/wg0.conf
```

Example sanitized server config:

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = YOUR_SERVER_PRIVATE_KEY

PostUp = iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE
```

## Check Linux Network Interface

Check the Linux network interface name:

```bash
ip a
```

In this lab, the interface used was:

```text
ens33
```

## Start WireGuard

Start the WireGuard interface:

```bash
sudo systemctl start wg-quick@wg0
```

Check WireGuard status:

```bash
sudo wg
```

## Restart WireGuard

Restart WireGuard after editing the config:

```bash
sudo systemctl restart wg-quick@wg0
```

Check status again:

```bash
sudo wg
```

## Check for iptables

Check whether `iptables` is installed:

```bash
which iptables
```

If it is missing, install it:

```bash
sudo apt install iptables
```

## Check for wg-quick

Check whether `wg-quick` is available:

```bash
which wg-quick
```

If it is missing, install WireGuard tools:

```bash
sudo apt install wireguard-tools
```

## Generate Client Keys

Generate the WireGuard client private and public keys:

```bash
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

View the generated client keys:

```bash
cat client_private.key
cat client_public.key
```

> Do not upload the real client private key to GitHub.

## Add Client Peer to Server Config

Open the server config:

```bash
sudo vim /etc/wireguard/wg0.conf
```

Add a sanitized client peer section:

```ini
[Peer]
PublicKey = YOUR_CLIENT_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
```

Restart WireGuard:

```bash
sudo systemctl restart wg-quick@wg0
```

Verify the peer appears:

```bash
sudo wg
```

## Create Client Config

Open the client config file:

```bash
sudo vim /etc/wireguard/client.conf
```

Example sanitized client config:

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

## Copy Client Config to User Home Directory

The `/etc/wireguard/` directory is restricted, so the client config was copied to the user home directory before using SCP.

```bash
sudo cp /etc/wireguard/client.conf /home/admin01/client.conf
sudo chmod 644 /home/admin01/client.conf
```

> Security Note: This should only be temporary because the file contains a VPN private key.

## Copy Client Config from Linux to Windows

From Windows PowerShell:

```powershell
scp admin01@VPN_SERVER_IP:/home/admin01/client.conf C:\Users\USERNAME\Desktop\client.conf
```

Replace:

```text
VPN_SERVER_IP
USERNAME
```

With the correct lab values.

## Install OpenSSH Server

Install SSH server on Ubuntu Server:

```bash
sudo apt install openssh-server
```

Enable SSH at startup:

```bash
sudo systemctl enable ssh
```

Start SSH:

```bash
sudo systemctl start ssh
```

Check SSH status:

```bash
sudo systemctl status ssh
```

## Install QR Code Tool

Install `qrencode`:

```bash
sudo apt install qrencode
```

## Display QR Code in Terminal

Display a QR code from the client config:

```bash
qrencode -t ansiutf8 < /home/admin01/client.conf
```

## Generate QR Code PNG

Generate a QR code image file:

```bash
qrencode -o /home/admin01/client.png < /home/admin01/client.conf
```

Verify the file exists:

```bash
ls /home/admin01
```

## WireGuard Windows Client Steps

These steps were completed in the Windows WireGuard app:

```text
1. Open WireGuard on Windows.
2. Click Add Tunnel.
3. Select Import from file.
4. Choose client.conf.
5. Confirm the tunnel appears in WireGuard.
```

## Useful Verification Commands

Check WireGuard status:

```bash
sudo wg
```

Check if the WireGuard service is running:

```bash
sudo systemctl status wg-quick@wg0
```

Check network interfaces:

```bash
ip a
```

Check routes:

```bash
ip route
```

Check if SSH is running:

```bash
sudo systemctl status ssh
```

Check files in the user home directory:

```bash
ls /home/admin01
```

## Troubleshooting Commands

Check if `iptables` exists:

```bash
which iptables
```

Install `iptables` if missing:

```bash
sudo apt install iptables
```

Check if `wg-quick` exists:

```bash
which wg-quick
```

Install WireGuard tools if missing:

```bash
sudo apt install wireguard-tools
```

Restart WireGuard after config changes:

```bash
sudo systemctl restart wg-quick@wg0
```

View WireGuard status:

```bash
sudo wg
```

Check SSH status before using SCP:

```bash
sudo systemctl status ssh
```

## Sanitized Config Examples

### Server Config Example

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

### Client Config Example

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

## Commands Not Confirmed

The following commands are commonly used in WireGuard VPN setups, but I do not remember confirming them in this lab. They should be tested and documented later if used.

Enable IPv4 forwarding temporarily:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Edit sysctl config for permanent IP forwarding:

```bash
sudo vim /etc/sysctl.conf
```

Possible setting:

```text
net.ipv4.ip_forward=1
```

Allow WireGuard through UFW:

```bash
sudo ufw allow 51820/udp
```

Check UFW status:

```bash
sudo ufw status
```

## Security Cleanup Commands

After copying the client config, remove temporary readable copies when they are no longer needed:

```bash
rm /home/admin01/client.conf
```

Remove the QR code image if it contains a scannable VPN config:

```bash
rm /home/admin01/client.png
```

Check that the files were removed:

```bash
ls /home/admin01
```

## Notes

- The VPN server was configured on Ubuntu Server.
- The VMware network mode stayed as NAT.
- The Linux interface used in the WireGuard NAT rule was `ens33`.
- The VPN subnet was `10.0.0.0/24`.
- The server VPN IP was `10.0.0.1`.
- The client VPN IP was `10.0.0.2`.
- The WireGuard port was `51820/UDP`.
- The VPN service came up, but full client connection and remote access testing were not completed.
- Bridged networking is recommended for a future retest because it would make the VPN server easier to reach from another network.
