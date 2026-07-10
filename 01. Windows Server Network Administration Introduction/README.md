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
| VMware lab environment | Build a safe local virtual network for practice |

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
│   └── setup-guide.md
└── README.md
```

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation or VMware Player |
| Server VM | Windows Server 2022 |
| Client VM(s) | Windows 10 / Windows 11 (1–2 machines) |
| Network | Internal virtual network (host-only or NAT) |

> ⚠️ **Important:** Enable virtualization (VT-x / AMD-V) in your BIOS/UEFI before starting the labs, or the VMs will not run.

---

## 🚀 Getting Started

1. **Set up VMware** — Install VMware Workstation or Player on your host machine.
2. **Create the Server VM** — Install Windows Server 2022, assign a static IP address, and give it a meaningful name.
3. **Install AD DS** — Add the Active Directory Domain Services role via Server Manager.
4. **Promote to Domain Controller** — Run the AD DS configuration wizard and set your domain name (e.g., `company.local`).
5. **Create Client VMs** — Install Windows 10/11 and join them to your domain.
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

### Windows Server Versions Covered

2000 · 2003 · 2008 · 2008 R2 · 2012 · 2012 R2 · 2016 · 2019 · **2022**

### Domain Controller Setup Checklist

- [ ] Windows Server installed on VM
- [ ] Static IP address assigned
- [ ] Meaningful server name configured
- [ ] AD DS role installed
- [ ] Server promoted to Domain Controller
- [ ] DNS configured and tested
- [ ] Client machine joined to domain

---

## 📝 Notes on Studying This Course

- **Don't rush.** Small mistakes in server administration can cause big problems. Take your time with each step.
- **Do the labs.** Reading alone is not enough — hands-on practice is what builds real skill.
- **Pay attention to DNS.** Misconfigured DNS is the most common reason clients can't join a domain.

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
