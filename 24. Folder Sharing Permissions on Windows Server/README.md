# 📁 Folder Sharing Permissions on Windows Server

> A practical guide to sharing server folders over a network — covering share permissions, security permissions, the dangers of default settings, and Role-Based Access Control (RBAC) best practices.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D4?style=flat-square&logo=windows&logoColor=white)
![Permissions](https://img.shields.io/badge/Topic-Share%20Permissions-4CAF50?style=flat-square)
![RBAC](https://img.shields.io/badge/Best%20Practice-RBAC-blueviolet?style=flat-square)
![Course](https://img.shields.io/badge/Session-21-orange?style=flat-square)
![Level](https://img.shields.io/badge/Level-Beginner-yellow?style=flat-square)

---

## 📖 Overview

This is **Session 21** of the Windows Server 2019 course. This session covers how to share server folders over a network, configure share permissions correctly, understand the three permission levels, and apply Role-Based Access Control (RBAC) for secure and manageable access control.

---

## 🎯 What This Session Covers

| Topic | Description |
|---|---|
| What is folder sharing | Making a folder accessible over the network |
| Two required steps | Sharing + permissions — both are mandatory |
| Share name vs folder name | The visible network name can differ from the physical name |
| Default "Everyone" risk | Why it must be removed immediately |
| Three permission types | Read, Change, Full Control |
| Share vs Security permissions | Broad access vs granular control |
| RBAC | Assigning permissions to groups instead of individuals |

---

## 🌐 Why Folder Sharing Requires Two Steps

Simply connecting machines to the same network switch does NOT give users access to server folders. Two explicit steps must be performed on every folder you want to share:

```
Step 1: SHARE the folder
         └── Makes the folder visible and accessible over the network
         └── Assigns a share name (can differ from the real folder name)

Step 2: SET PERMISSIONS
         └── Controls what users can actually DO inside the shared folder
         └── Cannot be skipped — skipping means default settings apply
```

> ⚠️ Many administrators only complete Step 1 and skip Step 2. This leaves the folder exposed to the default "Everyone" group — a **critical security risk**.

### Share Name vs Folder Name

The network share name does not need to match the physical folder name:

```
Physical folder on server:   C:\ServerData\data
Share name visible on network: HR_data

Users access it as: \\SERVER\HR_data
```

This is useful for organizing how resources appear to users without changing the server's folder structure.

---

## 🔧 How to Share a Folder

```
Right-click folder → Properties → Sharing tab
→ Advanced Sharing → Check "Share this folder"
→ Set Share Name (e.g., HR_data)
→ Click Permissions → configure access
→ OK → Apply
```

Or use the simpler Share wizard:

```
Right-click folder → Give access to → Specific people
→ Add users/groups → Set permission level → Share
```

---

## ⚠️ The "Everyone" Default — Remove It Immediately

When you first share a folder, Windows adds **Everyone — Read** as the default permission. This means:

```
Default state after sharing:
└── Everyone (Read) ← includes ANY user on the network
    ↓
Risk: Any person who connects to your network
      can read all files in this folder
```

> This is described as leaving your door wide open — anyone, including unauthorized users or outsiders who gain network access, can read your data.

### Correct First Action After Sharing

```
Folder Properties → Sharing → Advanced Sharing → Permissions
→ Select "Everyone" → Remove
→ Add → specific users or groups
→ Assign appropriate permission level
→ OK
```

---

## 🔑 Three Share Permission Levels

### Permission Comparison Table

| Permission | Read files | Create files | Edit files | Delete files | Change permissions |
|---|---|---|---|---|---|
| **Read** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Change** | ✅ | ✅ | ✅ | ✅ | ❌ |
| **Full Control** | ✅ | ✅ | ✅ | ✅ | ✅ |

---

### 📖 Read Permission

The most restrictive level. Users can view and copy files but cannot make any changes to the server's copy.

```
Read permission allows:
  ✅ Open and view files
  ✅ Copy files to another location

Read permission blocks:
  ❌ Editing files and saving back to the server
  ❌ Creating new files or folders
  ❌ Deleting files or folders
```

**Use case:** Give Read to users who need to view reports or reference documents without risk of accidental changes.

---

### ✏️ Change Permission

The most commonly used permission for active working folders. Users can read, create, edit, and delete.

```
Change permission allows:
  ✅ Everything Read allows
  ✅ Create new files and folders inside the share
  ✅ Edit and save changes to existing files
  ✅ Delete files and folders

Change permission blocks:
  ❌ Modifying share permissions
  ❌ Adding or removing users from the access list
```

> ⚠️ **Change permission includes the ability to delete files.** This cannot be restricted at the share permission level — deletion is bundled with Change. To block deletion while allowing editing, you must use **Security permissions** (see below).

---

### 👑 Full Control Permission

The highest level — includes everything in Read and Change, plus the ability to manage the permissions themselves.

```
Full Control allows:
  ✅ Everything Change allows
  ✅ Modify the Access Control List (ACL)
  ✅ Add or remove users and groups from the share
  ✅ Act as an administrator of the shared folder
```

> ⚠️ **Full Control is extremely dangerous when granted broadly.** Giving Full Control to "Everyone" or large groups is equivalent to handing your server's administrative keys to the entire organization — or anyone who gains network access.

Full Control should be granted **only to trusted administrators** who specifically need to manage the share's permissions.

---

## 🔒 Share Permissions vs Security Permissions

These are two separate layers of access control that work together:

| | Share Permissions | Security Permissions |
|---|---|---|
| **Where set** | Folder Properties → Sharing → Advanced Sharing → Permissions | Folder Properties → Security tab |
| **Applies to** | Network access to the shared folder | Both local and network access |
| **Granularity** | Broad — Read / Change / Full Control only | Granular — can control specific actions (e.g., allow Edit but block Delete) |
| **Delete control** | ❌ Cannot separate delete from Change | ✅ Can allow Write but deny Delete |
| **Best for** | Setting overall access level | Fine-tuning specific allowed actions |

### Combined Approach

```
Share Permissions:
  HR-Group → Change    ← broad access level

Security Permissions:
  HR-Group → Allow: Read, Write, Modify
  HR-Group → Deny: Delete   ← users cannot delete even though they have Change at share level
```

> The **more restrictive** of the two permission layers always wins. Configure both for complete control.

---

## 👥 Role-Based Access Control (RBAC) — Best Practice

**RBAC** means permissions are assigned to **groups**, not individual users. Users inherit permissions through their group membership.

### Why Groups Instead of Individuals?

```
❌ Wrong approach — assigning to individuals:
   ahmed.saad    → Change
   sara.ali      → Change
   omar.hassan   → Change
   (repeat for every new HR employee)

✅ Correct approach — assigning to groups:
   HR-Group → Change
   └── ahmed.saad  (member)
   └── sara.ali    (member)
   └── omar.hassan (member)
   └── [new HR hire] (just add to group — inherits permission automatically)
```

### Benefits of RBAC

| Benefit | Description |
|---|---|
| Simpler management | Add/remove users from a group instead of editing every permission |
| Fewer mistakes | One place to update access when an employee changes role |
| Consistent permissions | All users in a role get exactly the same access |
| Auditable | Easy to check who has access to what by reviewing group membership |

### Practical Example

```
Shared folder: \\SERVER\HR_data

Share Permissions:
  └── HR-Group      → Change    (HR staff can work with files)
  └── HR-Managers   → Full Control  (HR managers control permissions)
  └── IT-Group      → Full Control  (IT admin access for maintenance)
  └── Auditors      → Read      (view-only for auditing purposes)

[Everyone group: REMOVED]
```

---

## 📋 Sharing a Folder — Step-by-Step Checklist

```
1. Create the folder on the server
        ↓
2. Right-click → Properties → Sharing → Advanced Sharing
        ↓
3. Check "Share this folder" → set Share Name
        ↓
4. Click Permissions → REMOVE "Everyone"
        ↓
5. Click Add → enter group name (e.g., HR-Group)
        ↓
6. Select permission level (Read / Change / Full Control)
        ↓
7. Click OK → Apply
        ↓
8. (Optional) Go to Security tab → fine-tune specific actions
        ↓
9. Test: log in as a member of the group → access \\SERVER\ShareName
```

---

## ✅ Lab Completion Checklist

- [ ] Folder created on the server
- [ ] Folder shared via Advanced Sharing with a custom share name
- [ ] "Everyone" group removed from share permissions
- [ ] AD groups created for each department (HR-Group, IT-Group, etc.)
- [ ] Groups added to share permissions with appropriate levels
- [ ] Security tab configured for granular control (if needed)
- [ ] Tested access from a client machine logged in as a group member
- [ ] Verified Read-only users cannot edit or delete
- [ ] Verified Change users can create and edit but cannot change permissions
- [ ] Verified Full Control users can manage permissions

---

## 📚 Terminology Reference

| Term | Definition |
|---|---|
| **Share** | Making a folder accessible over the network to other machines |
| **Share name** | The name the folder appears under on the network (can differ from the real folder name) |
| **Share permissions** | Three-level access control (Read/Change/Full Control) for network access |
| **Security permissions** | Granular NTFS access control list (ACL) for both local and network access |
| **Everyone group** | A built-in Windows group that includes all users — dangerous to leave in share permissions |
| **Read** | View and copy files only — no changes |
| **Change** | Read + create + edit + delete — cannot change permissions |
| **Full Control** | All Change rights + ability to modify the share's access control list |
| **ACL** | Access Control List — the list of users/groups and their permissions on a resource |
| **RBAC** | Role-Based Access Control — assigning permissions to groups based on job role |
| **UNC path** | Universal Naming Convention path — `\\SERVER\ShareName` used to access network shares |

---

## 🔭 Next Session Preview

- **NTFS Security Permissions in depth** — special permissions, inheritance, and effective access
- **Shared folder access from the command line** — `net use` and `net share`
- **Mapping network drives via GPO** — automatically mapping shares at user login

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
