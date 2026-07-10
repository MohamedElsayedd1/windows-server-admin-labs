# 💾 Storage Types, Interfaces, File Systems & RAID Configurations

> A comprehensive reference guide covering DAS, NAS, and SAN storage architectures, disk interfaces, partitioning schemes, file systems, and RAID levels — with practical recommendations for server and client environments.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D4?style=flat-square&logo=windows&logoColor=white)
![Storage](https://img.shields.io/badge/Topic-Storage%20%26%20RAID-4CAF50?style=flat-square)
![Course](https://img.shields.io/badge/Session-19-blueviolet?style=flat-square)
![Level](https://img.shields.io/badge/Level-Intermediate-orange?style=flat-square)

---

## 📖 Overview

This is **Session 19** of the Windows Server 2019 course. This session covers everything an IT administrator needs to know about storage — from physical connection types and disk technologies, through partitioning and file systems, to RAID levels and when to use each.

---

## 🎯 What This Session Covers

| Topic | Description |
|---|---|
| DAS / NAS / SAN | Three storage architectures and their use cases |
| Disk interfaces | IDE, SATA, SAS, NVMe — speed and use case comparison |
| HDD vs SSD | Mechanical vs electronic storage |
| Basic vs Dynamic disks | Partitioning flexibility for servers |
| MBR vs GPT | Partition table schemes |
| File systems | FAT32, NTFS, exFAT, ReFS |
| RAID levels | 0, 1, 5, 6, 10 — benefits, drawbacks, minimum disks |
| Hot swapping | Replacing disks without downtime |

---

## 🗄️ Storage Architectures

### DAS — Direct Attached Storage

```
Server / PC
└── Physical cable (SATA, M.2, SAS)
    └── Storage device (SSD / HDD)
```

- Storage physically connected **directly** to the machine
- Simple to set up and manage
- **Not shared** over a network
- Limited scalability
- Examples: internal SSD, external USB drive, SATA HDD

### NAS — Network Attached Storage

```
Network Switch
├── Server
├── Workstations
└── NAS Device (IP: 192.168.1.50)
    ├── Drive 1
    ├── Drive 2
    ├── Drive 3
    └── Drive 4  (configured with RAID)
```

- Dedicated hardware device with multiple drives connected **over IP**
- Managed via a web console
- Supports RAID for redundancy
- Can join an Active Directory domain
- Supports user permissions and access controls
- Example use: shared file storage for a department

### SAN — Storage Area Network

```
Servers
└── Fiber Channel HBA
    └── Fiber Channel Switch
        └── SAN Array (large disk enclosure)
```

- Dedicated **high-speed fiber channel network** for storage traffic
- Lowest latency, highest performance
- Most expensive solution
- Used in **virtualized environments** and large data centers
- Completely separate network from the regular LAN

### Storage Architecture Comparison

| Feature | DAS | NAS | SAN |
|---|---|---|---|
| Connection | Direct physical cable | IP network | Dedicated fiber channel |
| Shared access | ❌ Single machine | ✅ Multiple users | ✅ Multiple servers |
| Performance | Good | Moderate | Excellent |
| Scalability | Low | Medium | High |
| Cost | Low | Medium | High |
| Best for | Individual machines | File sharing, small-medium org | Enterprise, virtualization |

---

## 🔌 Disk Interfaces

| Interface | Type | Max Speed | Hot Swap | Use case |
|---|---|---|---|---|
| **IDE** | Legacy HDD | ~133 MB/s | ❌ | Obsolete — legacy systems only |
| **SATA III** | HDD / SSD | 600 MB/s | ✅ | Desktops, laptops, budget servers |
| **SAS** | HDD / SSD | 12 Gb/s | ✅ | Enterprise servers, data centers |
| **NVMe (M.2 PCIe)** | SSD only | ~7 GB/s | ❌ (usually) | High-performance workstations, servers |

### IDE vs SATA vs SAS vs NVMe

```
Speed comparison (approximate sequential read):
IDE      ██░░░░░░░░░░░░░░  ~100 MB/s
SATA III █████░░░░░░░░░░░  ~500 MB/s
SAS      ███████░░░░░░░░░  ~1,200 MB/s
NVMe     ████████████████  ~7,000 MB/s
```

> **SAS** supports daisy-chaining up to **15 drives** per cable, making it ideal for dense server storage configurations. It is more reliable and faster than SATA at the cost of higher price.

---

## 💿 HDD vs SSD

| Feature | HDD (Hard Disk Drive) | SSD (Solid State Drive) |
|---|---|---|
| Technology | Spinning magnetic platters + read/write head | Flash memory chips — no moving parts |
| Speed | 80–200 MB/s | 500 MB/s (SATA) to 7,000 MB/s (NVMe) |
| RPM speeds | 5,400 / 7,200 / 10,000 / 15,000 RPM | N/A |
| Durability | Prone to failure from vibration/drops | More durable — no mechanical parts |
| Noise | Audible spinning/seeking | Silent |
| Price per GB | Lower | Higher |
| Best for | Bulk data storage, backups | OS, applications, databases, VMs |

### SSD Types

| SSD Type | Interface | Speed | Form factor |
|---|---|---|---|
| **SATA SSD** | SATA III | ~500 MB/s | 2.5" |
| **NVMe SSD** | PCIe M.2 | ~3,500–7,000 MB/s | M.2 stick |

---

## 🗂️ Disk Types: Basic vs Dynamic

| Feature | Basic Disk | Dynamic Disk |
|---|---|---|
| Partition types | Primary, Extended, Logical | Simple, Spanned, Mirrored, RAID-5 |
| RAID support | ❌ No | ✅ Yes (software RAID) |
| Extend across disks | ❌ No | ✅ Yes (spanned / striped volumes) |
| Mirroring | ❌ No | ✅ Yes (RAID 1) |
| Typical use | Client PCs | Servers, advanced storage |

> Convert a disk to dynamic in **Disk Management**:
> ```
> Right-click disk → Convert to Dynamic Disk
> ```
> ⚠️ This cannot be undone without losing data.

---

## 📐 Partition Schemes: MBR vs GPT

| Attribute | MBR (Master Boot Record) | GPT (GUID Partition Table) |
|---|---|---|
| Max disk size | **2 TB** | **1 ZB** (1 billion TB) |
| Max partitions | 4 primary (or 3 + 1 extended) | **128 primary** partitions |
| Boot firmware | Legacy BIOS | **UEFI** (modern systems) |
| Error detection | Limited | CRC32 checksums — better reliability |
| Redundancy | Single partition table | Backup partition table at end of disk |

> Use **GPT** for all new installations on modern hardware and any disk larger than 2 TB.

---

## 📁 File Systems

| File System | Max Volume | Max File Size | Security (ACLs) | Best for |
|---|---|---|---|---|
| **FAT32** | 32 GB | 4 GB | ❌ No | Legacy, USB drives, small media |
| **NTFS** | 256 TB+ | 16 TB | ✅ Yes | Windows OS, servers, all general use |
| **exFAT** | 128 PB | 16 EB | ❌ No | Flash drives, cross-platform sharing |
| **ReFS** | 35 PB | 35 PB | ✅ Yes | Server storage pools, large data sets |

### NTFS Key Features

- File and folder **permissions** (ACLs)
- **Encryption** (EFS)
- **Compression** of files and folders
- **Disk quotas** per user
- **Journaling** — transaction log for recovery after crashes
- Supports volumes larger than 2 TB (with GPT)

### ReFS Key Features

- Designed for **high data integrity** and large scale
- Automatic **error detection and correction**
- Supports **Storage Spaces** and mirror-accelerated parity
- Does NOT support bootable volumes
- Primarily used in **Hyper-V and Storage Spaces** on Windows Server

---

## 🔁 RAID — Redundant Array of Independent Disks

RAID combines multiple physical disks into one logical unit to improve **performance**, **redundancy**, or both.

> Always prefer **hardware RAID** (dedicated RAID controller) over software RAID for production environments — hardware RAID offloads calculations from the CPU and is more reliable.

### RAID Levels Explained

#### RAID 0 — Striping

```
Disk 1: [A1][A3][A5]
Disk 2: [A2][A4][A6]
→ Data split across both disks (interleaved)
```

| Property | Value |
|---|---|
| Minimum disks | 2 |
| Usable capacity | 100% of total |
| Fault tolerance | ❌ None — one disk fails = all data lost |
| Performance | ✅ Highest read/write speed |
| Use case | Video editing, scratch disks — where speed matters more than safety |

#### RAID 1 — Mirroring

```
Disk 1: [A1][A2][A3]
Disk 2: [A1][A2][A3]  ← exact copy
→ Every write goes to both disks
```

| Property | Value |
|---|---|
| Minimum disks | 2 |
| Usable capacity | 50% of total |
| Fault tolerance | ✅ Survives 1 disk failure |
| Performance | Read improved, write same |
| Use case | OS drives, critical data — simple redundancy |

#### RAID 5 — Striping with Parity

```
Disk 1: [A1][B1][P3]
Disk 2: [A2][P2][C1]
Disk 3: [P1][B2][C2]
→ Data and parity distributed across all disks
```

| Property | Value |
|---|---|
| Minimum disks | 3 |
| Usable capacity | (n-1) × disk size |
| Fault tolerance | ✅ Survives 1 disk failure |
| Performance | Good read, moderate write |
| Use case | File servers, NAS — balance of performance, capacity, redundancy |

#### RAID 6 — Double Parity

```
Like RAID 5 but with two parity blocks per stripe
→ Can survive TWO simultaneous disk failures
```

| Property | Value |
|---|---|
| Minimum disks | 4 |
| Usable capacity | (n-2) × disk size |
| Fault tolerance | ✅ Survives 2 disk failures |
| Performance | Slower writes (double parity calculation) |
| Use case | Large disk arrays where rebuild time is long |

#### RAID 10 — Mirror + Stripe (1+0)

```
Pair 1: Disk 1 ↔ Disk 2 (mirrored)
Pair 2: Disk 3 ↔ Disk 4 (mirrored)
→ Data striped across mirrored pairs
```

| Property | Value |
|---|---|
| Minimum disks | 4 |
| Usable capacity | 50% of total |
| Fault tolerance | ✅ Survives 1 disk per mirrored pair |
| Performance | ✅ Excellent read AND write |
| Use case | High-traffic databases, virtualization hosts |

### RAID Comparison Summary

| RAID | Min Disks | Usable Capacity | Fault Tolerance | Performance | Cost |
|---|---|---|---|---|---|
| **0** | 2 | 100% | ❌ None | ⭐⭐⭐⭐⭐ | Low |
| **1** | 2 | 50% | 1 disk | ⭐⭐⭐ | Medium |
| **5** | 3 | (n-1)/n | 1 disk | ⭐⭐⭐⭐ | Medium |
| **6** | 4 | (n-2)/n | 2 disks | ⭐⭐⭐ | High |
| **10** | 4 | 50% | 1 per pair | ⭐⭐⭐⭐⭐ | High |

---

## 🔄 Hot Swapping & Disk Rebuild

### Hot Swapping

The ability to **remove and replace a disk while the system is running** — no shutdown required.

| Interface | Hot Swap support |
|---|---|
| IDE | ❌ No |
| SATA | ✅ Yes |
| SAS | ✅ Yes |
| NVMe | ❌ Usually no |

### Disk Indicator Lights (RAID Enclosures)

| Light color | Meaning | Action required |
|---|---|---|
| 🟢 Green | Healthy — normal operation | None |
| 🟠 Orange | Warning — disk degrading or rebuilding | Monitor closely; prepare replacement |
| 🔴 Red | Faulty — disk has failed | Replace immediately |

### Rebuild Process

```
1. RAID controller detects failed disk (red light)
        ↓
2. Administrator hot-swaps the failed disk with a new one
        ↓
3. RAID controller automatically begins rebuild
        (uses parity or mirror data to reconstruct lost data)
        ↓
4. Orange light during rebuild — array is degraded but functional
        ↓
5. Green light — rebuild complete, array healthy again
```

> During rebuild, the array operates in a **degraded state**. RAID 5 can survive one disk failure during normal operation but NOT a second failure during rebuild. RAID 6 handles this by tolerating two simultaneous failures.

---

## 📋 Practical Recommendations

| Scenario | Recommended choice |
|---|---|
| Simple individual machine storage | **DAS** — SATA or NVMe SSD |
| Shared file storage for a team | **NAS** with RAID 5 or RAID 6 |
| High-performance enterprise storage | **SAN** with fiber channel |
| OS and application drive | **NVMe SSD** (fastest) |
| Bulk storage / backups | **HDD** (cheapest per GB) |
| Server OS disk | **Dynamic disk** + **GPT** + **NTFS** |
| Client PC disk | **Basic disk** + **GPT** + **NTFS** |
| Flash drive / cross-platform | **exFAT** |
| Server storage pools (Hyper-V) | **ReFS** |
| Max performance, no redundancy needed | **RAID 0** |
| Simple redundancy, 2 disks | **RAID 1** |
| Balance of performance + redundancy | **RAID 5** |
| High redundancy, large arrays | **RAID 6** |
| Best performance + redundancy | **RAID 10** |

---

## 📚 Terminology Reference

| Term | Definition |
|---|---|
| **DAS** | Direct Attached Storage — storage physically connected to one machine |
| **NAS** | Network Attached Storage — shared storage device accessed over IP |
| **SAN** | Storage Area Network — dedicated high-speed fiber channel storage network |
| **SATA** | Serial ATA — common interface for HDDs and SSDs up to 600 MB/s |
| **SAS** | Serial Attached SCSI — enterprise interface for servers, faster and more reliable than SATA |
| **NVMe** | Non-Volatile Memory Express — PCIe-based interface for M.2 SSDs, up to 7 GB/s |
| **HDD** | Hard Disk Drive — mechanical storage using spinning platters |
| **SSD** | Solid State Drive — electronic storage using flash memory chips |
| **RPM** | Revolutions Per Minute — HDD speed measurement (5,400 / 7,200 / 10,000 / 15,000) |
| **Basic Disk** | Standard Windows disk with no software RAID or spanning |
| **Dynamic Disk** | Windows disk type supporting software RAID, spanning, and mirroring |
| **MBR** | Master Boot Record — legacy partition scheme, max 2 TB, max 4 primary partitions |
| **GPT** | GUID Partition Table — modern scheme, max 1 ZB, max 128 partitions |
| **NTFS** | New Technology File System — default Windows FS with ACLs, encryption, compression |
| **ReFS** | Resilient File System — server-focused FS with error correction and large volume support |
| **RAID** | Redundant Array of Independent Disks — combining disks for performance or redundancy |
| **Parity** | Calculated data block allowing RAID to reconstruct lost data from a failed disk |
| **Hot swap** | Replacing a disk without shutting down the system |
| **Rebuild** | RAID controller reconstructing data onto a replacement disk after a failure |

---

## 🔭 Next Session Preview

- **Disk Management in Windows Server** — initializing disks, creating volumes, extending partitions
- **Storage Spaces** — software-defined storage pools in Windows Server
- **iSCSI** — connecting SAN storage over IP networks

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
