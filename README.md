# Windows Server Administration 2022 — 100 Hands-On Labs

A complete, self-contained **Windows Server 2022 administration course** delivered as 100 practical, screenshot-driven labs — covering everything from virtualization fundamentals and Active Directory through Group Policy, storage, networking services, deployment automation, high availability, file services, and remote access. Each lab folder documents the real configuration workflow end-to-end: the steps taken, the exact errors encountered along the way, and how they were resolved.

This repository is built for IT students, junior sysadmins, and anyone preparing for Windows Server administration roles or certification-track skills — learning by doing, not just reading theory.

---

## 📚 What's Inside

- **100 numbered lab folders**, each containing its own `README.md` walkthrough plus the full set of screenshots referenced in it
- Coverage spans **networking fundamentals → Active Directory → Group Policy → storage → core network services (DHCP/DNS) → deployment (WSUS/WDS/MDT) → Hyper-V → high availability (NLB/Failover Clustering) → file services (FSRM/DAC/DFS/Work Folders) → remote access (VPN/NAT/RADIUS/IPAM)** → capped off with 4 full enterprise-style integration labs
- Every lab explains not just *what* to click, but *why* — plus a running record of real errors hit during setup and their fixes

> 📦 **Note on repo size:** this repo contains ~2,000 screenshots across 100 labs. Images are tracked with **Git LFS** — if cloning, make sure you have [Git LFS installed](https://git-lfs.com/) first (`git lfs install`) so images pull correctly.

---

## 🗂️ Full Lab Index

### Part 1 — Foundations: Virtualization, Installation & Active Directory Basics
| # | Lab |
|---|---|
| 1 | Windows Server Network Administration Introduction |
| 2 | Virtualization |
| 3 | Installing VMware Workstation |
| 4 | Installing Windows 10 on a Virtual Machine |
| 5 | Installing Windows Server 2019 on a Virtual Machine |
| 6 | Setup Active Directory - ADDS Role |
| 7 | Active Directory Objects |
| 8 | Join Machine to Domain |
| 9 | Active Directory Domain Services - AD Objects |
| 10 | Bulk User Creation in Active Directory Using CSV + PowerShell + AI |
| 11 | Join Machine to Domain |
| 12 | Local Users & Groups After Join Domain |
| 13 | User Permissions and Local Groups on Domain-Joined Machines |
| 14 | Active Directory — Full Domain Setup & Domain Join Deep Dive |

### Part 2 — Group Policy (GPO)
| # | Lab |
|---|---|
| 15 | Group Policy Objects (GPOs) in Windows Domain Environments |
| 16 | Group Policy — Practical Lab (HideClock, NoProperties, DisableUSB, ALT+CTRL+DEL) |
| 17 | Advanced Group Policy Lab — Password Policy, Account Lockout, Logon Scripts, Shortcuts & Delegation |
| 18 | AD Sites & Services, Site-Level GPO, Folder Options via Preferences & Policy |
| 19 | GPO Lab — Control Panel Restrictions, Startup Scripts, Firewall Rules & Shortcuts |
| 20 | GPO Practical Lab Exam — exam.local Domain |

### Part 3 — Storage, Disks & File Permissions
| # | Lab |
|---|---|
| 21 | Storage Types, Interfaces, File Systems & RAID Configurations |
| 22 | Disk Management Lab — Partitioning, DiskPart, Dynamic Volumes & RAID |
| 23 | Windows Server Exam LAB (1) |
| 24 | Folder Sharing Permissions on Windows Server |
| 25 | Practical Lab — Folder Sharing, Permissions & Desktop Wallpaper GPO |
| 26 | NTFS vs FAT — File Systems, Permissions, Quotas, FSRM & Shadow Copy |
| 27 | NTFS Security Permissions — Practical Lab |
| 28 | NTFS Disk Quotas — Practical Lab |
| 29 | Shadow Copies & Previous Versions Lab |
| 30 | Mapping Network Drives Lab |
| 31 | File Server Resource Manager (FSRM) Lab |
| 32 | Home Folders & Roaming Profiles |
| 33 | Windows Server Exam LAB (2) |

### Part 4 — Core Network Services: DHCP & DNS
| # | Lab |
|---|---|
| 34 | DHCP — Dynamic Host Configuration Protocol Lecture |
| 35 | DHCP Server — Installation, Scope & Configuration Lab |
| 36 | DNS — Domain Name System Lecture |
| 37 | DNS Lab 1 — Windows Server DNS Management |
| 38 | DNS Lab 2 — Secondary DNS, Zone Transfers & Conditional Forwarding |
| 39 | Windows Server Exam LAB (3) |

### Part 5 — Print Services, Updates & Deployment
| # | Lab |
|---|---|
| 40 | Print and Document Services Lab |
| 41 | Windows Server Update Services (WSUS) Lab |
| 42 | Windows Deployment Services (WDS) Lab |
| 43 | WDS — Image Capture & Deployment Lab |
| 44 | WDS — Auto Domain Join After Deployment |

### Part 6 — Backup, Recovery & Server Core
| # | Lab |
|---|---|
| 45 | Windows Server Backup & Recovery Lab |
| 46 | Active Directory System State Recovery (DSRM) Lab |
| 46 | Windows Server 2022 Core — Installation & Initial Configuration Lab |
| 47 | Windows Server 2022 Core — Remote Management & Role Deployment Lab |

### Part 7 — Hyper-V & Virtualization
| # | Lab |
|---|---|
| 48 | Hyper-V Lab — Installing & Configuring Hyper-V on Windows Server 2022 |
| 49 | Hyper-V Replica Lab — VM Replication Between Hyper-V Hosts |
| 50 | Hyper-V Live Migration Lab — Moving VMs Between Hyper-V Hosts |
| 70 | Hyper-V Virtual Hard Disk (VHD/VHDX) Lab |
| 71 | Hyper-V Virtual Machine & Differencing Disk Lab |

### Part 8 — Remote Administration & Delegation
| # | Lab |
|---|---|
| 51 | Active Directory Administration Lab |
| 52 | Remote Desktop Protocol (RDP) Administration Lab |
| 53 | Windows Remote Assistance Administration Lab |
| 54 | Active Directory – Delegation of Control Lab |
| 55 | IIS Web Server Administration Lab |
| 56 | Windows Admin Center (WAC) Administration Lab |
| 61 | Windows Command Prompt (CMD) Basics Lab |

### Part 9 — Multi-Domain Controllers, FSMO & Forest/Domain Structure
| # | Lab |
|---|---|
| 57 | Additional Domain Controller (ADC) Lab |
| 58 | Active Directory – Additional Domain Controllers & FSMO Roles Lab |
| 59 | Active Directory FSMO Roles Transfer Lab |
| 60 | Active Directory Partitions, Schema & Replication Lab |
| 62 | Active Directory — Create a New Tree Domain (IT.local) in an Existing Forest Lab |
| 63 | Active Directory — Create a Child Domain (HR.company.local) Lab |
| 64 | Active Directory — DNS Stub Zone & External Trust Lab |
| 69 | Windows Server — Active Directory Inter-Site Replication Lab |

### Part 10 — Advanced Storage
| # | Lab |
|---|---|
| 65 | Windows Server — Storage Spaces, Storage Pools & Virtual Disks Lab |
| 66 | Windows Server — Storage Tiers, Virtual Memory & Mixed-Media Storage Pools Lab |
| 67 | Windows Server — Data Deduplication Lab |
| 68 | Windows Server — iSCSI Target & Initiator Lab |

### Part 11 — Web Services & High Availability
| # | Lab |
|---|---|
| 72 | IIS FTP Server Lab — Windows Server |
| 73 | Network Load Balancing (NLB) Lab — Windows Server |
| 74 | Failover Cluster Lab 1 — Shared Storage with iSCSI Target Server |
| 75 | Failover Cluster Lab 2 — Creating the Cluster, Adding a Highly Available File Server, and Configuring Quorum |
| 76 | Failover Cluster Lab 3 — Hyper-V Role and Highly Available Virtual Machines |

### Part 12 — Deployment Automation (MDT + WDS)
| # | Lab |
|---|---|
| 77 | MDT + WDS Lab — Building a Network-Based Windows Deployment Solution |
| 78 | MDT + WDS Lab (Continued) — Boot Image Deployment, Driver Injection, and Reference Image Capture |

### Part 13 — File Server Resource Manager & Access Control
| # | Lab |
|---|---|
| 79 | FSRM Quota Lab — From Built-in NTFS Disk Quotas to File Server Resource Manager |
| 80 | FSRM Lab — File Screening & Storage Reports |
| 81 | FSRM Lab — File Classification Infrastructure |
| 82 | FSRM Lab — Access-Denied Assistance & File Management Tasks |
| 83 | FSRM Lab — Dynamic Access Control (DAC) |

### Part 14 — File Services: BranchCache, Work Folders & DFS
| # | Lab |
|---|---|
| 84 | BranchCache Lab — Hosted Cache Mode |
| 85 | Work Folders Lab |
| 86 | Windows Work Folders Lab — Complete Setup Guide |
| 87 | Windows DFS (Distributed File System) Lab — Complete Setup Guide |
| 88 | DFS Replication Lab — Complete Setup Guide |

### Part 15 — Remote Access, VPN, NAT & RADIUS
| # | Lab |
|---|---|
| 90 | NAT (Network Address Translation) & Routing Lab — Complete Setup Guide |
| 91 | Remote Access VPN Lab — Complete Setup Guide |
| 92 | CMAK (Connection Manager Administration Kit) Lab — Complete Setup Guide |
| 93 | NPS / RADIUS Authentication for VPN — Complete Lab Guide |
| 94 | Deploying an SSTP VPN Secured with a Custom AD CS Certificate Template |
| 95 | Deploying an SSTP VPN Secured with a Custom AD CS Certificate Template |
| 96 | Site-to-Site VPN Using RRAS Demand-Dial Interfaces (IKEv2) |

### Part 16 — IP Management & Network Resilience
| # | Lab |
|---|---|
| 97 | Installing and Provisioning IP Address Management (IPAM) with GPO-Based Provisioning |
| 98 | Windows Server NIC Teaming for Network Fault Tolerance |

### Part 17 — Full Enterprise Integration Labs
| # | Lab |
|---|---|
| 100 | Windows Server Full Lab (4) — Complete Enterprise Domain Environment |

---

## 🧪 Lab Format

Every lab folder follows the same structure:

```
NN. Lab Title/
├── README.md      ← full walkthrough: steps, screenshots, explanations, and errors/fixes
└── (screenshots referenced directly in that folder or an images/ subfolder)
```

Each `README.md` typically includes:
- **Step-by-step configuration** with the reasoning behind each setting (not just "click here")
- **Embedded screenshots** for every major step
- **Errors actually encountered**, their root cause, and the exact fix applied
- A **summary table** of issues → causes → solutions where troubleshooting was involved

---

## 🎯 Who This Is For

- Students working through a structured Windows Server 2022 curriculum
- Junior sysadmins building real, hands-on troubleshooting experience
- Anyone preparing for Windows Server / infrastructure administration roles who wants to see actual errors and fixes, not just theory

## 🛠️ Prerequisites

- A virtualization platform (VMware Workstation, Hyper-V, or VirtualBox)
- Windows Server 2019/2022 evaluation ISO(s)
- Basic familiarity with networking concepts (covered from scratch in Part 1 if needed)

## 📥 Cloning This Repo

Since images are tracked via Git LFS:

```bash
git lfs install
git clone https://github.com/MohamedElsayedd1/windows-server-admin-labs.git
```

## 🤝 Contributing

Spotted an error, have a clarification, or want to help fill in missing lab numbers (89, 99)? Contributions are welcome — open an issue or pull request.

## 📄 License

Specify your preferred license here (e.g. MIT) if sharing this repository publicly.
