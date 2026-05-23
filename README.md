# Security Onion 2.4 Deployment and Troubleshooting in VirtualBox

## Project Overview

This project documents the deployment of Security Onion 2.4 inside a VirtualBox home lab environment and the troubleshooting process required to successfully access the SOC interface.

Rather than simply installing the software, this project focused on:

- Virtual network design
- Multi-NIC configuration
- Security Onion 2.4 architecture changes
- Firewall troubleshooting
- Connectivity testing
- Understanding management vs monitoring interfaces
- SOC access configuration

---

## Objective

Deploy a working Security Onion SIEM/NDR platform capable of:

- Centralized log management
- Network monitoring
- Traffic analysis
- Threat detection
- Future integration with lab systems such as:

    - Kali Linux
    - Windows systems
    - Ubuntu servers
    - Wazuh agents
    - Malware analysis systems

---

## Lab Environment

### Host System

- Windows 11

### Virtualization Platform

- Oracle VirtualBox

### Security Platform

- Security Onion 2.4.200

### Network Layout

| System | IP Address | Purpose |
|----------|------------|----------|
| Windows Host | 192.168.120.1 | VirtualBox Host Adapter |
| Kali Linux | 192.168.120.100 | Attack/Testing VM |
| Security Onion | 192.168.120.130 | SIEM/NDR |
| Network | 192.168.120.0/24 | Host-only network |

---

## VirtualBox Configuration

Security Onion requires multiple network interfaces.

### Adapter Configuration

Adapter 1:

- Host-only Adapter #4
- Used for:
    - Internal lab communication
    - SOC access
    - Sensor traffic

Adapter 2:
- NAT
- Used for:
    - Internet access
    - Updates
    - Package downloads

---

## Installation Process

### Step 1: Create VM

VM settings:

CPU:
- 6 cores

RAM:
–16GB

Storage:
- 300GB VDI

---

### Step 2: Configure Networking

Initially:

Adapter 1:
- Host-only adapter

Adapter 2:
- Nat

---

### Step 3: Start Security Onion Setup

Run:

```bash
sudo so-setup
```

The installer detected:

> This machine currently has 1 NIC but needs 2 to meet minimum requirements.

This required adding a second network adapter.

---

### Step 4: Select Management Interface

During installation:

Management Interface:

```text
enp0s3
```

Reason:

- Connected to management network
- Assigned:

```text
192.168.120.130/24
```

Monitoring interface:

```text
enp0s8
```

Used for monitoring and sensor traffic.

---

### Step 5: Configure Gateway

Gateway:

```text
192.168.120.1
```

DNS:

```text
8.8.8.8
```

---

## Initial Installation Failure

During installation:

```text
curl: (6) Could not resolve host:
sigs.securityonion.net
```

Setup repeatedly timed out.

---

## Root Cause

Security Onion could not reach the internet because of an incorrect network setup.

Only one interface had been configured correctly.

No internet connectivity meant:

- Package downloads failed
- Repository synchronization failed
- Installation stopped

---

## Troubleshooting Steps

### Verify routing

```bash
ip route
```

Observed:

```bash
default via 10.0.2.2
```

---

### Verify interface assignments

```bash
ip a
```

Confirmed:

Management interface:

```text
enp0s3
192.168.120.130
```

Monitoring interface:

```text
enp0s8
```

---

### Test connectivity

Ping host:

```bash
ping 192.168.120.1
```

Ping Kali:

```bash
ping 192.168.120.100
```

---

## Security Onion 2.4 Changes Discovered

While troubleshooting, older commands from previous versions did not work.

Attempted:

```bash
sudo so-setup
sudo so-status
sudo so-allow
```

Observed:

```text
command not found
```

---

## Difference Between Older Versions and Security Onion 2.4

Security Onion 2.4 significantly changed management tools.

Older versions relied on:

- so-allow
- sosetup
- manual firewall scripts

Security Onion 2.4 uses:

- so-firewall
- containerized architecture
- Docker-based services
- updated management workflows

---

## Firewall Access Configuration

SOC access required explicitly allowing hosts.

Allowed Windows host:

```bash
sudo so-firewall includehost analyst 192.168.120.1
```

Allowed Kali:

```bash
sudo so-firewall includehost analyst 192.168.120.100
```

Restarted firewall:

```bash
sudo so-firewall apply
```

---

## Verification

Verify services:

```bash
sudo so-status
```

Observed:

- Elasticsearch running
- Kibana running
- Zeek running
- Suricata running
- SOC services healthy

---

## Access SOC Interface

From browser:

```text
https://192.168.120.130
```

---

## Lessons Learned

This deployment highlighted several concepts:

- Security Onion requires multiple NICs
- NAT and Host-only adapters serve different purposes
- Network troubleshooting is critical
- Routing verification prevents installation failures
- Security Onion 2.4 introduced major changes from previous versions
- Understanding management vs monitoring interfaces is important
- Containerized security platforms behave differently from traditional applications

---

## Skills Demonstrated

- VirtualBox deployment
- Network troubleshooting
- Linux administration
- Routing analysis
- Security Onion deployment
- Firewall configuration
- SIEM implementation
- Documentation
- Problem solving
