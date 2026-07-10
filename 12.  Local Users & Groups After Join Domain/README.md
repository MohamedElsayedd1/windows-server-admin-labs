# 🔐 Local Users and Groups on Domain-Joined Machines

> A practical guide to understanding local vs domain user/group relationships, privilege levels, emergency access, IT Support delegation, and Group Policy restrictions on Windows 10 machines joined to a domain.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D4?style=flat-square&logo=windows&logoColor=white)
![Windows 10](https://img.shields.io/badge/Windows%2010-Client-0078D4?style=flat-square&logo=windows&logoColor=white)
![Group Policy](https://img.shields.io/badge/Group%20Policy-GPO-4CAF50?style=flat-square)
![Course](https://img.shields.io/badge/Session-11-blueviolet?style=flat-square)
![Level](https://img.shields.io/badge/Level-Intermediate-orange?style=flat-square)

---

## 📖 Overview

This is **Session 11** of the Windows Server 2019 course. This session explains how **local users and groups** on a domain-joined Windows 10 machine interact with **domain groups** — and how to correctly manage privilege levels for different types of users without over-granting domain-wide administrative rights.

> 📌 **Pre-requisite:** A Windows 10 machine joined to the domain (e.g., `dashpc.test.local`). Review Session 08 (Joining a Machine to a Domain) if needed.

---

## 🎯 What This Session Covers

| Topic | Description |
|---|---|
| Local Users and Groups | What they are and how to access them |
| Local Users group | Who belongs and what they can do |
| Local Administrators group | Full control on a single machine |
| Domain Users → Local Users | How domain joining links these groups |
| Domain Admins → Local Admins | Full admin on all machines by default |
| Local Administrator account | Why it's kept and when to use it |
| IT Support delegation | Local admin without domain admin rights |
| Specialized user admin | Per-machine admin for designers/devs |
| Group Policy restrictions | Limiting local admin capabilities |
| Ethics in IT | Responsible use of admin privileges |

---

## 🖥️ Accessing Local Users and Groups

```
Right-click This PC → Manage
→ Local Users and Groups
    ├── Users   ← all local user accounts
    └── Groups  ← all local groups
```

Or run directly:

```
lusrmgr.msc
```

---

## 👤 Local Groups — What They Do

### Users Group (Normal Users)

The **local Users group** contains standard users with restricted access. Members:

| Can do | Cannot do |
|---|---|
| Log in to the machine | Install or uninstall software |
| Run permitted applications | Change IP address settings |
| Save personal files | Rename the device |
| Use printers | Join or leave a domain |
| | Install heavy software (Photoshop, AutoCAD, etc.) |

> 💡 About **90% of domain users** fall into this category. Restricting them protects machines from performance issues caused by unauthorized software installations.

### Administrators Group (Full Control)

The **local Administrators group** has unrestricted access to the machine:

- Install and remove any software
- Change network settings
- Rename or reconfigure the machine
- Access all files and folders
- Manage other local users and groups

---

## 🔗 How Domain Joining Links Domain Groups to Local Groups

When a Windows 10 machine joins a domain, two automatic group mappings are created:

```
Domain Join
    ↓
Domain Users  ──────────→  Local Users group
                           (all domain users can log in as normal users)

Domain Admins ──────────→  Local Administrators group
                           (domain admins have full control on every machine)
```

### Full Privilege Mapping Table

| Domain Group | Added to Local Group | Result on Domain-Joined Machine |
|---|---|---|
| **Domain Users** | Local Users | Every domain user can log in as a normal user |
| **Domain Admins** | Local Administrators | Domain admins have full admin rights on all machines |

> Every user created in the domain is **automatically** a member of Domain Users — so they can log into any domain-joined machine by default, as a standard (restricted) user.

---

## 🚨 The Local Administrator Account — Emergency Fallback

The built-in **local Administrator** account exists on every Windows machine and serves as an emergency fallback when domain connectivity is lost.

### When Is It Needed?

| Scenario | Why Local Admin Is Required |
|---|---|
| Domain Controller is offline | Domain credentials cannot be verified |
| Network adapter failure | Machine cannot reach the DC |
| Cached credentials expired | No stored domain login available |
| AD corruption or disaster recovery | Domain authentication unavailable |

### Default Status

```
Windows XP and earlier:  Local Administrator = Enabled by default
Windows Vista and later: Local Administrator = Disabled by default
```

> ✅ **Best practice:** Always keep **one enabled local administrator account** on every machine with a known password. Without it, a machine becomes inaccessible when domain connectivity fails.

### Viewing Local Administrator Status

```
Computer Management → Local Users and Groups → Users
→ Look for "Administrator"
   ├── Arrow icon on account = Disabled
   └── Normal icon = Enabled
```

---

## ⚙️ Managing Local Admin Accounts at Scale

Maintaining individual local admin passwords across hundreds of machines is impractical. The solution is **Group Policy**.

```
Group Policy Object (GPO)
        ↓
Deploy standardized local Administrator account + password
        ↓
Applied automatically to all domain-joined machines
        ↓
Consistent emergency access across the entire fleet
```

> This ensures any IT team member can access any machine in an emergency using a known, standardized local admin credential — without needing domain connectivity.

---

## 🛠️ IT Support Delegation — Local Admin Without Domain Admin

Giving all technical support staff **Domain Admin** rights is a security risk. A single mistake by a support technician could affect the entire domain.

### The Problem

```
IT Support technician needs to:
  ├── Install software on user machines
  ├── Change system settings for troubleshooting
  └── Manage local services and drivers

❌ Wrong approach: Add support staff to Domain Admins
   → They gain control over ALL servers, DCs, and machines
   → One mistake = domain-wide damage
```

### The Correct Approach

```
1. Create a domain group: "IT-Support"
        ↓
2. Add support technicians as members of IT-Support
        ↓
3. Via Group Policy or script:
   Add IT-Support group → Local Administrators group on all machines
        ↓
Result:
  ✅ IT Support has admin rights on domain-joined computers
  ❌ IT Support has NO domain-wide admin rights
```

### Privilege Level Comparison

| Role | Domain Admin | Local Admin on all machines | Domain access |
|---|---|---|---|
| **Domain Admins** | ✅ Yes | ✅ Yes (automatic) | Full domain control |
| **IT-Support group** | ❌ No | ✅ Yes (via GPO) | Limited to machine management |
| **Domain Users** | ❌ No | ❌ No | Normal user access only |

---

## 🎨 Per-Machine Admin for Specialized Users

Some users — graphic designers, developers, database administrators — need administrative rights on **their own machine only** to:

- Install/remove fonts
- Modify registry settings
- Manage specialized plugins or drivers
- Run scripts or developer tools

### The Wrong Approach

```
❌ Add designer to Domain Admins
   → They gain full control over ALL servers and machines in the domain
   → Massive security risk for a non-IT user
```

### The Correct Approach

```
✅ Add the user to the local Administrators group
   on their specific machine only

ADUC or GPO → target machine → Local Administrators
→ Add: test\alice.baker  (only on ALICE-PC)

Result:
  ✅ Alice has admin rights on ALICE-PC
  ❌ Alice is a normal user on every other machine
```

---

## 🛡️ Restricting Local Admins with Group Policy

Even when a user is a local administrator, **Group Policy can restrict what they can do** — preventing misuse while still allowing necessary tasks.

### Common Restrictions Applied via GPO

| Restriction | What it prevents |
|---|---|
| Disable USB ports | Unauthorized data transfer or malware via USB drives |
| Block network settings change | Prevents manual IP changes or disabling network |
| Prevent leaving the domain | User cannot remove the machine from the domain |
| Remove access to domain/workgroup change | Cannot switch to workgroup mode |
| Restrict Control Panel sections | Limits configuration changes to approved areas |

```
Group Policy flow:

DC creates GPO with restrictions
        ↓
GPO applied to OU containing user machines
        ↓
Machines receive policy at next gpupdate cycle
        ↓
Local admins can do their job
but cannot break the network or security model
```

> ✂️ This is often called "cutting the wings" of local admins — they have enough freedom to work, but not enough to cause domain-wide damage.

---

## ⚖️ Ethics of IT Administration

Domain Admins and IT Support staff have elevated access to user files and machine data. This comes with professional responsibility.

| ✅ Legitimate reasons to access user files | ❌ Not acceptable |
|---|---|
| Troubleshooting a reported issue | Browsing user personal files out of curiosity |
| Disaster recovery — restoring lost data | Accessing confidential documents without authorization |
| Security incident investigation | Sharing or copying user data for personal use |
| Compliance audit (with authorization) | Using admin access for personal gain |

> Admin access is granted for **operational purposes only**. Misuse of admin privileges violates professional ethics and may constitute a legal breach depending on jurisdiction.

---

## 📋 Quick Reference — Privilege Decision Guide

```
User needs to log in normally?
└── Domain Users → Local Users group (automatic on domain join) ✅

User needs admin on ALL machines? (e.g., IT Manager)
└── Add to Domain Admins ✅

Support tech needs admin on client machines only?
└── Create IT-Support group → add to local Admins via GPO ✅

Designer/Dev needs admin on their machine only?
└── Add directly to local Administrators on that specific machine ✅

Need to restrict what local admins can do?
└── Apply Group Policy restrictions to the target OU ✅

Emergency — domain is unreachable?
└── Use local Administrator account (fallback) ✅
```

---

## ✅ Lab Completion Checklist

- [ ] Local Users and Groups console opened (`lusrmgr.msc`)
- [ ] Local Users group membership verified (Domain Users present)
- [ ] Local Administrators group membership verified (Domain Admins present)
- [ ] Local Administrator account status checked
- [ ] IT-Support domain group created in ADUC
- [ ] IT-Support group added to local Administrators via GPO or script
- [ ] Specialized user added to local Administrators on their machine only
- [ ] GPO created to restrict local admin capabilities (USB, network, domain exit)
- [ ] GPO applied to correct OU and verified with `gpupdate /force`

---

## 📚 Terminology Reference

| Term | Definition |
|---|---|
| **Local Users and Groups** | Users and groups defined on a single machine, independent of the domain |
| **Domain Users** | Default domain group — every domain account is a member automatically |
| **Domain Admins** | Domain group with full administrative rights across all domain-joined machines |
| **Local Administrator** | Built-in local account with full control over one machine; disabled by default since Vista |
| **Cached Credentials** | Stored domain login credentials on the machine allowing login when the DC is unreachable |
| **Group Policy (GPO)** | Microsoft management tool for enforcing settings and restrictions across multiple machines |
| **IT-Support group** | Custom domain group granted local admin rights without domain-wide admin privileges |
| **lusrmgr.msc** | The Local Users and Groups management console |

---

## 🔭 Next Session Preview

- Introduction to **Group Policy Objects (GPOs)**
- Creating and linking GPOs to OUs
- Common policy settings: password policy, desktop restrictions, software control
- Using `gpupdate /force` and `gpresult /r` to verify applied policies

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
