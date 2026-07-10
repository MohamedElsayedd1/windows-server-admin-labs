# 🗂️ NTFS vs FAT — File Systems, Permissions, Quotas, FSRM & Shadow Copy

> A comprehensive reference guide covering the differences between FAT32 and NTFS, NTFS security permissions and inheritance, disk quotas, File Server Resource Manager (FSRM), and Shadow Copy — with practical implications for Windows Server administration.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D4?style=flat-square&logo=windows&logoColor=white)
![NTFS](https://img.shields.io/badge/File%20System-NTFS-4CAF50?style=flat-square)
![FSRM](https://img.shields.io/badge/Tool-FSRM-blueviolet?style=flat-square)
![Course](https://img.shields.io/badge/Session-23-orange?style=flat-square)
![Level](https://img.shields.io/badge/Level-Intermediate-yellow?style=flat-square)

---

## 📖 Overview

This is **Session 23** of the Windows Server 2019 course. This session covers the fundamental differences between FAT32 and NTFS, how formatting works, NTFS security features, disk quotas, folder-level quotas via FSRM, file screening, and Shadow Copy snapshots for quick data recovery.

---

## 🎯 What This Session Covers

| Topic | Description |
|---|---|
| What formatting really means | Reinitializing the disk structure — not just deleting data |
| FAT32 limitations | Cluster size, max file size, max disk size, no security |
| NTFS improvements | Smaller clusters, larger limits, full security feature set |
| Converting FAT to NTFS | Using `convert` without data loss |
| NTFS security permissions | Read, Write, Create, Delete — and how Deny works |
| Permission inheritance | Parent folder settings propagating to children |
| Disk quotas | Per-user storage limits at the partition level |
| FSRM | Folder-level quotas and file type screening |
| Shadow Copy | Volume snapshots for quick recovery |

---

## 💡 What Does "Formatting" Actually Mean?

A common misconception is that formatting simply deletes data. In reality:

```
Formatting = Reinitializing the file system structure on a disk

Before format: disk has old data and old structure
After format:  disk has a fresh, empty file system structure
               (old data may still exist physically but is no longer indexed)
```

The format defines **how data is written and organized** on the disk — FAT32 and NTFS write data in fundamentally different ways.

---

## 📊 FAT32 vs NTFS — Full Comparison

| Feature | FAT32 | NTFS |
|---|---|---|
| **Max disk size** | 32 GB | 2 TB+ |
| **Max file size** | **4 GB** | Several TB |
| **Cluster size** | 16 KB (typical) | **4 KB** |
| **Storage efficiency** | Lower — more slack space | Higher — less wasted space |
| **Security permissions** | ❌ None | ✅ Full ACL support |
| **Disk quotas** | ❌ No | ✅ Yes (partition level) |
| **Folder quotas** | ❌ No | ✅ Yes (via FSRM) |
| **File screening** | ❌ No | ✅ Yes (via FSRM) |
| **Shadow Copy** | ❌ No | ✅ Yes |
| **Encryption (EFS)** | ❌ No | ✅ Yes |
| **Compression** | ❌ No | ✅ Yes |
| **Best for** | USB drives, legacy devices | Windows OS, servers, all modern use |

---

## 🗃️ Cluster Size and Storage Efficiency

A **cluster** is the smallest unit of storage allocation on a disk. When a file is written, it occupies one or more full clusters — any leftover space in the last cluster is **wasted** (called slack space or internal fragmentation).

### FAT32 — 16 KB Cluster

```
File: 20 KB
Cluster 1: [████████████████] 16 KB — full
Cluster 2: [████░░░░░░░░░░░░]  4 KB used + 12 KB wasted

Wasted: 12 KB per file minimum if smaller than one cluster
```

### NTFS — 4 KB Cluster

```
File: 20 KB
Cluster 1: [████] 4 KB
Cluster 2: [████] 4 KB
Cluster 3: [████] 4 KB
Cluster 4: [████] 4 KB
Cluster 5: [████] 4 KB

Total: 5 × 4 KB = 20 KB  ← no wasted space
```

> NTFS's smaller 4 KB cluster size means far less wasted space, especially with many small files. This significantly improves overall disk utilization on large volumes.

---

## 🔄 Converting FAT32 to NTFS Without Data Loss

Normally, changing a file system requires formatting — which destroys all data. Windows provides a conversion command that upgrades FAT32 to NTFS while **keeping existing files intact**:

```cmd
convert E: /fs:ntfs
```

```
Output:
The type of the file system is FAT32.
Windows is verifying files and folders...
Conversion complete
```

> ⚠️ This conversion is **one-way** — you cannot convert NTFS back to FAT32 without formatting. Always take a backup before converting on production systems.

### When Is This Useful?

A common scenario: copying a large ISO file (e.g., Windows Server 2022 ISO — ~5 GB) to a USB drive formatted as FAT32 fails because FAT32 has a **4 GB maximum file size**. Converting the USB to NTFS solves this without losing other files already on the drive.

---

## 🔐 NTFS Security Permissions

NTFS introduces granular, file-level security through **Access Control Lists (ACLs)**. Permissions can be assigned to individual users or groups.

### Basic NTFS Permission Types

| Permission | What it allows |
|---|---|
| **Read** | View file contents and folder listings |
| **Write** | Modify file contents |
| **Create** | Add new files or subfolders |
| **Delete** | Remove files and folders |
| **Read & Execute** | View and run executable files |
| **Modify** | Read + Write + Delete (but not change permissions) |
| **Full Control** | All above + change permissions and take ownership |

### Deny Overrides Allow — Always

When a user belongs to multiple groups with conflicting permissions, **Deny always wins**:

```
User: ahmed.saad
├── HR-Group     → Allow: Write
└── Restricted   → Deny: Write

Effective permission: DENY Write
(Deny from Restricted overrides Allow from HR-Group)
```

| User permission | Group permission | Effective | Rule |
|---|---|---|---|
| Allow | Deny | **Deny** | Deny wins |
| Deny | Allow | **Deny** | Deny wins |
| Allow | Allow | **Allow** | Both allow → access granted |
| Deny | Deny | **Deny** | Both deny → blocked |

> ⚠️ Use **Deny** sparingly. It is a powerful tool but can unexpectedly block access when users belong to multiple groups.

### Practical Permission Example

```
HR Data folder
├── HR-Group → Allow: Read, Write, Create
├── HR-Group → Deny: Delete      ← cannot delete files
└── Domain Admins → Full Control
```

This setup lets HR users work with files freely but prevents accidental or intentional deletion. The Delete restriction cannot be achieved with Share permissions alone — it requires NTFS Security permissions.

---

## 🔗 Permission Inheritance

By default, permissions set on a **parent folder propagate to all child folders and files** automatically.

```
HR-Data\                      ← HR-Group has Change here
├── Reports\                  ← inherits HR-Group Change
│   ├── Q1.xlsx               ← inherits HR-Group Change
│   └── Q2.xlsx               ← inherits HR-Group Change
└── Archive\                  ← inherits HR-Group Change
    └── 2023-data.xlsx        ← inherits HR-Group Change
```

### Blocking Inheritance

To apply different permissions to a specific subfolder:

```
Right-click subfolder → Properties → Security → Advanced
→ Disable inheritance
→ Choose: Convert inherited permissions into explicit permissions
           OR Remove all inherited permissions
→ Now set custom permissions for this folder only
```

---

## 📏 Disk Quotas (Partition Level)

NTFS supports **disk quotas** — limits on how much disk space each user can consume on a partition.

```
Partition E: → 100 GB total
├── User quota: 2 GB per user
│   ├── ahmed.saad  → can use up to 2 GB
│   ├── sara.ali    → can use up to 2 GB
│   └── omar.hassan → can use up to 2 GB
```

### Setting Disk Quotas

```
File Explorer → Right-click drive → Properties → Quota tab
→ Enable quota management
→ Set default quota limit (e.g., 2 GB)
→ Set warning level (e.g., 1.5 GB — user gets warned)
→ Apply
```

> ⚠️ **Limitation:** Disk quotas apply at the **partition level only** — you cannot set a quota on a specific folder (e.g., limit the HR folder to 10 GB) using the built-in quota system. For folder-level control, use **FSRM**.

---

## 🛠️ File Server Resource Manager (FSRM)

FSRM is a Windows Server role that extends disk management with two powerful features: **folder-level quotas** and **file screening**.

### Installing FSRM

```
Server Manager → Add Roles and Features
→ File and Storage Services → File and iSCSI Services
→ File Server Resource Manager → Install
```

### Feature 1 — Folder Quotas

Set storage limits on **specific folders**, not just partitions:

```
FSRM → Quota Management → Quotas → Create Quota
→ Quota path: E:\HR-Data
→ Quota type: Hard limit (blocks when reached) or Soft limit (warns only)
→ Set limit: 10 GB
→ Create
```

| Quota Type | Behavior | Best for |
|---|---|---|
| **Hard limit** | Blocks writes when limit is reached | Enforced storage limits |
| **Soft limit** | Warns user but still allows writes | Monitoring usage |

### Feature 2 — File Screening

Block specific file types from being saved in certain folders:

```
FSRM → File Screening Management → File Screens → Create File Screen
→ File screen path: E:\HR-Data
→ File screen template: Block Audio and Video Files
   OR create custom: block .mp3, .exe, .pdf, .avi, etc.
→ Create
```

### Example Screening Rules

| Folder | Blocked file types | Reason |
|---|---|---|
| `E:\HR-Data` | `.mp3`, `.avi`, `.mkv` | Keep work folder clean |
| `E:\Shared` | `.exe`, `.bat`, `.ps1` | Prevent malware uploads |
| `E:\Archive` | All except `.pdf`, `.docx` | Archive format compliance |

---

## 📸 Shadow Copy (Volume Snapshots)

**Shadow Copy** creates point-in-time snapshots of a volume, allowing users and admins to restore previous versions of files without a full backup restore.

### How Shadow Copy Works

```
10:00 AM  → Snapshot taken  [original state]
           User modifies file.docx
11:00 AM  → User accidentally deletes important.xlsx
           Restore from 10:00 AM snapshot → file recovered ✅

12:00 PM  → Snapshot taken  [current state]
           Only the delta (changes) since last snapshot is stored
```

### Enabling Shadow Copy

```
Right-click drive → Properties → Shadow Copies tab
→ Select volume → Enable
→ Configure schedule (every 30 min, hourly, daily, etc.)
→ Set storage location (preferably a different disk)
→ OK
```

### Restoring a Previous Version

```
Right-click file or folder → Properties → Previous Versions tab
→ Select a snapshot from the list
→ Restore (overwrites current) or Copy (saves alongside)
```

### Shadow Copy Storage — Best Practices

```
Source data:   Disk 0 (C: or E:)
Shadow copies: Disk 1 (separate physical disk)
```

> Storing shadow copies on the same disk as the source data means a disk failure destroys both — defeating the purpose.

### Shadow Copy vs Backup

| | Shadow Copy | Full Backup |
|---|---|---|
| **Purpose** | Quick file-level recovery | Disaster recovery |
| **Speed** | Very fast restore | Slower restore |
| **Storage** | Delta only | Full copy |
| **Survives disk failure?** | ❌ No — same volume dependency | ✅ Yes — independent media |
| **Survives format?** | ❌ No | ✅ Yes |
| **Use as backup?** | ❌ Not a replacement | ✅ Primary recovery method |

> Shadow Copy is the **first line of defense** for quick recovery from accidental deletion or modification. It is **not a substitute for proper backups**.

---

## 📋 Practical Recommendations

| Scenario | Recommendation |
|---|---|
| USB drive for large files (>4 GB) | Convert to NTFS or use exFAT |
| Server OS and data volumes | Always use NTFS |
| Need to limit storage per user | NTFS Disk Quotas (partition level) |
| Need to limit storage per folder | FSRM Folder Quotas |
| Block MP3/EXE files in work folders | FSRM File Screening |
| Quick recovery from accidental deletion | Enable Shadow Copy with frequent schedule |
| Disaster recovery | Windows Server Backup or third-party backup solution |
| Allow edit but block delete | NTFS Security → Allow Modify, Deny Delete |

---

## ✅ Lab Completion Checklist

- [ ] `convert E: /fs:ntfs` run successfully — conversion confirmed
- [ ] NTFS security permissions configured on a test folder
- [ ] Deny Delete applied to HR-Group — deletion blocked
- [ ] Permission inheritance tested on subfolders
- [ ] Disk quota set on a partition — limit confirmed
- [ ] FSRM role installed on Windows Server
- [ ] Folder quota created for `E:\HR-Data`
- [ ] File screen created blocking `.mp3` and `.exe` files
- [ ] Shadow Copy enabled on a volume with a schedule
- [ ] Previous version restored from a Shadow Copy snapshot
- [ ] VM snapshot taken after configuration

---

## 📚 Terminology Reference

| Term | Definition |
|---|---|
| **Format** | Reinitializing the file system structure on a disk or partition |
| **Cluster** | Smallest storage allocation unit — 16 KB in FAT32, 4 KB in NTFS |
| **Slack space** | Wasted space within a cluster when a file is smaller than the cluster size |
| **ACL** | Access Control List — the list of users/groups and their NTFS permissions on a resource |
| **Deny** | NTFS permission that overrides Allow in all cases of conflict |
| **Inheritance** | Parent folder permissions automatically propagating to child folders and files |
| **Disk Quota** | Per-user storage limit applied at the partition level |
| **FSRM** | File Server Resource Manager — Windows Server tool for folder quotas and file screening |
| **File Screening** | FSRM feature that blocks specific file extensions from being saved in a folder |
| **Shadow Copy** | Point-in-time volume snapshot enabling previous version restore |
| **Delta snapshot** | Shadow Copy stores only the changes since the last snapshot, not a full copy |
| **EFS** | Encrypting File System — NTFS feature for encrypting individual files and folders |

---

## 🔭 Next Session Preview

- **Shared folder access via `net use` and `net share`** commands
- **Mapping network drives via GPO Preferences** — deploying `Z: → \\SERVER\Share` at login
- **Auditing file access** — logging who accessed or modified files

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
