# 📡 Lecture: DHCP — Dynamic Host Configuration Protocol

**Topic:** DHCP Protocol — Concepts, Configuration & Troubleshooting  
**Platform:** Windows Server 2016 / 2019  
**Difficulty:** Beginner–Intermediate

---

## 📋 Table of Contents

1. [What is DHCP?](#1-what-is-dhcp)
2. [IP Address Assignment Methods](#2-ip-address-assignment-methods)
3. [IP Address Structure & Classes](#3-ip-address-structure--classes)
4. [Types of IP Communication](#4-types-of-ip-communication)
5. [Loopback Address & NIC Testing](#5-loopback-address--nic-testing)
6. [Installing DHCP on Windows Server](#6-installing-dhcp-on-windows-server)
7. [DORA Process](#7-dora-process)
8. [Lease Duration & Renewal](#8-lease-duration--renewal)
9. [MAC Address Binding & Reservations](#9-mac-address-binding--reservations)
10. [APIPA — Automatic Private IP Addressing](#10-apipa--automatic-private-ip-addressing)
11. [Troubleshooting IP & Connectivity Issues](#11-troubleshooting-ip--connectivity-issues)
12. [Scope Planning & Exclusion Ranges](#12-scope-planning--exclusion-ranges)
13. [DHCP Scope Options](#13-dhcp-scope-options)
14. [MAC Address Filtering](#14-mac-address-filtering)
15. [Useful Commands](#15-useful-commands)
16. [Key Terms](#16-key-terms)

---

## 1. What is DHCP?

**Dynamic Host Configuration Protocol (DHCP)** is a standard network protocol used to automatically distribute IP configurations to devices on a network.

- Not exclusive to Windows Server — runs on Linux, routers, and firewalls too
- Recommended on **Windows Server** for better control and AD integration
- Operates over **UDP port 67** (server) and **UDP port 68** (client)
- Every device must have an IP address to communicate on the network

---

## 2. IP Address Assignment Methods

| Method | Description | Best For |
|---|---|---|
| **Static / Manual** | IP set manually via Network Settings | Servers, printers, critical devices |
| **DHCP (Dynamic)** | IP assigned automatically by DHCP server | Workstations, laptops, mobile devices |

Manual assignment becomes unmanageable at scale (hundreds of devices) — conflicts and administrative overhead increase significantly. DHCP solves this.

---

## 3. IP Address Structure & Classes

An IPv4 address consists of **4 octets**, each 8 bits (0–255).

Every address has two parts:
- **Network portion** — identified by subnet mask bits set to `255`
- **Host portion** — unique per device within the subnet

### IP Address Classes

| Class | Range | Default Subnet Mask | Use |
|---|---|---|---|
| A | 1.0.0.0 – 126.255.255.255 | 255.0.0.0 (/8) | Large networks |
| B | 128.0.0.0 – 191.255.255.255 | 255.255.0.0 (/16) | Medium networks |
| C | 192.0.0.0 – 223.255.255.255 | 255.255.255.0 (/24) | Small networks (most common) |
| D | 224.0.0.0 – 239.255.255.255 | — | Multicast only |
| E | 240.0.0.0 – 254.255.255.255 | — | Research/experimental |

### Reserved Addresses (cannot be assigned to hosts)

- **Network ID** — first address in subnet (e.g., `192.168.1.0`)
- **Broadcast** — last address in subnet (e.g., `192.168.1.255`)
- **Loopback** — `127.0.0.0/8` range, commonly `127.0.0.1`

---

## 4. Types of IP Communication

| Type | Description | Example |
|---|---|---|
| **Unicast** | One-to-one | Client → Server |
| **Multicast** | One-to-many (specific group) | Video streaming to subscribers |
| **Broadcast** | One-to-all (entire subnet) | DHCP Discover |

---

## 5. Loopback Address & NIC Testing

The loopback address `127.0.0.1` is used to test whether the local **Network Interface Card (NIC)** is functioning correctly.

```cmd
ping 127.0.0.1
```

- **4 replies** = NIC is functional
- **No reply** = NIC hardware or driver issue

This is the first diagnostic step when troubleshooting network connectivity.

---

## 6. Installing DHCP on Windows Server

1. Open **Server Manager → Add Roles and Features**
2. Select **DHCP Server** role → complete installation
3. After installation, click **Complete DHCP Configuration**
4. **Authorize** the DHCP server in Active Directory:
   - Requires **Domain Admin**, **Schema Admin**, or **Enterprise Admin** credentials
   - Authorization prevents rogue DHCP servers from distributing IPs on the domain
5. Create a **Scope** (IP address pool) — see [Section 12](#12-scope-planning--exclusion-ranges)

---

## 7. DORA Process

When a client device is set to **Obtain an IP address automatically**, the following 4-step exchange takes place:

```
Client                          DHCP Server
  |                                  |
  |---- DHCP Discover (broadcast) -->|   "I need an IP address"
  |                                  |
  |<--- DHCP Offer ------------------|   "Here's 192.168.1.82, want it?"
  |                                  |
  |---- DHCP Request (broadcast) --->|   "Yes, I'll take that IP"
  |                                  |
  |<--- DHCP Acknowledge (ACK) -------|   "Confirmed, it's yours for 8 days"
  |                                  |
```

### What the server provides in the Offer:

| Option | Example Value |
|---|---|
| IP Address | 192.168.1.82 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.1.1 |
| DNS Server(s) | 192.168.1.2, 192.168.1.3 |
| Domain Name | test.local |
| Lease Duration | 8 days |

---

## 8. Lease Duration & Renewal

DHCP assigns IPs for a limited **lease duration** (default ~8 days on wired networks).

### Renewal Timeline

| Point | Action |
|---|---|
| **50% of lease elapsed** | Client sends Renewal Request to DHCP server |
| **87.5% of lease elapsed** | Client sends another Renewal Request (fallback) |
| **100% (lease expired)** | DHCP server releases the IP — available for reassignment |

This mechanism prevents IP conflicts and ensures efficient address reuse for devices that leave the network.

---

## 9. MAC Address Binding & Reservations

- DHCP maintains a **database** mapping MAC addresses to assigned IPs within the scope
- IPs are assigned **sequentially** within the defined range
- **Reservations** bind a specific MAC address to a fixed IP — the device always receives the same address without manual static IP configuration

### Creating a Reservation

In DHCP Manager → expand Scope → **Reservations** → right-click → **New Reservation**:
- Reservation name
- IP address (must be within scope range)
- MAC address (format: `00-1A-2B-3C-4D-5E`)

---

## 10. APIPA — Automatic Private IP Addressing

If a device **cannot reach a DHCP server** after multiple attempts (retrying every 3 minutes), Windows automatically assigns an **APIPA** address:

$$169.254.0.0 \text{ – } 169.254.255.255$$

- Allows **local communication only** — no internet or domain access
- Device continues broadcasting to find a DHCP server in the background
- Many devices showing `169.254.x.x` = **DHCP server is down or unreachable**
- Can cause **broadcast storms** on large networks

---

## 11. Troubleshooting IP & Connectivity Issues

Users typically report "no internet" or "application not working" — the root cause is often IP assignment failure. Follow these steps:

1. **Ping loopback** → `ping 127.0.0.1` — confirms NIC is working
2. **Check IP address** → `ipconfig` — look for `169.254.x.x` (APIPA = no DHCP)
3. **Assign manual IP** → test if device can communicate on the subnet
4. **Check physical layer** → reseat cables, verify switch port LED
5. **Check DHCP server** → confirm service is running and scope has available addresses
6. **Release and renew** → `ipconfig /release` then `ipconfig /renew`

---

## 12. Scope Planning & Exclusion Ranges

A DHCP scope must match the subnet of the server's NIC. Plan carefully to avoid conflicts with statically assigned devices.

### Example Scope Plan (`192.168.1.0/24`)

| Range | Purpose |
|---|---|
| 192.168.1.1 | Default Gateway (router) |
| 192.168.1.2 | Domain Controller (PDC) |
| 192.168.1.3 | Secondary DC |
| 192.168.1.4 – 192.168.1.29 | Static servers (web, file, print) |
| **192.168.1.30 – 192.168.1.254** | **DHCP scope (dynamic pool)** |
| 192.168.1.50 – 192.168.1.53 | Exclusion range (printers) |

Always **exclude** the IP ranges used by static devices within the scope to prevent duplicate address conflicts.

---

## 13. DHCP Scope Options

DHCP distributes more than just an IP address. Configure these under **Scope Options** or **Server Options**:

| Option Code | Option | Example |
|---|---|---|
| 003 | Default Gateway | 192.168.1.1 |
| 006 | DNS Servers | 192.168.1.2, 192.168.1.3 |
| 015 | Domain Name | zohdy.local |
| 042 | NTP/Time Server | 192.168.1.2 |
| 066/067 | WDS Boot Server/File | For PXE network booting |

---

## 14. MAC Address Filtering

DHCP supports two filtering modes to control which devices receive an IP:

| Mode | Behavior |
|---|---|
| **Allow List** | Only MAC addresses on the list receive an IP |
| **Deny List** | MAC addresses on the list are blocked from receiving an IP |

Useful for preventing unauthorized devices (e.g., rogue laptops, Wi-Fi freeloaders) from joining the network.

---

## 15. Useful Commands

```cmd
ipconfig                  # View current IP configuration
ipconfig /all             # View full config including MAC address
ipconfig /release         # Release current DHCP lease
ipconfig /renew           # Request a new IP from DHCP server
ping 127.0.0.1            # Test local NIC
ping <gateway IP>         # Test connectivity to default gateway
```

---

## 16. Key Terms

| Term | Definition |
|---|---|
| **DHCP** | Dynamic Host Configuration Protocol — auto IP distribution |
| **DORA** | Discover, Offer, Request, Acknowledge — DHCP exchange sequence |
| **Scope** | Defined pool of IP addresses the DHCP server can distribute |
| **Exclusion Range** | IPs within the scope reserved for static assignment |
| **Reservation** | Fixed IP binding to a specific MAC address |
| **Lease** | Time period for which a DHCP-assigned IP is valid |
| **APIPA** | Automatic Private IP Addressing — self-assigned `169.254.x.x` when no DHCP |
| **Loopback** | `127.0.0.1` — used to test the local NIC |
| **Unicast** | One-to-one IP communication |
| **Multicast** | One-to-many IP communication (Class D) |
| **Broadcast** | One-to-all communication within subnet |
| **UDP** | Connectionless transport protocol used by DHCP (ports 67/68) |

---

## 👨‍💻 Author

> Lecture notes prepared for Windows Server administration coursework.  
> Topic: DHCP Protocol — Configuration & Troubleshooting on Windows Server.
