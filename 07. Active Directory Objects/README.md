# 👥 Active Directory Objects Management

> Lab guide for managing users, groups, and Organizational Units using the Active Directory Users and Computers (ADUC) console in Windows Server 2019.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D4?style=flat-square&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active%20Directory-ADUC-4CAF50?style=flat-square)
![Course](https://img.shields.io/badge/Session-06-blueviolet?style=flat-square)
![Level](https://img.shields.io/badge/Level-Beginner-orange?style=flat-square)

---

## 📖 Overview

This is **Session 6** of the Windows Server 2019 course by **Mohamed Zahdi**. Building on the Domain Controller setup from the previous session, this lab covers day-to-day Active Directory administration — creating Organizational Units, user accounts, and groups — using the **Active Directory Users and Computers (ADUC)** console.

> 📌 **Pre-requisite:** Domain Controller must already be promoted with AD DS installed and `DC.local` domain configured.

---

## 🎯 What This Lab Covers

| Topic | Description |
|---|---|
| Opening ADUC | Accessing the management console via Server Manager Tools |
| Default AD containers | Built-in, Computers, Users, Domain Controllers |
| Organizational Units | Creating and nesting OUs by department |
| User account creation | Naming conventions, passwords, account options |
| Account management | Disabling vs deleting accounts |
| Group creation | Security groups and adding members |
| Group membership | Verifying via Member Of tab |
| Computer accounts | Where joined machines appear in AD |

---

## 🖥️ Opening the ADUC Console

```
Server Manager → Tools → Active Directory Users and Computers
```

Or run directly:

```powershell
dsa.msc
```

The console tree shows the domain (`DC.local`) and all its default containers:

| Container | Contents |
|---|---|
| `Builtin` | Default built-in local groups (Administrators, Users, etc.) |
| `Computers` | Machines joined to the domain land here by default |
| `Domain Controllers` | All promoted DCs in the domain |
| `ForeignSecurityPrincipals` | Objects from trusted external domains |
| `Managed Service Accounts` | Service accounts managed by AD |
| `Users` | Default container for users and some built-in groups |

---

## 🗂️ Organizational Units (OUs)

OUs are **folder-like containers** inside AD used to logically organize users, computers, and groups by department or function. They are the building blocks of delegated administration and Group Policy targeting.

### OU Structure Used in This Lab

This matches the ADUC tree shown in the lab screenshot:

```
DC.local
├── Builtin
├── Computers
├── Domain Controllers
├── ForeignSecurityPrincipals
├── Managed Service Accounts
├── Users
├── Sales          ← department OU
├── Finance        ← department OU
├── HR             ← department OU
└── IT             ← department OU
    ├── IT-Users       ← sub-OU for IT user accounts
    └── IT-Computers   ← sub-OU for IT machines
```

### Why Use OUs?

| Benefit | Example |
|---|---|
| Logical organization | Group all HR users under the HR OU |
| Delegated administration | Let the HR manager reset passwords in the HR OU only |
| Targeted Group Policy | Disable USB ports for Finance users only |
| Scalability | Add sub-OUs as teams grow |

### Creating an OU

```
Right-click the domain (DC.local) → New → Organizational Unit
→ Enter name (e.g., HR) → OK

To create a sub-OU:
Right-click IT → New → Organizational Unit → "IT-Users"
Right-click IT → New → Organizational Unit → "IT-Computers"
```

---

## 👤 User Account Creation

### Naming Convention

Use a consistent format across the domain to avoid confusion:

```
firstname.lastname       →  ahmed.saad
                            maya.saad

If duplicate first names exist:
firstname.middlename     →  mohamed.ahmed
firstname.lastname2      →  mohamed.abdelsettar
```

> Avoid titles, numbers, or special characters in usernames. Keep them clean and predictable.

### Creating a User Account

```
Right-click the target OU (e.g., IT-Users) → New → User
        ↓
First name:   Ahmed
Last name:    Saad
Username:     ahmed.saad
        ↓
Set initial password (must meet complexity requirements)
        ↓
Password options:
  ☑ User must change password at next logon   ← recommended
  ☐ User cannot change password
  ☐ Password never expires
  ☐ Account is disabled
        ↓
Click Finish
```

### Password Complexity Requirements

| Rule | Requirement |
|---|---|
| Minimum length | 7 characters (12+ recommended) |
| Uppercase letters | At least one (A–Z) |
| Lowercase letters | At least one (a–z) |
| Numbers | At least one (0–9) |
| Special characters | At least one (`!`, `@`, `#`, `$`, etc.) |
| Expiration default | Every **42 days** |

### Account Status: Disable vs Delete

| Action | When to use | Effect |
|---|---|---|
| **Disable** | Employee leaves temporarily or permanently | Account preserved; cannot log in; auditable |
| **Delete** | Object permanently no longer needed | Permanently removes the account and its SID |

> ✅ **Best practice:** Always **disable** accounts rather than deleting them. Deleted accounts lose their Security Identifier (SID) and cannot be recovered.

```
Right-click user → Disable Account
Right-click user → Enable Account    ← to re-enable
```

---

## 👥 Group Management

Groups allow you to manage permissions and policies for **multiple users at once**, regardless of which OU they belong to.

### Creating a Group

```
Right-click target OU (e.g., IT-Users) → New → Group
        ↓
Group name:   IT-Group
Group scope:  Global        ← standard for most cases
Group type:   Security      ← use Security (not Distribution) for permissions
        ↓
Click OK
```

### Adding Users to a Group

```
Right-click the group (IT-Group) → Properties → Members tab
→ Click Add → type username → Check Names → OK
```

Or from the user's properties:

```
Right-click user (Ahmed Saad) → Properties → Member Of tab
→ Click Add → type group name → Check Names → OK
```

### Lab Example — Ahmed Saad's Group Membership

As shown in the lab screenshot (`task1-1.png`), user **Ahmed Saad** is a member of:

| Group | AD Path |
|---|---|
| Domain Users | `DC.local/Users` ← default for all domain users |
| IT-Group | `DC.local/IT/IT-Users` ← manually assigned |

```
Ahmed saad → Properties → Member Of:
  ├── Domain Users    (DC.local/Users)       ← automatic
  └── IT-Group        (DC.local/IT/IT-Users) ← assigned in lab
```

> 💡 Users can belong to **multiple groups** regardless of which OU they are in. A Finance user can be a member of the HR-Group if the business requires it.

---

## 👁️ ADUC Console — Lab Snapshot

The IT → IT-Users OU as configured in this lab (`task1.png`):

```
IT-Users OU contents:
┌──────────────┬────────────────┬─────────────┐
│ Name         │ Type           │ Description │
├──────────────┼────────────────┼─────────────┤
│ Ahmed saad   │ User           │             │
│ IT-Group     │ Security Group │             │
│ Maya Saad    │ User           │             │
└──────────────┴────────────────┴─────────────┘
```

---

## 🔑 Administrator Account — Best Practice

The built-in **Administrator** account has unrestricted domain access. Using it for daily tasks is a security risk.

| Recommendation | Action |
|---|---|
| Create a named admin account | New user with same privileges (e.g., `john.admin`) |
| Disable the built-in Administrator | Right-click → Disable Account |
| Use the named account for daily admin | Keeps audit logs meaningful |

---

## 💻 Computer Accounts

When a Windows client joins the domain, a **computer account** is automatically created in the `Computers` container. You can then move it to the appropriate OU:

```
Drag computer object → move to IT-Computers OU
```

Or move via right-click:

```
Right-click computer object → Move → select IT-Computers → OK
```

> Organizing computer accounts into OUs allows you to apply Group Policies to machines by department (e.g., lock screen timeout for all Finance computers).

---

## 📋 Quick Reference — ADUC Common Tasks

| Task | Steps |
|---|---|
| Open ADUC | `Server Manager → Tools → Active Directory Users and Computers` |
| Create OU | Right-click domain/OU → New → Organizational Unit |
| Create user | Right-click OU → New → User |
| Create group | Right-click OU → New → Group |
| Add user to group | User Properties → Member Of → Add |
| Disable user | Right-click user → Disable Account |
| Move object | Right-click object → Move → select target OU |
| Reset password | Right-click user → Reset Password |
| Check group members | Right-click group → Properties → Members |

---

## ✅ Lab Completion Checklist

- [ ] ADUC console opened via Server Manager Tools
- [ ] OUs created: `Sales`, `Finance`, `HR`, `IT`
- [ ] Sub-OUs created inside IT: `IT-Users`, `IT-Computers`
- [ ] Users created inside IT-Users: `Ahmed Saad`, `Maya Saad`
- [ ] Security group created: `IT-Group`
- [ ] Ahmed Saad added to `IT-Group`
- [ ] Group membership verified via Member Of tab
- [ ] Built-in Administrator account disabled
- [ ] VM snapshot taken after configuration

---

## 📚 Terminology Reference

| Term | Definition |
|---|---|
| **ADUC** | Active Directory Users and Computers — the primary GUI console for managing AD objects |
| **OU** | Organizational Unit — a container for grouping AD objects logically |
| **Security Group** | A group used to assign permissions and apply policies to multiple users at once |
| **Domain Users** | Default group that every domain user account belongs to automatically |
| **SID** | Security Identifier — a unique ID assigned to every AD object; lost permanently on deletion |
| **Group Policy** | Rules applied to OUs that control user and computer settings across the domain |
| **DSRM** | Directory Services Restore Mode — emergency recovery mode for the DC |

---

## 🔭 Next Session Preview

- How to **join a Windows client machine to the domain**
- Step-by-step domain join process and what happens in AD when a machine joins

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
