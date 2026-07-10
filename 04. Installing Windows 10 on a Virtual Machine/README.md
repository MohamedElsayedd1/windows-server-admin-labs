# 🖥️ Windows Server Network Administration

> Structured notes, hands-on lab guides, and configuration references for a complete course on network administration using Windows Server — MCSE-aligned curriculum.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2022-0078D4?style=flat-square&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active%20Directory-Enabled-4CAF50?style=flat-square)
![Lab](https://img.shields.io/badge/Lab-VMware-607D8B?style=flat-square)
![Level](https://img.shields.io/badge/Level-Beginner%20Friendly-orange?style=flat-square)

---

## 📖 About

This repository contains everything you need to follow along with a comprehensive Windows Server network administration course. The goal is to take you from zero to confidently managing a domain environment — users, policies, DNS, and more — through a mix of theory and hands-on labs.

No prior server experience or certifications (like CCNA) are required. Only basic Windows familiarity is assumed.

---

## 🎯 What You Will Learn

| Topic | Description |
|---|---|
| Networking fundamentals | LAN, WAN, switches, routers, IP addressing |
| Workgroup vs domain | When to use each and why domains win at scale |
| Active Directory & AD DS | Install, promote to DC, manage users and policies |
| DNS configuration | Name resolution, zones, integration with Active Directory |
| Server installation & setup | Static IP, server naming, roles and features |
| Virtualization & VMware | Run multiple VMs on one machine for safe lab practice |
| VMware installation | Download, extract, install, and configure VMware Workstation |
| Windows 10 VM setup | Create a Windows 10 guest VM, partition disk, and install VMware Tools |

---

## ✅ Prerequisites

- Basic computer skills
- Familiarity with Windows client OS (navigating, installing programs)
- No CCNA or prior server experience needed

---

## 🗂️ Repo Structure

```
windows-server-admin/
├── 01-networking-fundamentals/
│   └── notes.md
├── 02-workgroup-vs-domain/
│   └── notes.md
├── 03-server-installation/
│   ├── notes.md
│   └── lab.md
├── 04-active-directory/
│   ├── notes.md
│   └── lab.md
├── 05-dns-configuration/
│   ├── notes.md
│   └── lab.md
├── 06-vmware-lab-setup/
│   ├── notes.md
│   └── lab.md
├── 07-windows10-vm-install/
│   ├── notes.md
│   └── lab.md
└── README.md
```

---

## 💻 Virtualization Concepts

Virtualization allows multiple operating systems to run **simultaneously on one physical machine** by creating isolated virtual environments. Understanding it is essential before building the lab.

### Host vs Guest

| Term | Meaning |
|---|---|
| Host machine | Your physical computer running the hypervisor (e.g., Windows 10/11, 64-bit) |
| Guest machine | A virtual machine created inside the host, running its own OS |
| Hypervisor | Software layer (VMware / Hyper-V) that manages VM creation and resource allocation |

### Resource Allocation Guide

| Resource | How it works | Recommendation |
|---|---|---|
| RAM | Taken directly from host physical RAM | Leave at least 3–4 GB for the host OS |
| Disk | Dynamically allocated — only used space is consumed | SSD strongly recommended for performance |
| CPU | Virtual cores time-share the physical CPU via hypervisor scheduling | 1–2 vCPUs per VM is sufficient for labs |
| Network adapter | Connected through a virtual switch | Use Host-only for isolation, Bridged for internet |

### Network Modes

| Mode | Description | Use case |
|---|---|---|
| Host-only | VM traffic stays inside the host machine | Secure, isolated lab environments |
| Bridged | VM gets direct access to the external network | Simulating real-world connectivity |

### VM Setup Workflow

```
1. Install hypervisor (VMware Workstation / Player)
        ↓
2. Create new VM — set OS type, RAM, disk size, CPU cores
        ↓
3. Allocate resources carefully — don't over-commit RAM
        ↓
4. Configure virtual network adapter (Host-only or Bridged)
        ↓
5. Power on VM and install the guest OS
```

---

## 📦 Installing VMware Workstation

> Based on the lab tutorial by **Mohamed Zohdy**. Follow these steps before creating any VMs.

### System Requirements

| Requirement | Detail |
|---|---|
| OS | 64-bit Windows (Windows 10 / Windows Server) |
| Virtualization | VT-x / AMD-V enabled in BIOS/UEFI |
| Tools needed | WinRAR (to extract the installer archive) |

### Installation Steps

```
1. Download VMware Workstation installer
        ↓
2. Download and install WinRAR (if not already installed)
        ↓
3. Extract the VMware installer archive using WinRAR
        ↓
4. Run the extracted setup (.exe) as Administrator
        ↓
5. Follow the installation wizard — accept defaults
        ↓
6. On the Virtual Network Editor step:
   └── Disable DHCP on virtual adapters (VMnet1 / VMnet8)
        ↓
7. Complete installation and restart if prompted
```

### Virtual Network Configuration

After installation, VMware creates virtual network adapters on your host. Configure them correctly for lab use:

| Adapter | Mode | DHCP | Recommended action |
|---|---|---|---|
| VMnet1 | Host-only | Disable | Used for isolated internal lab network |
| VMnet8 | NAT | Disable | Used for internet-connected VMs |

> ⚠️ **Disable DHCP** on both virtual adapters. IP addresses will be assigned manually to each VM for full control over the network configuration.

---

## 🪟 Installing Windows 10 on a Virtual Machine

> Based on the lab tutorial by **Mohamed Zohdy**. Complete the [VMware installation](#-installing-vmware-workstation) first.

### VM Hardware Requirements

| Resource | Minimum | Recommended |
|---|---|---|
| RAM | 2 GB | 4 GB |
| CPU | 1 vCPU | 2 vCPUs |
| Disk space | 50 GB | 80 GB (dynamically allocated) |
| Network adapter | NAT or Bridged | Bridged (for internet access during setup) |

### Step 1 — Download the Windows 10 ISO

- Download the official Windows 10 ISO from Microsoft's website using the **Media Creation Tool**.
- Choose the correct **edition** (Home / Pro) to avoid compatibility issues later.
- Save the ISO somewhere easy to find (e.g., `D:\ISOs\win10.iso`).

### Step 2 — Create a New Virtual Machine in VMware

```
1. Open VMware Workstation → Click "Create a New Virtual Machine"
        ↓
2. Choose "Typical (recommended)" configuration
        ↓
3. Select "Installer disc image file (ISO)" → browse to your Win10 ISO
        ↓
4. Set VM name and storage location
        ↓
5. Set disk size (recommended: 80 GB) → Store as a single file
        ↓
6. Customize hardware:
   ├── RAM: 2–4 GB
   ├── CPUs: 1–2 cores
   └── Network adapter: Bridged or NAT
        ↓
7. Click Finish
```

### Step 3 — Install Windows 10

```
1. Power on the VM
        ↓
2. Boot from ISO → Windows Setup screen appears
        ↓
3. Choose language, time, and keyboard → click Install Now
        ↓
4. Select edition (e.g., Windows 10 Pro)
        ↓
5. Accept license terms
        ↓
6. Choose "Custom: Install Windows only (advanced)"
        ↓
7. Partition the disk:
   └── Select unallocated space → click New → Apply → Format → Next
        ↓
8. Windows installs and reboots automatically
        ↓
9. Complete OOBE setup:
   ├── Create a local user account with administrative privileges
   └── Skip Microsoft account sign-in (recommended for lab VMs)
```

### Step 4 — Install VMware Tools

After Windows 10 is running, install VMware Tools to optimize the guest VM:

```
VMware menu → VM → Install VMware Tools → Run setup inside the VM
```

| VMware Tools benefit | Description |
|---|---|
| Better graphics | Enables full-resolution display and smooth rendering |
| Seamless mouse | Mouse moves freely between host and guest without capture |
| Shared clipboard | Copy/paste between host and guest OS |
| File sharing | Drag-and-drop files between host and guest |
| Better performance | Improved I/O and overall VM responsiveness |

### Step 5 — Post-Install Checklist

- [ ] Windows 10 VM boots successfully
- [ ] Local admin user account created
- [ ] VMware Tools installed
- [ ] Network adapter configured (Bridged or NAT)
- [ ] Internet access verified (if using Bridged/NAT)
- [ ] VM snapshot taken as a clean baseline backup

> 💡 **Tip:** Take a snapshot immediately after a clean install. If anything breaks during labs, you can roll back in seconds.

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation or VMware Player |
| Server VM | Windows Server 2022 |
| Client VM(s) | Windows 10 / Windows 11 (1–2 machines) |
| Network | Host-only (isolated lab) or Bridged (internet access) |

> ⚠️ **Important:** Enable virtualization (VT-x / AMD-V) in your BIOS/UEFI before starting the labs, or the VMs will not run.

---

## 🚀 Getting Started

1. **Set up VMware** — Follow the [VMware installation steps](#-installing-vmware-workstation) above. Disable DHCP on VMnet1 and VMnet8 during setup.
2. **Create the Server VM** — Install Windows Server 2022, assign a static IP, and give it a meaningful name.
3. **Install AD DS** — Add the Active Directory Domain Services role via Server Manager.
4. **Promote to Domain Controller** — Run the AD DS configuration wizard and set your domain name (e.g., `company.local`).
5. **Create Client VMs** — Install Windows 10/11 and join them to the domain.
6. **Follow the labs** — Work through each folder in order.

---

## 📚 Key Concepts

### Workgroup vs Domain

| Aspect | Workgroup | Domain |
|---|---|---|
| Management | Decentralized, per device | Centralized via Domain Controller |
| User accounts | Local to each device | Stored centrally on Domain Controller |
| Passwords | Managed per device | Single sign-on across domain |
| Best for | Very small networks (2–3 devices) | Medium to large organizations |
| Security & policies | Limited | Granular control and enforcement |

### Domain Controller Setup Checklist

- [ ] VMware installed and virtualization enabled in BIOS
- [ ] Windows Server VM created with adequate RAM and disk
- [ ] Static IP address assigned to server VM
- [ ] Meaningful server name configured
- [ ] AD DS role installed
- [ ] Server promoted to Domain Controller
- [ ] DNS configured and tested
- [ ] Client machine joined to domain

---

## 📝 Study Tips

- **Don't rush.** Small mistakes in server administration can cause big problems. Take your time with each step.
- **Do the labs.** Reading alone is not enough — hands-on practice is what builds real skill.
- **Pay attention to DNS.** Misconfigured DNS is the most common reason clients can't join a domain.
- **Watch your RAM.** Always leave enough memory for the host OS when running multiple VMs simultaneously.

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
