# 🛡️ Group Policy Objects (GPOs) in Windows Domain Environments

> A practical guide to understanding, creating, and managing Group Policy Objects — covering the application hierarchy, user vs computer configuration, policy refresh, and common real-world use cases.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D4?style=flat-square&logo=windows&logoColor=white)
![Group Policy](https://img.shields.io/badge/Group%20Policy-GPO-4CAF50?style=flat-square)
![Active Directory](https://img.shields.io/badge/Active%20Directory-GPMC-blueviolet?style=flat-square)
![Course](https://img.shields.io/badge/Session-14-orange?style=flat-square)
![Level](https://img.shields.io/badge/Level-Intermediate-yellow?style=flat-square)

---

## 📖 Overview

This is **Session 14** of the Windows Server 2019 course. This session covers **Group Policy Objects (GPOs)** — the primary tool domain administrators use to control user and computer settings across an entire network from a single place.

> 📌 **Pre-requisite:** A working Domain Controller with AD DS installed, OUs created, and at least one client machine joined to the domain. Review Sessions 06–08 if needed.

---

## 🎯 What This Session Covers

| Topic | Description |
|---|---|
| What is Group Policy | Definition and purpose in a domain environment |
| Local vs Domain GPO | Standalone machine policy vs domain-wide policy |
| GPO application hierarchy | Local → Site → Domain → OU |
| User Configuration | Policies that follow the user across machines |
| Computer Configuration | Policies that apply to a machine regardless of user |
| Practical GPO examples | USB restriction, hiding UI elements, Task Manager |
| GPO exceptions | Allowing specific users to bypass a policy |
| GPO tools | GPEDIT.MSC and Group Policy Management Console |
| Registry interaction | How GPOs modify the Windows Registry |
| Policy refresh | 90-minute auto-refresh and `gpupdate /force` |

---

## 📌 What Is Group Policy?

**Group Policy** is a centralized administrative tool that allows administrators to **enable or disable features and permissions** on user accounts and computers across a Windows domain — without touching each machine individually.

```
Administrator creates GPO on the Domain Controller
        ↓
GPO linked to an OU (or domain/site)
        ↓
Policy settings pushed to all targeted users/computers
        ↓
Registry values updated on each machine
        ↓
Settings enforced automatically
```

> Every Group Policy setting ultimately writes a value to the **Windows Registry** on the target machine. GPOs are the mechanism; the registry is where the effect lives.

---

## 🏗️ GPO Application Hierarchy

Group Policies are applied in a strict order. Policies applied later override earlier ones when there is a conflict.

```
1. LOCAL          ← Applied first (lowest priority)
        ↓
2. SITE           ← Physical location (e.g., a building or campus)
        ↓
3. DOMAIN         ← Applied to all users and computers in the domain
        ↓
4. OU             ← Applied last (highest priority, most granular)
```

### Hierarchy Summary Table

| Level | Scope | Priority | Most common? |
|---|---|---|---|
| **Local** | Single standalone machine | Lowest | Rarely — used without a domain |
| **Site** | Physical geographic location with multiple DCs | Low | Occasionally — for location-specific settings |
| **Domain** | All users and computers in the entire domain | Medium | Yes — for baseline security policies |
| **OU** | Specific group of users or computers | Highest | ✅ Most common — granular and flexible |

> In practice, **OU-level GPOs** are used for most day-to-day policy management because they target specific departments or machine groups without affecting the whole domain.

---

## 👤 User Configuration vs 💻 Computer Configuration

Every GPO contains two independent sections. Understanding the difference is critical:

| | User Configuration | Computer Configuration |
|---|---|---|
| **Applied to** | User account | Computer account |
| **Follows** | The user — to any machine they log into | The machine — affects any user who logs in |
| **Takes effect** | At user logon / logoff | At machine startup / restart |
| **Example** | Hide Control Panel for user Ahmed regardless of which PC he uses | Disable USB ports on HR-PC01 for all users |

### Choosing the Right Configuration

```
Should the policy follow the PERSON?
└── Use User Configuration
    Example: Block access to Task Manager for all standard users

Should the policy follow the MACHINE?
└── Use Computer Configuration
    Example: Disable USB on all computers in the Finance OU
```

---

## 🔧 GPO Management Tools

### 1. GPEDIT.MSC — Local Group Policy Editor

Used on a **standalone machine** (not domain-joined) to manage local policies only.

```
Run → gpedit.msc
→ Local Computer Policy
    ├── Computer Configuration
    └── User Configuration
```

> Changes made here only affect the local machine and are not domain-managed.

### 2. Group Policy Management Console (GPMC)

Used on the **Domain Controller** to create, link, and manage domain GPOs.

```
Server Manager → Tools → Group Policy Management
→ Forest → Domains → domain.local
    ├── Default Domain Policy   ← baseline policy for entire domain
    ├── HR OU
    │   └── HR-USB-Block GPO    ← targeted policy for HR
    └── Finance OU
        └── Finance-Restrictions GPO
```

### Creating a New GPO

```
GPMC → Right-click target OU → Create a GPO in this domain and link it here
→ Name the GPO (e.g., "HR-Disable-USB")
→ Right-click the new GPO → Edit
→ Group Policy Management Editor opens
→ Navigate to the setting → Enable or Disable
→ Close editor → GPO is saved and linked
```

---

## ⚙️ Practical GPO Examples

### 1. Disable USB Access for HR Users

```
GPO: HR-Disable-USB
Linked to: HR OU
Configuration: Computer Configuration
Path: Computer Configuration → Administrative Templates
      → System → Removable Storage Access
      → All Removable Storage classes: Deny all access → Enabled
```

Effect: No user on any HR machine can use USB drives.

### 2. Hide System Clock Settings

```
GPO: Prevent-Time-Change
Linked to: Domain or specific OU
Configuration: User Configuration
Path: User Configuration → Administrative Templates
      → Control Panel → Date and Time
      → Prevent changing date and time → Enabled
```

Effect: The targeted user cannot change system time on any machine.

### 3. Disable Task Manager

```
GPO: Block-Task-Manager
Linked to: target OU
Configuration: User Configuration
Path: User Configuration → Administrative Templates
      → System → Ctrl+Alt+Del Options
      → Remove Task Manager → Enabled
```

### 4. Block Access to Run Command

```
GPO: Disable-Run
Configuration: User Configuration
Path: User Configuration → Administrative Templates
      → Start Menu and Taskbar
      → Remove Run menu from Start Menu → Enabled
```

---

## 🚪 GPO Exceptions — Excluding Specific Users

Sometimes a policy should apply to a whole OU **except** for specific users (e.g., the HR Manager should be able to use USB while all other HR staff cannot).

### Method — Security Filtering

```
GPMC → select the GPO
→ Scope tab → Security Filtering
→ Remove "Authenticated Users"
→ Add the specific group that SHOULD receive the policy
   (e.g., "HR-Standard-Users")

Result:
  ✅ HR-Standard-Users → policy applied (USB blocked)
  ❌ HR Manager        → policy NOT applied (USB allowed)
```

### Method — Deny Permission

```
GPMC → select the GPO → Delegation tab → Advanced
→ Add the user to exclude (e.g., HR Manager)
→ Set "Apply Group Policy" → Deny

Result:
  ✅ All HR users  → USB blocked
  ❌ HR Manager    → exempted from this GPO
```

---

## 🔄 Policy Refresh and Enforcement

### Automatic Refresh

Group Policies automatically refresh every **90 minutes** on domain-joined machines. The DC pushes updated policy settings to all targets at this interval.

| Machine type | Refresh interval |
|---|---|
| Domain-joined workstations | Every 90 minutes (with random 0–30 min offset) |
| Domain Controllers | Every 5 minutes |

### Force Immediate Update

To apply a policy without waiting for the automatic refresh:

```powershell
# Run on the target machine (as admin)
gpupdate /force

# Output:
# Updating Computer policy...
# Updating User policy...
# Computer Policy update has completed successfully.
# User Policy update has completed successfully.
```

### When Policies Take Effect

| Policy type | Requires |
|---|---|
| **User Configuration** | User must **log off and log back on** |
| **Computer Configuration** | Machine must **restart** |

> `gpupdate /force` refreshes the policy from the DC — but some settings (especially Computer Configuration) still require a restart to fully apply.

---

## ✅ Verifying Applied Policies

### Check which policies are applied to a machine or user:

```powershell
# View applied GPOs and their settings
gpresult /r

# Full detailed HTML report
gpresult /h C:\gpo-report.html
```

### Check from GPMC

```
GPMC → Group Policy Results (right-click domain)
→ Run Group Policy Results Wizard
→ Select target computer and user
→ View full policy application report
```

---

## 📋 GPO Planning Reference

Before creating a GPO, answer these questions:

| Question | Answer determines |
|---|---|
| Who should this affect — users or machines? | User Configuration vs Computer Configuration |
| Should it apply domain-wide or to a specific department? | Domain-level vs OU-level link |
| Are there exceptions — users who should be exempt? | Security Filtering or Deny permissions |
| When does it need to take effect? | Logoff/logon (user) or restart (computer) |
| How will I verify it worked? | `gpresult /r` or GPMC Results Wizard |

---

## ✅ Lab Completion Checklist

- [ ] Group Policy Management Console opened on DC
- [ ] New GPO created and linked to HR OU: `HR-Disable-USB`
- [ ] USB restriction policy configured under Computer Configuration
- [ ] New GPO created and linked to domain: `Prevent-Time-Change`
- [ ] Clock change policy configured under User Configuration
- [ ] GPO exception created for HR Manager using Security Filtering
- [ ] `gpupdate /force` run on client machine
- [ ] `gpresult /r` run to verify policies applied
- [ ] Policies tested on a standard HR user account
- [ ] VM snapshot taken after GPO configuration

---

## 📚 Terminology Reference

| Term | Definition |
|---|---|
| **GPO** | Group Policy Object — a named set of policy configurations applied to users or computers |
| **GPMC** | Group Policy Management Console — the tool on the DC for managing domain GPOs |
| **GPEDIT.MSC** | Local Group Policy Editor — for standalone machines without a domain |
| **User Configuration** | GPO section whose settings follow the user account across all machines |
| **Computer Configuration** | GPO section whose settings apply to a specific machine for all users |
| **OU** | Organizational Unit — the most common target for GPO links |
| **Security Filtering** | Controls which users or groups a GPO applies to within an OU |
| **gpupdate /force** | Command to immediately pull the latest policies from the DC |
| **gpresult /r** | Command to display which GPOs are currently applied to a user/machine |
| **Registry** | Windows system database where GPO settings are ultimately stored and enforced |
| **Site** | A physical geographic grouping of domain controllers for policy targeting |

---

## 🔭 Next Session Preview

- **Advanced GPO settings** — software deployment, logon scripts, folder redirection
- **Fine-Grained Password Policies** — different password rules for different user groups
- **GPO troubleshooting** — resolving conflicts and inheritance issues

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
