# 🪟 Installing Windows Server 2019 on a Virtual Machine

> Step-by-step lab guide covering VM setup, Windows Server 2019 installation, post-install configuration, and an introduction to Server Manager.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D4?style=flat-square&logo=windows&logoColor=white)
![VMware](https://img.shields.io/badge/Hypervisor-VMware-607D8B?style=flat-square)
![Lab](https://img.shields.io/badge/Type-Hands--on%20Lab-4CAF50?style=flat-square)
![Level](https://img.shields.io/badge/Level-Beginner-orange?style=flat-square)

---

## 📖 Overview

This lab walks you through the complete process of installing Windows Server 2019 inside a VMware virtual machine, applying the three essential post-install configurations, and getting familiar with the Server Manager dashboard — the central hub for all server administration tasks.

---

## 🎯 What This Lab Covers

| Topic | Description |
|---|---|
| VM creation | Name, disk allocation, ISO loading in VMware |
| Windows Server 2019 install | Edition selection, partitioning, password setup |
| Password security | Complexity rules enforced by Windows Server |
| Post-install configuration | Date/time, IP address, computer name |
| Server Manager | Dashboard overview and key management areas |

---

## 💻 VM Hardware Requirements

| Resource | Minimum | Recommended |
|---|---|---|
| RAM | 2 GB | 4 GB |
| CPU | 1 vCPU | 2 vCPUs |
| Disk space | 60 GB | 80 GB (dynamically allocated) |
| Network adapter | NAT | Host-only (for isolated lab) |

> ⚠️ Make sure virtualization (VT-x / AMD-V) is enabled in BIOS/UEFI before starting.

---

## 🗂️ Step 1 — Create the Virtual Machine in VMware

```
1. Open VMware Workstation → "Create a New Virtual Machine"
        ↓
2. Select "Typical (recommended)"
        ↓
3. Choose "Installer disc image file (ISO)" → browse to Windows Server 2019 ISO
        ↓
4. Set a clear VM name (e.g., "WinServer2019-DC")
        ↓
5. Set disk size (recommended: 80 GB) → Store as single file
        ↓
6. Customize hardware:
   ├── RAM: 2–4 GB
   ├── CPUs: 1–2 cores
   └── Network adapter: Host-only or NAT
        ↓
7. Click Finish → Power on the VM
```

---

## 🔧 Step 2 — Install Windows Server 2019

```
1. Boot from ISO → Windows Setup screen appears
        ↓
2. Select language, time format, and keyboard layout → Next
        ↓
3. Click "Install Now"
        ↓
4. Select edition:
   ├── Windows Server 2019 Standard (recommended for labs)
   └── Windows Server 2019 Datacenter (for advanced scenarios)
        ↓
5. Choose "Desktop Experience" for a GUI (not Core)
        ↓
6. Accept license terms
        ↓
7. Select "Custom: Install Windows only (advanced)"
        ↓
8. Partition the disk:
   └── Select unallocated space → New → Apply → Format → Next
        ↓
9. Installation begins — VM reboots automatically
        ↓
10. Set the Administrator password (see password rules below)
        ↓
11. Press Ctrl+Alt+Del (or VMware shortcut) → Log in
```

---

## 🔐 Password Complexity Requirements

Windows Server 2019 enforces strict password rules by default. Your Administrator password **must** meet all of the following:

| Rule | Requirement |
|---|---|
| Minimum length | At least **7 characters** (12+ recommended) |
| Uppercase letters | At least one (A–Z) |
| Lowercase letters | At least one (a–z) |
| Numbers | At least one (0–9) |
| Special characters | At least one (e.g., `!`, `@`, `#`, `$`) |
| No username | Cannot contain the account name |

> 💡 **Example of a strong password:** `Admin@2019!` — meets all rules and is easy to remember in a lab context.

---

## ⚙️ Step 3 — Post-Install Configuration (Critical)

After the first login, perform these **three mandatory configurations** before anything else. Skipping them can cause network and operational issues later.

### 1. Set Date and Time

```
Right-click taskbar clock → Adjust date/time
└── Set correct timezone, date, and time
    (Incorrect time breaks Kerberos authentication in Active Directory)
```

### 2. Set a Static IP Address

```
Server Manager → Local Server → Ethernet → Properties
└── Internet Protocol Version 4 (TCP/IPv4) → Use the following IP address:

    IP Address:      192.168.1.10   (example — match your lab network)
    Subnet Mask:     255.255.255.0
    Default Gateway: 192.168.1.1
    DNS Server:      127.0.0.1      (points to itself after AD DS install)
```

> ⚠️ A static IP is **required** for a server. Dynamic IPs from DHCP will cause clients to lose contact with the server after a lease renewal.

### 3. Rename the Computer

```
Server Manager → Local Server → Computer Name → Change
└── Set a clear, meaningful name (e.g., "DC01" or "SRV-MAIN")
    → Restart when prompted
```

### Post-Install Checklist

- [ ] Date and time set correctly with proper timezone
- [ ] Static IP address assigned
- [ ] Computer renamed to a meaningful name
- [ ] Server restarted after rename
- [ ] VM snapshot taken as a clean baseline

---

## 📊 Step 4 — Server Manager Overview

Server Manager opens automatically on login and is the central dashboard for all server administration. Key areas to know:

| Section | Purpose |
|---|---|
| **Dashboard** | Health overview — warnings, events, services at a glance |
| **Local Server** | Quick access to computer name, IP, Windows Update, firewall, time |
| **All Servers** | Manage multiple servers from one pane |
| **Add Roles and Features** | Install server roles (AD DS, DNS, DHCP, IIS, etc.) |
| **Tools menu** | Launch admin tools — Active Directory, DNS Manager, Event Viewer, etc. |

```
Server Manager layout:

┌─────────────────────────────────────────────────┐
│  Dashboard                                       │
│  ├── Roles and Server Groups                     │
│  ├── Events (errors / warnings)                  │
│  ├── Services (running / stopped)                │
│  └── Performance (CPU / memory alerts)           │
│                                                  │
│  Local Server                                    │
│  ├── Computer Name        DC01                   │
│  ├── IP Address           192.168.1.10 (static)  │
│  ├── Windows Firewall     On                     │
│  ├── Remote Desktop       Enabled / Disabled     │
│  └── Windows Update       Status                 │
└─────────────────────────────────────────────────┘
```

---

## 🧪 Lab Environment Reference

| Component | Value |
|---|---|
| Hypervisor | VMware Workstation / Player |
| Guest OS | Windows Server 2019 (Desktop Experience) |
| VM name | `WinServer2019-DC` (example) |
| Static IP | `192.168.1.10` (adjust to your lab subnet) |
| Admin password | Complex — meets all 5 complexity rules |
| Network mode | Host-only (isolated lab) |

---

## 📝 Key Takeaways

- Always use **Desktop Experience** edition for GUI-based administration in labs.
- The three post-install steps — **date/time, static IP, rename** — must be done before joining or creating a domain.
- **Server Manager** is your primary tool; learn it well before moving to Active Directory.
- Take a **VM snapshot** after a clean, configured install so you can roll back any time.
- Password complexity is enforced by default and cannot be bypassed during setup.

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
