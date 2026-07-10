# 1. Infrastructure Setup

## Overview
This section covers the initial setup of the Ubuntu Server that will host the Splunk SIEM, including network configuration, static IP assignment, timezone configuration, and firewall setup. All steps here were completed before any security tooling was installed.

## Environment
| Detail | Value |
|---|---|
| OS | Ubuntu Server 24.04 LTS |
| IP Address | 10.0.0.x |
| Network Interface | eno2 (Ethernet) |
| Gateway | 10.0.0.1 |
| DNS | 8.8.8.8, 8.8.4.4 |

## Steps Completed

### Step 1 - Ubuntu Server Installation
- Downloaded Ubuntu Server 24.04 LTS ISO
- Created bootable USB
- Booted target PC and completed installation
- Selected OpenSSH Server during setup for remote access

### Step 2 - System Updates
First action after login was to update all packages before installing anything:
```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```
**Why:** Running updates before installing software ensures security patches are applied and prevents compatibility issues.

### Step 3 - Static IP Configuration
The server was initially connected via WiFi. Switched to ethernet and configured a permanent static IP address.

Edited Netplan configuration:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Applied this configuration:
```yaml
network:
  version: 2
  ethernets:
    eno2:
      dhcp4: no
      addresses:
        - 10.0.0.x/24
      routes:
        - to: default
          via: 10.0.0.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

```bash
sudo netplan apply
```

**Why static IP:** A SIEM server needs a permanent address. If the IP changes after reboot, forwarders lose connection and logs stop flowing, creating gaps in monitoring coverage.

**Why ethernet over WiFi:** WiFi can drop connections and is less reliable for a server that needs to be always available. Ethernet provides stable consistent connectivity.

### Step 4 - Timezone Configuration
The server defaulted to UTC. Changed to local timezone to ensure log timestamps match the monitored endpoints.

```bash
sudo timedatectl set-timezone America/New_York
timedatectl
```

**Why this matters:** If the SIEM server and monitored endpoints are in different timezones, log timestamps will not align. A security event at 9PM Eastern would appear as 1AM UTC, making time based searches inaccurate and investigations unreliable.

### Step 5 - UFW Firewall Configuration
Enabled the firewall before any services were installed. Only opened ports needed for the SIEM deployment covered in the next section.

```bash
sudo ufw allow 22/tcp
sudo ufw allow 8000/tcp
sudo ufw allow 9997/tcp
sudo ufw enable
sudo ufw status
```

| Port | Protocol | Purpose |
|---|---|---|
| 22 | TCP | SSH remote access to server |
| 8000 | TCP | Splunk web dashboard |
| 9997 | TCP | Splunk log receiving from forwarders |

**Why:** Servers should only accept traffic on ports they actively use. All other traffic is blocked by default. This follows the principle of least privilege.

## Key Concepts Learned
- YAML syntax and indentation rules
- DHCP vs static IP addressing
- Network interfaces, ethernet vs WiFi
- UFW firewall port management
- Principle of least privilege
- Timezone synchronization in SOC environments
- Netplan as Ubuntu's network configuration tool

## Verification Commands
```bash
ip a              # verify static IP on eno2
sudo ufw status   # verify firewall rules active
timedatectl       # verify correct timezone
ping 10.0.0.1     # verify gateway connectivity
```

## Next Section
[2. Splunk SIEM Deployment](../2-Splunk-SIEM-Deployment/README.md)
