# 🏛️ Windows Server as a Domain Controller — Deep Dive

> Comprehensive lab guide covering the full journey from a bare Windows Server install to a fully configured Domain Controller — including AD DS roles, forests, domains, AD objects, and user/group management.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2019%2F2022-0078D4?style=flat-square&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active%20Directory-AD%20DS-4CAF50?style=flat-square)
![Course](https://img.shields.io/badge/Session-09-blueviolet?style=flat-square)
![Level](https://img.shields.io/badge/Level-Intermediate-orange?style=flat-square)

---

## 📖 Overview

This is **Session 9** of the Windows Server 2019 course. This session is a comprehensive deep dive into everything that makes a Windows Server a **Domain Controller** — from understanding what a bare server can and cannot do, through installing AD DS, promoting the server, understanding forests and functional levels, to creating and managing AD objects like users, OUs, and groups.

> 📌 **Pre-requisite:** VMware installed, Windows Server 2019 installed on a VM, static IP assigned, and the machine named. Review Sessions 03–05 if needed.

---

## 🎯 What This Session Covers

| Topic | Description |
|---|---|
| Bare Windows Server | What a fresh install can and cannot do |
| Roles vs Features | The difference and why it matters |
| Installing AD DS | Adding the role via Server Manager |
| Promoting to DC | Configuring forest, domain, and functional levels |
| Forest & domain structure | Root domain, child domains, trust relationships |
| DC types | Standard DC, Member Server, RODC, Global Catalog |
| AD database | NTDS.DIT and transaction log files |
| AD objects | Users, computers, OUs, and groups |
| Password policies | Complexity, expiration, and account status |
| Built-in groups | Domain Admins, Administrators, Domain Users |

---

## 🖥️ What Is a Bare Windows Server?

A freshly installed Windows Server behaves like a normal desktop OS (Windows 10 / 11). It does **nothing special** until the administrator assigns it a role.

```
Fresh Windows Server install
        ↓
No roles → Just an OS
        ↓
Administrator installs roles → Server gains purpose
        ↓
Examples:
  ├── AD DS role    → Domain Controller
  ├── DHCP role     → DHCP Server
  ├── DNS role      → DNS Server
  ├── IIS role      → Web Server
  └── File Services → File Server
```

> The administrator decides what the server does. Without a role, it is just an empty machine.

---

## ⚙️ Roles vs Features

| Term | Definition | Example |
|---|---|---|
| **Role** | The primary function the server will perform | Active Directory Domain Services |
| **Feature** | Supporting component required for a role to work | Group Policy Management, DNS tools |

Roles are the main service. Features are the tools and libraries that support them. When you install a role, Server Manager automatically suggests the features it needs.

---

## 🔧 Step 1 — Install the AD DS Role

```
Server Manager → Manage → Add Roles and Features
        ↓
Installation type: Role-based or feature-based → Next
        ↓
Select destination server → Next
        ↓
Server Roles: ✅ Active Directory Domain Services
→ "Add Features" when prompted
        ↓
Features page → Next (defaults are fine)
        ↓
AD DS info page → Next
        ↓
Confirm → Install
        ↓
Wait for installation to complete
        ↓
Click the yellow flag ⚠️ in Server Manager:
"Promote this server to a domain controller"
```

> ⚠️ Installing the AD DS role is **not enough** — you must also promote the server. Installation only places the binaries on disk. Promotion configures the server to actually act as a DC.

---

## 🌲 Step 2 — Promote the Server to a Domain Controller

### Deployment Configuration Options

| Option | When to use |
|---|---|
| Add a domain controller to an existing domain | Joining an existing forest — adds redundancy |
| Add a new domain to an existing forest | Creating a child domain (e.g., `eu.company.local`) |
| **Add a new forest** | Starting from scratch — first DC in the environment ✅ |

For a new lab environment, always choose **Add a new forest**.

### Forest & Domain Structure

```
Forest: company.local  ← Forest Root Domain (first domain created)
├── company.local      ← Root domain — managed by DC1
├── eu.company.local   ← Child domain — managed by DC2 (Europe branch)
└── us.company.local   ← Child domain — managed by DC3 (US branch)
```

- **Forest** = the top-level container; all domains share one schema
- **Forest root domain** = the first domain created in the forest
- **Child domains** = sub-domains under the root, with their own DCs
- **Trust relationships** = domains in the same forest automatically trust each other

### Domain Controller Options

```
Forest Functional Level:   Windows Server 2016
Domain Functional Level:   Windows Server 2016
        ↓
✅ Domain Name System (DNS) server
✅ Global Catalog (GC)
☐  Read-only domain controller (RODC)
        ↓
Set DSRM password → store it securely outside the server
```

### Functional Levels Reference

| Functional Level | Minimum DC Version Required | Notes |
|---|---|---|
| Windows Server 2008 R2 | Server 2008 R2 | Legacy — avoid for new environments |
| Windows Server 2012 | Server 2012 | Supports Protected Users group |
| Windows Server 2016 | Server 2016 | Azure AD features, PAM trust ✅ recommended |
| Windows Server 2022 | Server 2022 | Latest features |

> Functional levels cannot be lowered after being set. Always choose the highest level your environment supports.

### Paths (leave as defaults)

| Component | Default path |
|---|---|
| NTDS database | `C:\Windows\NTDS` |
| Log files | `C:\Windows\NTDS` |
| SYSVOL | `C:\Windows\SYSVOL` |

### Promotion Summary

```
1. Install AD DS role via Server Manager
2. Click yellow flag → Promote this server to a domain controller
3. Choose "Add a new forest" → set root domain name (e.g., DC.local)
4. Set Forest and Domain Functional Levels → Windows Server 2016
5. Enable DNS Server and Global Catalog
6. Set DSRM password
7. Set NetBIOS name (auto-populated)
8. Leave NTDS / SYSVOL paths as default
9. Review Options → (optionally export PowerShell script)
10. Prerequisites Check → Install
11. Server reboots automatically → DC promotion complete
```

---

## 🗄️ Active Directory Database

Active Directory stores all directory data in two key components:

| Component | File | Purpose |
|---|---|---|
| **NTDS.DIT** | `C:\Windows\NTDS\ntds.dit` | Main database — stores all user, computer, group, and policy objects |
| **Transaction Logs** | `C:\Windows\NTDS\*.log` | Records every change to AD before it is committed to NTDS.DIT |
| **SYSVOL** | `C:\Windows\SYSVOL` | Stores Group Policy templates and logon scripts; replicated across all DCs |

> Back up the **Active Directory System State** regularly. This backup contains NTDS.DIT, logs, and SYSVOL — everything needed to recover the domain after a disaster.

---

## 🖧 DC Types Explained

| Type | Description | Use case |
|---|---|---|
| **Domain Controller** | Windows Server with AD DS installed and promoted | Primary authentication and policy management |
| **Member Server** | Windows Server joined to the domain but NOT a DC | File server, web server, print server |
| **Global Catalog (GC)** | DC that stores a partial replica of all objects across all domains in the forest | Speeds up cross-domain object searches |
| **Read-Only DC (RODC)** | DC with a read-only copy of AD — does not store passwords locally | Branch offices or physically insecure locations |

```
Domain Controller types:

Standard DC   ──── Full read/write AD database
Global Catalog ─── Full AD + partial replica of all forest objects
RODC           ─── Read-only AD, no local passwords, forwards auth to writable DC
Member Server  ─── Joined to domain, no AD DS role
```

---

## 👥 Active Directory Objects

All entities managed by Active Directory are represented as **objects**. Every object must have a **unique name** within the domain.

| Object type | Description | Example |
|---|---|---|
| **User Account** | Represents an employee; used for login and authentication | `ahmed.saad` |
| **Computer Account** | Represents a device joined to the domain | `WINDOWS-PC` |
| **Organizational Unit (OU)** | Container for organizing users, computers, and groups by department | `HR`, `IT`, `Finance` |
| **Group** | Collection of users or computers for permission and policy management | `IT-Group`, `Domain Admins` |

---

## 🗂️ Organizational Units (OUs)

OUs are the primary tool for organizing AD objects and applying Group Policies selectively.

```
DC.local
├── Sales          ← all Sales users and computers
├── Finance        ← all Finance users and computers
├── HR             ← all HR users and computers
└── IT
    ├── IT-Users       ← IT staff accounts
    └── IT-Computers   ← IT machines
```

| OU benefit | Example |
|---|---|
| Logical organization | All HR staff under the HR OU |
| Delegated admin | HR manager can reset passwords in HR OU only |
| Targeted Group Policy | Disable USB ports for Finance users only |
| Scalability | Add sub-OUs as the org grows |

> **OUs vs Groups:** OUs are containers for organization and policy. Groups are security principals for assigning permissions. They serve different purposes and should not be confused.

---

## 👤 User Account Management

### Naming Convention

```
Format:   firstname.lastname     →   ahmed.saad
                                      maya.saad

Duplicates:
          firstname.middlename   →   mohamed.ahmed
          firstname.lastname2    →   mohamed.abdelsettar
```

### Creating a User

```
ADUC → Right-click target OU → New → User
        ↓
First name / Last name / Username
        ↓
Set initial password (complexity required)
        ↓
Password options:
  ☑ User must change password at next logon
  ☐ User cannot change password
  ☐ Password never expires
  ☐ Account is disabled
        ↓
Finish
```

### Password Policy

| Rule | Default |
|---|---|
| Minimum length | 7 characters |
| Complexity | Must meet 3 of 4: uppercase, lowercase, number, special character |
| Expiration | Every **42 days** by default |
| Lockout | Configurable via Fine-Grained Password Policy |

### Account Status

| Action | When to use | Result |
|---|---|---|
| **Disable** | Employee on leave or resigned | Account kept; cannot log in; SID preserved |
| **Enable** | Employee returns | Restores login access |
| **Delete** | Object permanently not needed | Removes account and SID — irreversible |

> ✅ Always **disable** rather than delete. Deleted accounts lose their SID permanently and cannot be recovered.

### Extra User Properties

User accounts can store additional organizational data:

```
User Properties tabs:
├── General     — Name, description
├── Address     — Office, street, city, country
├── Account     — Logon name, password options, expiry
├── Profile     — Home folder, logon script
├── Telephones  — Phone numbers
├── Organization — Job title, department, manager, company
└── Member Of   — Group memberships
```

---

## 👥 Group Management

Groups simplify permission management by allowing bulk assignment — instead of assigning permissions to individual users, you assign them to a group once.

```
Without groups:                     With groups:
  User A → permission               User A ─┐
  User B → permission               User B ─┤→ Group → permission
  User C → permission               User C ─┘
  (repeat for every change)         (change group once, affects all)
```

### Built-in AD Groups

| Group | Scope | Purpose |
|---|---|---|
| **Domain Admins** | Global | Full administrative control over the entire domain |
| **Administrators** | Local | Local admin privileges on domain controllers |
| **Domain Users** | Global | Default group — every domain user is a member automatically |

### Creating a Group

```
ADUC → Right-click OU → New → Group
        ↓
Group name:   IT-Group
Group scope:  Global      ← standard for most cases
Group type:   Security    ← use for permissions (not Distribution)
        ↓
OK
```

### Adding Users to a Group

```
Option 1 — From the group:
Right-click group → Properties → Members → Add

Option 2 — From the user:
Right-click user → Properties → Member Of → Add

Option 3 — Bulk add:
Select multiple users → Right-click → Add to a group
```

---

## 🔐 Local vs Domain Administrator

| Account | Scope | Created when | Use case |
|---|---|---|---|
| **Local Administrator** | Single machine only | During OS installation | Emergency local access; IT-only |
| **Domain Administrator** | Entire domain | Automatically when DC is promoted | Domain-wide administration |

> The domain administrator account is created automatically during DC promotion and inherits full control over the domain. Keep its credentials secure and use a named admin account for daily tasks.

---

## ✅ Lab Completion Checklist

- [ ] AD DS role installed via Server Manager
- [ ] Server promoted to Domain Controller (new forest)
- [ ] Domain name set (e.g., `DC.local`)
- [ ] Forest and Domain functional levels set to Windows Server 2016
- [ ] DNS and Global Catalog enabled
- [ ] DSRM password set and stored securely
- [ ] Server rebooted and DC promotion verified
- [ ] OUs created: Sales, Finance, HR, IT (with IT-Users and IT-Computers sub-OUs)
- [ ] User accounts created with correct naming convention
- [ ] Security groups created and users added
- [ ] Built-in Administrator account disabled; named admin account created
- [ ] Active Directory System State backup configured
- [ ] VM snapshot taken after full DC setup

---

## 📚 Terminology Reference

| Term | Definition |
|---|---|
| **Domain** | Logical boundary of user and computer accounts managed by a DC |
| **Domain Controller** | Windows Server with AD DS installed and promoted to manage the domain |
| **Forest** | Top-level AD container; one or more domains sharing a schema |
| **Forest Root Domain** | The first domain created in a new forest |
| **Global Catalog** | DC storing a partial replica of all forest objects for cross-domain search |
| **RODC** | Read-Only Domain Controller — safe for branch offices; no local passwords |
| **Member Server** | Windows Server joined to domain but not acting as a DC |
| **NTDS.DIT** | The main Active Directory database file |
| **SYSVOL** | Shared folder on DCs containing Group Policy templates and scripts |
| **Functional Level** | Minimum Windows Server version required for DC features |
| **System State Backup** | Backup of AD data (NTDS.DIT, logs, SYSVOL) for disaster recovery |
| **DSRM** | Directory Services Restore Mode — emergency DC recovery mode |
| **OU** | Organizational Unit — container for organizing AD objects |
| **Security Group** | AD group used to assign permissions collectively |

---

## 🔭 Next Session Preview

- Joining Windows client machines to the domain
- What happens in Active Directory when a machine joins
- Login methods: domain account vs local account

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
