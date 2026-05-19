# WireGuard VPN Lab

## Overview

This project documents my WireGuard VPN lab for my SysAdmin home lab. The VPN server was configured on `LINUX01`, an Ubuntu Server VM running inside VMware NAT.

The purpose of this project was to practice setting up a Linux-based VPN server, creating WireGuard server and client configurations, generating VPN keys, adding a VPN peer, and understanding how VPN traffic works in a lab environment.

This project is documented honestly as a VPN configuration and troubleshooting lab. The WireGuard service came up, but full client connection and remote access testing were not completed because the server stayed behind VMware NAT.

> Project Status: WireGuard VPN configured successfully on Ubuntu Server, but full client connection and remote access testing were not completed due to VMware NAT limitations.

## Goal

The goal of this project was to:

- Install and configure WireGuard on Ubuntu Server
- Create a VPN subnet for lab access
- Generate server and client VPN keys
- Create a WireGuard server configuration
- Create a WireGuard client configuration
- Add a Windows VM as a VPN peer
- Start and verify the WireGuard service
- Understand how NAT, bridged networking, firewall rules, and port forwarding affect VPN connectivity
- Document troubleshooting steps and lessons learned

## Skills Demonstrated

- WireGuard VPN installation and configuration
- Ubuntu Server administration
- Linux command-line usage
- VPN key generation
- WireGuard peer configuration
- VPN subnet planning
- Basic firewall and NAT concepts
- iptables MASQUERADE configuration
- VMware NAT troubleshooting
- Remote access troubleshooting
- SCP file transfer between Linux and Windows
- QR code generation for VPN client setup
- Technical documentation for a sysadmin portfolio

## Environment

| Component | Details |
|---|---|
| VPN Server | LINUX01 |
| VPN Server OS | Ubuntu Server |
| VPN Client | Windows VM |
| VPN Tool | WireGuard |
| Network Type | VMware NAT |
| VPN Subnet | `10.0.0.0/24` |
| VPN Server IP | `10.0.0.1` |
| VPN Client IP | `10.0.0.2` |
| Linux Interface | `ens33` |
| Listen Port | `51820/UDP` |
| Firewall Rules | iptables NAT/MASQUERADE rule was configured or attempted |
| Final Status | VPN service came up, but remote client testing was not fully completed |

## VPN Design

The VPN server was configured on `LINUX01`, an Ubuntu Server VM running inside VMware NAT.

The intended design was to use WireGuard to allow a Windows VM client to connect to the VPN server and securely access lab resources on the internal network.

The VPN network used the subnet `10.0.0.0/24`:

- VPN server: `10.0.0.1`
- VPN client: `10.0.0.2`
- WireGuard port: `51820/UDP`
- Linux network interface: `ens33`

The server configuration included an iptables MASQUERADE rule using the `ens33` interface.

Example:

```bash
PostUp = iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE
```

The intended traffic flow was:

```text
Windows VPN Client
        |
        | WireGuard VPN Tunnel
        |
LINUX01 Ubuntu Server
        |
        | VMware NAT / Lab Network
        |
Internal Lab Resources
```

The VPN service came up, but full remote access testing was not completed. Because the VPN server stayed behind VMware NAT and the lab setup only allowed one NIC per VM, I could not fully test outside VPN access from a separate network.

This project is documented as a configuration and troubleshooting lab rather than a fully completed production-style VPN deployment.

### NAT vs Bridged Networking Lesson

The original setup used VMware NAT.

NAT allowed the Linux VM to reach the internet for normal outbound traffic, such as installing packages and downloading updates.

However, inbound VPN traffic was harder to test because the VM was not directly reachable from the physical network.

The issue was:

```text
Remote Client
        |
        | Attempts to reach VPN server
        |
Home Router
        |
        | Cannot directly reach VMware NAT-only VM IP
        |
LINUX01 inside VMware NAT
```

A better future design would be bridged networking:

```text
Remote Client
        |
        | WireGuard VPN
        |
Home Router
        |
        | Port forward UDP 51820
        |
LINUX01 with bridged network IP
```

With bridged networking, `LINUX01` would receive an IP address on the real home network, making it easier to forward UDP port `51820` from the router to the VPN server.

## Security Note

Before publishing this project to GitHub, sensitive information must be removed or replaced with placeholders.

Do not upload:

- WireGuard private keys
- Real public IP addresses
- Router login information
- Personal router details
- Passwords
- Tokens
- Full VPN client configuration files containing private keys
- Screenshots showing sensitive network information

Use placeholders such as:

```text
YOUR_SERVER_PRIVATE_KEY
YOUR_CLIENT_PRIVATE_KEY
YOUR_SERVER_PUBLIC_KEY
YOUR_CLIENT_PUBLIC_KEY
YOUR_PUBLIC_IP
YOUR_INTERNAL_DNS_SERVER
YOUR_SERVER_ENDPOINT
```

## Project Steps

### Step 1: Installed WireGuard on LINUX01

**What I did:**  
Installed WireGuard on the Ubuntu Server VM named `LINUX01`.

**Why I did it:**  
WireGuard was needed to create the VPN server for the lab.

**Commands or settings used:**

```bash
sudo apt install wireguard
```

**Verification:**  
WireGuard tools were installed and available on the server.

---

### Step 2: Generated Server Keys

**What I did:**  
Generated a WireGuard private and public key pair for the VPN server.

**Why I did it:**  
WireGuard uses public and private keys to authenticate the VPN server and clients.

**Commands or settings used:**

```bash
wg genkey | tee server_private.key | wg pubkey > server_public.key
```

To view the generated keys:

```bash
cat server_private.key
cat server_public.key
```

**Verification:**  
The server private key and server public key files were created.

**Security note:**  
The private key should never be uploaded to GitHub.

---

### Step 3: Created the WireGuard Server Configuration

**What I did:**  
Created the WireGuard server configuration file.

**Why I did it:**  
The server needed a VPN interface, VPN IP address, listening port, private key, and NAT rule.

**Commands or settings used:**

```bash
sudo vim /etc/wireguard/wg0.conf
```

Sanitized example configuration:

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = YOUR_SERVER_PRIVATE_KEY

PostUp = iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE
```

**Verification:**  
The server configuration was created using the `wg0` interface.

---

### Step 4: Confirmed the Linux Network Interface

**What I did:**  
Checked the Linux network interface name.

**Why I did it:**  
The iptables MASQUERADE rule needs the correct outbound network interface.

**Commands or settings used:**

```bash
ip a
```

**Verification:**  
The network interface was identified as:

```text
ens33
```

---

### Step 5: Started the WireGuard Service

**What I did:**  
Started the WireGuard interface using `wg-quick`.

**Why I did it:**  
Starting the service brings the VPN interface online.

**Commands or settings used:**

```bash
sudo systemctl start wg-quick@wg0
sudo wg
```

**Verification:**  
The VPN service came up and `sudo wg` showed WireGuard status information.

---

### Step 6: Checked for Missing WireGuard or Firewall Tools

**What I did:**  
Checked whether required tools such as `iptables` and `wg-quick` were installed.

**Why I did it:**  
WireGuard may fail to start if supporting commands are missing.

**Commands or settings used:**

```bash
which iptables
```

If `iptables` was missing:

```bash
sudo apt install iptables
```

Checked for `wg-quick`:

```bash
which wg-quick
```

If `wg-quick` was missing:

```bash
sudo apt install wireguard-tools
```

Restarted WireGuard:

```bash
sudo systemctl start wg-quick@wg0
```

**Verification:**  
WireGuard could start without missing-command errors.

---

### Step 7: Generated Client Keys

**What I did:**  
Generated a WireGuard private and public key pair for the VPN client.

**Why I did it:**  
Each VPN client needs its own key pair so the server can identify it as a peer.

**Commands or settings used:**

```bash
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

To view the generated keys:

```bash
cat client_private.key
cat client_public.key
```

**Verification:**  
The client private key and client public key files were created.

**Security note:**  
The client private key should never be uploaded to GitHub.

---

### Step 8: Added the Client Peer to the Server Configuration

**What I did:**  
Added the Windows VM client as a peer in the WireGuard server configuration.

**Why I did it:**  
The WireGuard server needs a peer entry for every allowed VPN client.

**Commands or settings used:**

```bash
sudo vim /etc/wireguard/wg0.conf
```

Sanitized peer example:

```ini
[Peer]
PublicKey = YOUR_CLIENT_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
```

Restarted WireGuard:

```bash
sudo systemctl restart wg-quick@wg0
```

Checked status:

```bash
sudo wg
```

**Verification:**  
The peer appeared in the WireGuard status output.

---

### Step 9: Created the Client Configuration

**What I did:**  
Created a WireGuard client configuration file.

**Why I did it:**  
The Windows VPN client needed a configuration file to import the tunnel.

**Commands or settings used:**

```bash
sudo vim /etc/wireguard/client.conf
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

**Verification:**  
The client configuration file was created.

**Security note:**  
This file contains a private key and should not be uploaded to GitHub.

---

### Step 10: Copied the Client Configuration for Windows Import

**What I did:**  
Prepared the client configuration file so it could be copied to the Windows VM.

**Why I did it:**  
The Windows WireGuard client needs the `.conf` file to import the tunnel.

**Commands or settings used:**

The original file was located in a restricted directory:

```text
/etc/wireguard/client.conf
```

To make it easier to copy, I copied it to the Linux user home directory:

```bash
sudo cp /etc/wireguard/client.conf /home/admin01/client.conf
sudo chmod 644 /home/admin01/client.conf
```

Then from Windows PowerShell, I copied the file using SCP:

```powershell
scp admin01@VPN_SERVER_IP:/home/admin01/client.conf C:\Users\USERNAME\Desktop\client.conf
```

**Verification:**  
The config file could be copied to the Windows desktop.

**Security note:**  
Changing the config file to readable permissions should only be temporary because the file contains a VPN private key.

---

### Step 11: Imported the Tunnel into WireGuard for Windows

**What I did:**  
Imported the client configuration into the Windows WireGuard application.

**Why I did it:**  
This was needed so the Windows VM could act as the VPN client.

**Commands or settings used:**

1. Opened WireGuard on Windows
2. Clicked **Add Tunnel**
3. Chose **Import from file**
4. Selected `client.conf`

**Verification:**  
The tunnel appeared in the Windows WireGuard client.

**Final result:**  
The VPN service was up on the server, but full client connection testing was not completed.

---

### Step 12: Generated a QR Code for Possible Phone Testing

**What I did:**  
Generated a QR code from the client configuration.

**Why I did it:**  
WireGuard on mobile can import a VPN profile by scanning a QR code.

**Commands or settings used:**

Installed QR code tool:

```bash
sudo apt install qrencode
```

Displayed the QR code in the terminal:

```bash
qrencode -t ansiutf8 < /home/admin01/client.conf
```

Generated a PNG QR code:

```bash
qrencode -o /home/admin01/client.png < /home/admin01/client.conf
```

Verified the file existed:

```bash
ls /home/admin01
```

**Verification:**  
A QR code image was created.

**Final result:**  
Phone testing was not completed.

---

### Step 13: Installed and Started SSH for SCP

**What I did:**  
Installed and started OpenSSH Server on the Linux VM.

**Why I did it:**  
SCP requires SSH to transfer files between Linux and Windows.

**Commands or settings used:**

```bash
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

**Verification:**  
SSH was running and available for SCP file transfer.

---

### Step 14: Documented NAT Limitation

**What I did:**  
Documented why full VPN testing was limited.

**Why I did it:**  
The VPN server remained behind VMware NAT, which made inbound VPN testing difficult.

**Commands or settings used:**  
No command fixed this in the final lab setup. The identified future fix is to use bridged networking or another reachable network design.

**Verification:**  
The VPN service came up, but a full successful client connection and remote access test was not confirmed.

## Troubleshooting

### Issue: WireGuard Service Did Not Start Cleanly

**Problem:**  
WireGuard did not start correctly during setup.

**Cause:**  
A possible cause was a missing dependency such as `iptables` or `wg-quick`.

**Fix:**  
Checked whether the required commands existed:

```bash
which iptables
which wg-quick
```

Installed missing tools if needed:

```bash
sudo apt install iptables
sudo apt install wireguard-tools
```

Restarted the WireGuard service:

```bash
sudo systemctl start wg-quick@wg0
```

**Verification:**  
Ran:

```bash
sudo wg
```

**Lesson learned:**  
WireGuard depends on supporting Linux networking tools. If the service fails, checking missing commands is a good troubleshooting step.

---

### Issue: Client Config Could Not Be Copied Directly from `/etc/wireguard/`

**Problem:**  
SCP could not directly copy the client configuration from `/etc/wireguard/`.

**Cause:**  
The `/etc/wireguard/` directory is restricted because it stores sensitive VPN configuration files and keys.

**Fix:**  
Copied the file to the Linux user home directory and temporarily changed permissions:

```bash
sudo cp /etc/wireguard/client.conf /home/admin01/client.conf
sudo chmod 644 /home/admin01/client.conf
```

Copied it from Windows:

```powershell
scp admin01@VPN_SERVER_IP:/home/admin01/client.conf C:\Users\USERNAME\Desktop\client.conf
```

**Verification:**  
The client configuration was copied to the Windows desktop.

**Lesson learned:**  
VPN config files contain sensitive keys. They should be handled carefully and should not be left world-readable longer than necessary.

---

### Issue: Phone QR Code Was Difficult to Scan

**Problem:**  
The phone did not properly scan the QR code displayed in the Linux terminal.

**Cause:**  
Terminal-rendered QR codes can be hard for a phone camera to read.

**Fix:**  
Generated the QR code as a PNG image:

```bash
qrencode -o /home/admin01/client.png < /home/admin01/client.conf
```

**Verification:**  
Verified the PNG existed:

```bash
ls /home/admin01
```

**Lesson learned:**  
A QR code image is usually easier to scan than a QR code displayed directly in the terminal.

---

### Issue: Full VPN Client Testing Was Not Completed

**Problem:**  
The VPN service came up, but the full client connection test was not completed.

**Cause:**  
The server stayed inside VMware NAT, and the lab setup did not allow the network design needed for a clean external client test.

**Fix:**  
No final fix was completed in this lab. The recommended future fix is to move the VPN server VM to bridged networking or build a network design where the VPN server has a reachable IP address.

**Verification:**  
WireGuard service status was verified, but a successful client tunnel and remote access test were not confirmed.

**Lesson learned:**  
A VPN server must be reachable by the VPN client. A running VPN service is not enough if the network path is blocked by NAT or virtualization limits.

---

### Issue: VMware NAT Limited Inbound VPN Testing

**Problem:**  
Testing inbound VPN connections was difficult because the VPN server was behind VMware NAT.

**Cause:**  
VMware NAT allows outbound traffic from the VM, but inbound traffic from another network does not automatically reach the VM.

**Fix:**  
The recommended future fix is bridged networking.

**Verification:**  
Bridged networking was identified as the better design, but it was not fully retested in this project.

**Lesson learned:**  
NAT is useful for giving VMs internet access, but bridged networking is usually easier when hosting services that need inbound connections, such as a VPN server.

## Verification and Testing

| Test | Result |
|---|---|
| WireGuard installed on LINUX01 | Completed |
| Ubuntu Server used for VPN server | Completed |
| Server keys generated | Completed |
| Client keys generated | Completed |
| Server config created | Completed |
| Client peer added to server config | Completed |
| Client config created | Completed |
| Linux interface confirmed as `ens33` | Completed |
| WireGuard service came up | Completed |
| `sudo wg` showed VPN status | Completed |
| Windows VM selected as VPN client | Completed |
| Full VPN client connection test | Not completed |
| Phone testing | Not completed |
| Remote access to internal services | Not completed |
| Bridged networking test | Not completed |
| Final successful VPN handshake | Not confirmed |

## Commands Used

### Install WireGuard

```bash
sudo apt install wireguard
```

### Generate Server Keys

```bash
wg genkey | tee server_private.key | wg pubkey > server_public.key
cat server_private.key
cat server_public.key
```

### Create Server Config

```bash
sudo vim /etc/wireguard/wg0.conf
```

### Check Network Interfaces

```bash
ip a
```

### Start WireGuard

```bash
sudo systemctl start wg-quick@wg0
sudo wg
```

### Check for iptables

```bash
which iptables
sudo apt install iptables
```

### Check for wg-quick

```bash
which wg-quick
sudo apt install wireguard-tools
```

### Generate Client Keys

```bash
wg genkey | tee client_private.key | wg pubkey > client_public.key
cat client_private.key
cat client_public.key
```

### Edit Server Config

```bash
sudo vim /etc/wireguard/wg0.conf
```

### Restart WireGuard

```bash
sudo systemctl restart wg-quick@wg0
sudo wg
```

### Create Client Config

```bash
sudo vim /etc/wireguard/client.conf
```

### Copy Client Config to User Home Directory

```bash
sudo cp /etc/wireguard/client.conf /home/admin01/client.conf
sudo chmod 644 /home/admin01/client.conf
```

### SCP Client Config from Windows

```powershell
scp admin01@VPN_SERVER_IP:/home/admin01/client.conf C:\Users\USERNAME\Desktop\client.conf
```

### Install QR Code Tool

```bash
sudo apt install qrencode
```

### Display QR Code in Terminal

```bash
qrencode -t ansiutf8 < /home/admin01/client.conf
```

### Generate QR Code PNG

```bash
qrencode -o /home/admin01/client.png < /home/admin01/client.conf
```

### Verify Files

```bash
ls /home/admin01
```

### Install and Start SSH Server

```bash
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

## Screenshots to Add Later

| Screenshot | What to Capture | Why It Matters | Suggested Filename |
|---|---|---|---|
| WireGuard installed | Terminal showing WireGuard installed or `wg` command available | Shows the VPN tool was installed | `wireguard-installed.png` |
| Server config sanitized | `wg0.conf` with keys removed | Shows server configuration safely | `wg0-config-sanitized.png` |
| WireGuard status | `sudo wg` output with sensitive keys hidden | Shows the VPN service came up | `wireguard-status.png` |
| Network interface | `ip a` showing `ens33` with sensitive details hidden | Shows the Linux interface used for NAT rule | `linux-interface-ens33.png` |
| Windows WireGuard client | Imported tunnel in the Windows WireGuard app | Shows the client setup process | `windows-wireguard-client.png` |
| QR code generation | QR code PNG file with sensitive details protected | Shows mobile setup attempt | `wireguard-qr-code.png` |
| VMware NAT settings | VM network adapter showing NAT mode | Documents the network limitation | `vmware-nat-settings.png` |
| Example bridged setting | VMware bridged network option | Shows the recommended future improvement | `vmware-bridged-option.png` |
| Network diagram | Simple NAT vs bridged VPN diagram | Explains the troubleshooting lesson | `vpn-network-diagram.png` |

## What I Learned

This project helped me understand the basic setup process for a WireGuard VPN server on Ubuntu Server. I practiced installing WireGuard, generating keys, creating a server configuration, creating a client configuration, adding peers, and starting the VPN service.

The biggest lesson was that VPN configuration and network design are both important. Even if the VPN service is running, remote clients still need a valid network path to reach the VPN server.

Because `LINUX01` stayed inside VMware NAT, outside remote access testing was limited. I learned that VMware NAT is useful for outbound traffic from a VM to the internet, but it makes inbound connections harder because the VM is not directly reachable from the physical network.

A better future test would be to move the VPN server to bridged networking or use a dedicated VM/network setup that allows proper client-to-server VPN testing.

## Recommended Improvements

- Retest the VPN server using bridged networking
- Confirm a successful VPN client handshake
- Test VPN access from a separate client network
- Test access to internal lab services through the VPN
- Confirm whether IP forwarding was enabled
- Confirm whether a host firewall rule was needed for UDP `51820`
- Create separate client configs for each device
- Remove temporary readable copies of VPN client configs after transfer
- Add monitoring or log review for WireGuard connection attempts
- Add a clean VPN network diagram to the repo
- Keep all private keys and public IP addresses out of GitHub

## Resume Bullets

- Configured a WireGuard VPN server on Ubuntu Server to practice VPN setup, key generation, peer configuration, and Linux network interface configuration.
- Troubleshot VPN testing limitations caused by VMware NAT and documented why inbound VPN access requires a reachable server IP or bridged networking.
- Created beginner-friendly VPN documentation covering server configuration, client configuration, NAT behavior, verification steps, and lessons learned.

## Interview Explanation

I configured a WireGuard VPN lab on an Ubuntu Server VM named `LINUX01`. I installed WireGuard, generated server and client keys, created the VPN server configuration, added a client peer, and configured the VPN subnet using `10.0.0.0/24`.

The VPN service came up, but I kept the VM on VMware NAT, so full remote access testing was limited. I learned that even when the VPN configuration is correct, the network design still matters. Since the VPN server was behind VMware NAT, it was difficult to test inbound VPN connections from another network.

The main takeaway from the project was understanding how WireGuard is configured and why bridged networking or a reachable server IP is usually needed when hosting a VPN server in a lab.
